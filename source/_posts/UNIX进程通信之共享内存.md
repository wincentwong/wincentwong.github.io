---
title: UNIX进程通信之共享内存
date: 2017-04-23 00:43:18
tags: UNIX
---

## Code:

``` c
#include <stdio.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/sem.h>
#define SHMKEY 18001     /* 共享内存关键字 */
#define SIZE 1024        /* 共享内存长度 */
#define SEMKEY1 19001    /* 信号量组1关键字 */
#define SEMKEY2 19002    /* 信号量组2关键字 */

static void semcall(int sid,int op)
{
    struct sembuf sb;
    sb.sem_num = 0;     //信号灯编号0
    sb.sem_op  = op;    //信号灯操作数加1或减1
    sb.sem_flg = 0;     //操作标记
    if(semop(sid,&sb,1) == -1)
        perror("semop");//出错处理
};
void semWait(int sid)
{
    semcall(sid,-1);
}
void semSignal(int sid)
{
    semcall(sid,1);
}


int creatsem(key)
key_t key;
{
 int sid;
 union semun {   /* 如sem.h中已定义，则省略 */
	  int val;
	  struct semid_ds *buf;
	  ushort *array;
 } arg;
 if((sid=semget(key,1,0666|IPC_CREAT))==-1)
	  perror("semget");
 arg.val=1;
 if(semctl(sid,0,SETVAL,arg)==-1)
	  perror("semctl");
 return(sid);
}

main(){
	char *segaddr;
	int segid,sid1,sid2;
	/* 创建共享内存段 */
	if((segid=shmget(SHMKEY,SIZE,IPC_CREAT|0666))==-1)perror("shmget");
	/* 将共享内存映射到进程数据空间 */
	segaddr=shmat(segid,0,0);
	sid1=creatsem(SEMKEY1);
	/* 创建两个信号量，初值为1 */
	sid2=creatsem(SEMKEY2);
	semWait(sid2);
	/* 置信号量2值为0，表示缓冲区空 */
	if(!fork())
	while(1){           /* 子进程，接收和输出 */
		 semWait(sid2);
		 printf("Received from Parent: %s\n",segaddr);
		 semSignal(sid1);
	}
	while(1) {              /* 父进程，输入和存储 */
		semWait(sid1);
		scanf("%s",segaddr);
		semSignal(sid2);
	}
}


```
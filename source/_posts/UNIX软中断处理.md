---
title: UNIX软中断处理
date: 2017-04-23 00:35:17
tags: UNIX
---

## Code:

``` c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <signal.h>
//父子进程通信
main(){
  int pid,status=1;
  void func();
  signal(SIGUSR1,func);
  while((pid=fork())==-1);
  if(pid){
   //父进程
   printf("Parent Process.\n");
   printf("Parent send signal.\n");
   kill(pid,SIGUSR1);
   pid=wait(&status);
   printf("Child process=%d,status=%d\n",pid,status);
  }else{
   //子进程
    sleep(2);
    printf("Child process.\n");
    execl("/bin/ls","ls","-l",(char*)0); /* 映象改换 */
    printf("execl  error.\n");           /* 映象改换失败 */
    exit(2);
  }
  printf("Program Finshed.Pid=%d.\n",pid);
}
void func(){
 //print current date.
 system("date");
}
```
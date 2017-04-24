---
title: UNIX进程通信之消息通信
date: 2017-04-23 00:42:54
tags: UNIX
---
## Code:

### msgcom.h
``` c
#include <stdio.h>
#include <errno.h>
#include <sys/types.h> //类型头文件
#include <sys/ipc.h>   //通信头文件
#include <sys/msg.h>   //消息头文件
#define MSGKEY 74123
struct msgtype{
 long mtype;
 int text;
};

```

### msgserver.c
``` c
#include "msgcom.h"
main(){
    struct msgtype buf;
    int qid,pid;
    //MSGKEY,Access Control Premission
    qid=msgget(MSGKEY,IPC_CREAT|0666);
    buf.mtype=1;
    buf.text=pid=getpid();
    msgsnd(qid,&buf,sizeof(buf.text),IPC_NOWAIT|04000);
    msgrcv(qid,&buf,512,pid,MSG_NOERROR);
    printf("Request recived a message from server,type is :%d\n",buf.mtype);
}
```

### msgclient.c
``` c
#include "msgcom.h"
main(){
    struct msgtype buf;
    int qid;
    if((qid=msgget(MSGKEY,IPC_CREAT|0666))==-1)
        return (-1);
    while(1){
      msgrcv(qid,&buf,512,1,MSG_NOERROR);
      printf("Server receive a request from process:%d\n",buf.text);
      buf.mtype=buf.text;
      msgsnd(qid,&buf,sizeof(int),IPC_NOWAIT|04000);
    }
}
```




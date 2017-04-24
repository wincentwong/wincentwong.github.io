---
title: UNIX进程通信之远程进程间通信
date: 2017-04-23 17:23:43
tags: UNIX
---

## Code:
### sockcom.h
``` c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>

```

### sockserver.c
``` c
#include "sockcom.h"
main()
{
    int sockfd,newsockfd,length,count;
    struct sockaddr_in server;
    char buf[1024];
    /*生成套接字*/
    sockfd=socket(AF_INET,SOCK_STREAM,0);
    server.sin_family=AF_INET;  /* 构成socked名（地址）和建立联系 */
    server.sin_addr.s_addr=INADDR_ANY;
    server.sin_port=0;          /* 选择一个已释放的端口号 */
    if(bind(sockfd,(struct sockaddr *)&server,sizeof(server))<0)
        perror("bind stream socket");
    /* 获取并打印端口号 */
    length=sizeof(server);
    if(getsockname(sockfd,(struct sockaddr *)&server,&length)<0)
        perror("getting socket name");
    printf("socket port #%d\n",ntohs(server.sin_port));
    listen(sockfd,5);
    while(1)
    {
        newsockfd=accept(sockfd,(struct sockaddr *)0,(int *)0);
        if(!fork())
        {
            /*子进程*/
            close(sockfd);
            bzero(buf,sizeof(buf));
            /*调用库函数，清缓冲区*/
            if((count=recv(newsockfd,buf,sizeof(buf),0))<0)
                perror("Reading stream message");
            printf("message received: %s\n",buf);
            exit(0);
        }
        close(newsockfd);
    }
}
```

### sockclient.c
``` c
#include "sockcom.h"
main(int argc,char **argv)
{
    int sockfd;
    struct sockaddr_in server;
    struct hostent *hp,*gethostbyname();
    char msg[1024];
    sockfd=socket(AF_INET,SOCK_STREAM,0);
    /* 与由命令行参数指定的主机建立连接 */
    hp=gethostbyname(argv[1]);
    server.sin_family = AF_INET;
    bcopy((char *)hp->h_addr,(char *)&server.sin_addr.s_addr,hp->h_length);
    server.sin_port=htons(atoi(argv[2]));
    connect(sockfd,(struct sockaddr *)&server,sizeof(server));
    while(1)
    {
        printf("Enter send message:");
        scanf("%s",msg);
        if(!strlen(msg)) break;
        if(send(sockfd,msg,strlen(msg),0)<0)
            perror("sendint message");
        bzero(msg,sizeof(msg));
    }
    printf("EOF...disconnect\n");
    close(sockfd);
    exit(0);
}
```
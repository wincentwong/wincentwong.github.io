---
title: UNIX线程操作
date: 2017-04-23 00:43:34
tags: UNIX
---

## Code:

``` c
#include <pthread.h>
#include <stdio.h >
#include <stdlib.h>
/*Linux Complie Command: gcc thread.c -o thread -plthread  */
int nthreads = 1;

void *dowork (void *params)
{
    int j = *(int *)params;
    int term = (j+1)*(j+1);
    *(int *)params = term;
    printf("the thread [%d]: term =%d\n",j,term);
}

void main(int argc, char **argv)
{
    int i;
    pthread_t threads[100];
    int pthread_data[100];
    int sum = 0;
    if (argc==2)
        nthreads = atoi (argv[1]);
    for (i=0; i<nthreads; i++)
    {
        pthread_data[i]=i ;
        pthread_create(&threads[i], NULL, dowork,&pthread_data[i]);
    }
    for (i=0; i<nthreads; i++)
    {
        pthread_join(threads [i], NULL);
        sum+=pthread_data[i];
    }
    printf("The Sum = %d\n", sum);
}
```

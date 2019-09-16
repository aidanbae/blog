---
title: "공유 메모리를 활용한 프로세스간 통신 with C"
date: 2019-08-14T19:01:14+09:00

author: "Aidan.bae"
categories: ['Code', 'DevOps']
tags: ['공유메모리', '코어']
noSummary: false
draft: true
---

어쩌다보니 c 프로그래밍을 깊게 해야될 일이 생겼습니다.
대학교때 너무 재미없어서 이쪽은 절대 안한다고 했었는데... 컴퓨터를 알아갈수록 로우한 프로그래밍 지식이 필요해지는 순간들이 자주 찾아오네요.  

쓰레드간 통신이 아닌 멀티 프로세스 프로그래밍에서 프로세스간 통신을 위해 OS단에서 제공하는 ipc라는 녀석이 있습니다.
리눅스에서 제공하는 ipc(inter process communication)는 아래 3가지입니다.

- 공유 메모리 (Shared Memory)
- 세마포어 (Semaphore)
- 메세지 큐 (Message Queue)

그 중 첫번째 공유메모리는 아직도 자주 활용되는 녀석입니다.

공유메모리는 단어 뜻에서 알 수 있듯이 하나의 프로세스에서가 아니라 여러 프로세스가 함께 사용하는 메모리를 말합니다.  
이 공유메모리(Shared Memory)를 활용하면 프로세스끼리 통신을 할 수 있으며, 같은 데이터를 공유 할 수 있습니다.

이렇게 같은 메모리 영역을 공유하기 위해서는 공유 메모리를 생성한 후에 프로세스의 자신의 영역에 첨부를 한 후에  
마치 자신의 메모리를 사용하듯 사용하면 됩니다.

자주 쓰이는 함수  
shmget (shared memory get)  
shmat (shared memory attach)  
shmdt (shared memory detach)  
 


예제

두개의 프로세스를 만들겠습니다. counter.c 라는 예제는 공유메모리에 1초마다 0부터 계속 증가하는 카운터 문자열을 공유메모리에 넣을 예정이구요.
show_counter.c에서는 공유메모리를 화면에 출력하겠습니다.


counter.c :
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/shm.h>

#define KEY_NUM     9527
#define MEM_SIZE    1024

int main(void) {

    int shm_id;
    void *shm_addr;
    int count;

     if ( -1 == ( shm_id = shmget( (key_t)KEY_NUM, MEM_SIZE, IPC_CREAT|0666))){
        printf("공유메모리 생성 실패\n");
        return -1;
     }

    if ( ( void *)-1 == ( shm_addr = shmat( shm_id, ( void *)0, 0)))
    {
       printf( "공유 메모리 첨부 실패\n");
       return -1;
    }

    count = 0;
   while( 1 )
   {
      sprintf( (char *)shm_addr, "%d", count++);       // 공유 메모리에 카운터 출력
      sleep( 1);
   }

    return 0;
}
```

show_counter.c :
```c
#include <stdio.h>      // printf()
#include <unistd.h>     // sleep()
#include <sys/ipc.h>
#include <sys/shm.h>

#define  KEY_NUM     9527
#define  MEM_SIZE    1024

int main( void)
{
   int   shm_id;
   void *shm_addr;

   if ( -1 == ( shm_id = shmget( (key_t)KEY_NUM, MEM_SIZE, IPC_CREAT|0666)))
   {
      printf( "공유 메모리 생성 실패\n");
      return -1;
   }

   if ( ( void *)-1 == ( shm_addr = shmat( shm_id, ( void *)0, 0)))
   {
      printf( "공유 메모리 첨부 실패\n");
      return -1;
   }

   while( 1 )
   {
      printf( "%s\n", (char *)shm_addr);    // 공유 메모리를 화면에 출력
      sleep( 1);
   }
   return 0;
}
```

Compile
```cassandraql
$ gcc counter.c -o counter
$ gcc show_counter.c -o show_counter
```

이렇게 되면 counter, show_counter라는 두 개의 바이너리 실행파일이 떨어지구요
순차 실행해보겠습니다.

```
ui-MacBookPro:cproject-aidan a.$ ./counter &
[1] 90724
ui-MacBookPro:cproject-aidan a.$ ./show_counter
16
17
18
19
20
21
22
23
24
//...
```

프로세스가 정상으로 뜬것을 grep으로 확인합니다.
앞서 프로세스 실행과정에서 반환되었던 프로세스 넘버 90724가 확인됩니다.

```
ui-MacBookPro:fe a.$ ps -ef | grep counter
  501 91702 56053   0  7:07PM ttys002    0:00.00 grep counter
  501 90724 46581   0 목07PM ttys003    0:00.02 ./counter
  501 91637 46581   0  7:07PM ttys003    0:00.01 ./show_counter
```

더 나아가 ipcs명령어로 우리 컴퓨터에 실행되는 공유메모리 정보:
```
ui-MacBookPro:fe a.$ ipcs -ma
IPC status from <running system> as of Thu Aug 15 19:47:37 KST 2019
T     ID     KEY        MODE       OWNER    GROUP  CREATOR   CGROUP NATTCH  SEGSZ  CPID  LPID   ATIME    DTIME    CTIME
Shared Memory:
m  65536 0x00002537 --rw-rw-rw-       a.    staff       a.    staff      2   1024  86993  90746 19:42:48 19:13:18 18:54:54
``` 




---
title: "tcmalloc"
date: 2019-04-16T09:31:27+09:00

categories: ['Server']
tags: ['tcmalloc', 'malloc', 'computer science']
author: "Aidan.bae"
noSummary: false
---

go tune memory 발표자료를 보면서 구경한 파생 지식 tcmalloc

구글 공식 문서
http://goog-perftools.sourceforge.net/doc/tcmalloc.html

구글에서 만든 TCMalloc  
보통 멀티 스레드 환경의 서버를 만들다보면 메모리 풀을 사용하게 된다.  
메모리 풀의 이점:

- 빠른메모리할당
- 메모리 단편화 감소

하나의 거대한 메모리 풀을 사용하면 단순하게 malloc을 호출해서 메모리를 할당하는 것 보다 속도가 빠르다. 메모리 풀에 따라 다르겠지만 일반적으로 볼ㄸ ㅐ 처음 프로그래밍이 실행될 때 필요한 메모리를 통째로 잡아놓고 메모리가 필요한 경우에는 잡아놓은메모리 안에서 잘라서 주기 때문입니다. 하지만 멀티 쓰레드 환경에서는 메모리 풀의 속도가 저하됩니다. 왜냐하면 여러 쓰레드에서 하나의  메ㄹ모리 풀에 접근을하게 되면 lock에 대한 코스트가 발생합니다 동시에 접근하는 쓰레드가 많을 수록 lock을 기다리는 시간이 많아지고 그러다보면 메모리를 얻기위하여 기다리는 스레드들은 일을 할 수 었습니다.

그래서 구글은 Thread별로 따로 메모리풀을 두고 관리하고
다른 쓰레드에서 접근해야하는 메모리는 전역 메모리풀을 두어 관리하고.

TCMalloc(Thread Caching malloc)은 메모리를 Thread local cache and central heap으로 나누어 관리합니다.
쓰레드 로컬케시는 32k이하의 작은 오브젝트들을 담당하여 메모리가 부족할 시에는 센트럴 힙에서 메모리를 얻어와 할당합니다.
그리고. 32k가 넘어가는 큰오브젝트들은 Central Heap 에서 4k의 페이지 단위로 나누어서 메모리 맵을 이용하여 할당합니다. 사실 단순한 Central heap은 일반적으로 사용하는 메모리 풀과 다를 바 없습니다. 하지만 TLC(Thread Local Cache)가 있음으로 해서 메모리 할당시 불필요한 동기화가 줄어들게 되고 그에 따른 lock cost가 많이 감소되어 성능이 향상됩니다.

특히 리눅스환경에서는 pthread_mutex_t가 느리기 때문에 많은 향상을 얻을수 있다.

ptmalloc에 비해 20%이상의 성능향상

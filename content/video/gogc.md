---
title: "[dotGo 2019] golang gc 튜닝"
date: 2019-04-15T09:31:27+09:00

categories: ['Code', 'Golang']
tags: ['golang', 'gogc', 'garbage collector']
youtube: "uyifh6F_7WM"
author: "Aidan.bae"
noSummary: false
---

페이스북을 기술정보를 얻는 용도로 사용하는데 평소 추종하던 홍혜종님이 공유해준 발표자료이다.  
go tune your memory라는 이름으로 dotgo 2019에서 진행된 발표
(Go 1.12 기준으로 설명)

gc언어를 사용한다면, garbage collector가 호출되는 것에 대해 알 필요가있다.
왜냐하면 gc가 호출되는 순간, 해당 프로그램은 잠깐동안의 lack상태가 되기 때문이다. 찰나의 순간이지만 게임처럼 퍼포먼스가 중요한 클라이언트, 서버는 gc호출을 튜닝해야한다.
개발자는 그러므로 gc의 잦은 호출을 막을 수 있도록 크게 두가지 행위를 할 수 있다.

첫번째는 garbage 자체가 생기지 않도록 하는 것.(ex. object pooling)  
두번째는 gc호출에 대한 파라미터를 건드는 것.(gc옵션)

웹게임 클라이언트 개발을 했던 javascript의 경우, 모든 유저들의 웹브라우저 환경에 영향을 받기 때문에 gc에 대한 옵션을 건들 수 없다. 그래서 좀더 애니메이션 프레임 호출, 오브젝트 풀링에 신경을 많이 썼었다.
하지만 서버로 사용하는 golang의 경우, build시 gogc 파라미터를 조절함으로써 메모리 할당과 해제에 관해 어느정도 gc의 호출빈도를 조절할 수 있다.


**발표핵심요약**  

 - Go의 GC에서 mark sweep만 있고 compact가 없어도 되는 이유는 메모리 할당을 런타임에서 tcmalloc기반으로 해줌
 -  Java의 gc옵션은 너무 많지만 golang은 GOGC 단 하나(simple해서 좋다)
 - GOGC는 기존 메모리 대비 얼마만큼의 메모리가 더 할당 되면 gc가 트리거 되는지에 대한 환경변수(백분율)
 - 힙의 크기가 크면 GOGC값을 낮추고
힙의 크기가 작으면 GOGC값을 높이는 게 낫다. (영상에서는 400으로함)

---

gonna talk about the specific behavior inside the gc and that could change
Importan point
Go doesn’t sort of squish down the blocks that are remaining
It doesn’t compact

Crucial point
The size that it lets it build up to the run the go runtime is choosing where that point is garbage collect is equal to where it got to at the bottom of the last garbage

go는 그때그때 그 빈틈에 맞는 메모리를 할당해줌 메모리파편화 걱정하지마세요.



**사용법**
```
//의미: 메모리가 기존 메모리 대비 150% 정도일때 gc가 트리거 되도록 하겠다.
GOGC=150 go build ~
```


**runtime/malloc.go intro** 에서 tcmalloc을 사용하는 것을 알 수 있다.

```
// Memory allocator.
//
// This was originally based on tcmalloc, but has diverged quite a bit.
// http://goog-perftools.sourceforge.net/doc/tcmalloc.html
```
[tcmalloc에 대한 구글 공식 문서](http://goog-perftools.sourceforge.net/doc/tcmalloc.html)  
[golang doc runtime malloc](https://golang.org/src/runtime/malloc.go)




gc가 자주 일어나도록 만들고 싶다면,
Gogc = 50  
gogc를 400으로 하면 gc는 조금 일어남


GOGC=off go build
off의 경우에는 gc를 사용하지 않겠다는 건데, 1회성 프로그램(한번 실행하고 종료되는 프로그램)의 경우, 유용하게 사용할 수 있다. 대신 시스템이 터질 수 있다. ㅎㅎ;

**정리**

| situation             | Action    | +             | -           |
| --------------------- | --------- | ------------- | ----------- |
| Large static datâ set | Gogc down | Smaller heap  | More cpu    |
| Tiny heap, rapid gc   | Gogc up   | Lower latency | More ram    |
| One-shot exec         | Gogc off  | Runs faster   | May explode |

**마무리**

스낵게임서버의 경우 메모리 평균사용량이 200mb였는데 이 기준으로 보면 GOGC옵션을 좀더 높여 적용하면 퍼포먼스상 좋을것 같다.  
표를 참고해서 자신의 프로젝트에 맞는 옵션을 선택하면 좋을듯하다. default는 100이다.

**참고하면 좋은자료**
  
라인 기술블로그 "Go 언어의 GC에 대해"  
https://engineering.linecorp.com/ko/blog/go-gc/  
데이브 체니 gogc  
https://dave.cheney.net/tag/gogc

**gogc말고 다른부분을 건드려 gc사이클을 튜닝한 사례**
  
최근에 세계 최대 스트리밍 서비스인 twitch에서 golang gc와 관련해서 글포스트가 레딧에 올라왔다.  
twitch도 golang을 활용해 visage라는 서비스를 운용중 ballast 변수를 만들어서 힙사이즈를 강제로 크게 조절
힙 할당량에 의해 gc사이클 주기가 컨트롤되어 cpu사이클을 줄인 예시가 있다. 4월중 가장 핫했던 글! 참고참고  
https://blog.twitch.tv/go-memory-ballast-how-i-learnt-to-stop-worrying-and-love-the-heap-26c2462549a2  
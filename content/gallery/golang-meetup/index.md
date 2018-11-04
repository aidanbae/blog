---
title: "Golang과 websocket을 활용한 서버 프로그래밍 (장애 없는 서버 런칭 도전기)"
date: 2018-07-30T09:31:27+01:00

categories: ['Code', 'Golang']
tags: ['golang', 'websocket', 'server']
author: "Aidan.bae"
noSummary: false
---

#### Intro

2018, 7월 25일 제1회 golang meetup이 열렸다.  

그간 golang korea 커뮤니티에서 눈팅으로 이득을 취한 나...
장애없이 런칭한 Golang 스낵게임서버를 개발할 때에도 수많은 golang 블로그에서 도움을 받았기때문에
이번 meetup에서 그분들을 만나뵙고 이야기를 나눌 수 있으면 좋겠다 생각해서 참가를 신청했다.
발표자가 없어 존폐위기까지 거론하셔서 발표를 하는 방향으로 결정

작은 토즈방에서 20명정도 모여서 소근소근 토론하는 자리가 될줄 알았는데 160명이나!!  
개발인생 첫 발표인데 많은 사람이 와주셔서 감사한 마음으로 준비했다.

##### GopherCon Talks

Golang Korea에서 주최하는 밋업, 세미나 및 컨퍼런스의 발표 자료들을 모아두는 저장소이다. 여기서 다른 좋은 발표자료도 함께 확인할 수 있다.  
https://github.com/golangkorea/gophercon-talks

---
<iframe src="//www.slideshare.net/slideshow/embed_code/key/IAOoU055tM9yPS" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/SangikBae/golang-websocket-109095156" title="golang과 websocket을 활용한 서버프로그래밍 - 장애없는 서버 런칭 도전기" target="_blank">golang과 websocket을 활용한 서버프로그래밍 - 장애없는 서버 런칭 도전기</a> </strong> from <strong><a href="https://www.slideshare.net/SangikBae" target="_blank">Aidan</a></strong> </div>

#### 발표 구성
발표시간 40분, 질의응답시간 10분 (총 50분)  
첫 meetup에 어울리는 주제가 무엇일까 생각했는데
`Concurrency`가 Golang의 디자인철학과 가장 밀접하다고 판단, Build나 배포 pipeline구축 경험을 공유하는 것보다
개발과정에서 Concurrency와 관련해 방황했던 경험을 공유하는 쪽으로 가닥을 잡았다.
그리고 마무리는 `websocket을 활용한 demo`를 보여주는 시나리오를 구성했다.

---

#### 죽지않는 서버를 위해 거시적인 시스템 구성

**1. 단위테스트**

혼자인데다 시간에 쫓겼기때문에 단위테스트에 신경을 많이 못썼다. Join처리라던지 중요한 프로세스만 단위테스트 파일을 유지했다. race옵션은 경합이 일어날 것 같은 프로세스 테스트에 자주 활용했다. git push 이전에 꼭 돌려봤다.

**2. GoCD**

빌드 배포파이프라인을 미리 짜두었다. agent를 손 볼 때면 가끔은 플러그인이 많은 Jenkins가 부럽지만 UI가 이뻐서 좋다.

**3. Mesos-marathon**

플랜 b가 있으면 플랜 c도 만들어라.
서버 프로그래머라면 장애라는 늘 최악의 상황을 대비해야한다.  
메소스마라톤의 페일오버전략은 환상적이다.

**4. ngrinder 부하테스트**

5분 10분이라고 ppt에 적어두었지만 다양한 시간대별로 체크를했다. 주말내내 돌려본적도있다. 테스트코드가 잘못되어있다면 잘못된 원인진단으로 이어지고 오히려 잘 쌓은 탑을 엎어버리게 된다. 생각보다 많은 삽질을 했다. 테스트코드의 중요성을 크다. 기능을 추가할 때마다 조금씩 업데이트를 했다.

**5. 메모리 고루틴 회수 체크**

GC언어로 손쉽게 프로그램을 짠만큼 자원회수확인은 꼭 확인해야할 중요한 부분이다. admin페이지는 간단하게 생성된 방의 갯수, 접속자수, 고루틴갯수, 메모리사용량 등을 표시했다. 그리고 대기방 정보 리스트를 지속적으로 갱신해서 혹시나 오랜 시간동안 매칭되지않는 방이 있는지도 확인할 수 있도록했다. 화면에 보이는 것 자체가 데이터인 MVVM, VueJS 좋다! 부하테스트를 정해진 시간에 따라 종료하다보면 수많은 가상유저가 동시에 접속을 끊게 된다. 이 경우 방이 파괴되지않고 남아있는 경우가 있었다.


**이 거시적인 프로세스를 개발초부터 갖추고 시작하진 않았다.** 개발기간동안 프로세스도 지속적으로 완성해가면서 프로젝트를 진행했다. 그래서 초기엔 잘못된 테스트 결과를 믿고 쌓았던 탑을 엎는 경우도 있었다. 특히 ngrinder 테스트코드는 goroutine돌리듯 백그라운드에서 계속 보강했다. 여담으로 작업시기가 봄이었기때문에 이전에 만들어놨던 클라이언트 게임 벚꽃에디션 작업까지 진행해야했다. 일을 조금씩 미루다가 디자인팀에서 '이러다 벚꽃 다지겠다'고 간곡한 요청이 와서 하루에 5개 언어를 만지작거렸다. 내가 지금 만지고 있는 이 배열이 golang slice인지 python인지 java arraylist인지 javascript인지 그날 밤은 동시성보다 병렬성이 필요한 밤이었다. 부사수가 있으면 좋겠다는 생각이 처음 들었다. 개발에 들어간지 4주차쯤에는 이 프로세스가 거의 완성되어서 편리하게 개발했다. 시간적 여유가 되는 팀이라면 구축해놓고 시작하는 게 좋은 거 같다.

---
#### Golang을 어떻게 사용할까
고린이가 golang에서 방황했던 이야기를 중심으로
Golang의 생각을 바꾸는 과정을 이야기했다.


##### 1) 적당한 방법론을 택하자 (mutex vs channel)

초기엔 뮤텍스에 의존을 많이했었다. 퍼포먼스도 잘나왔고 부하테스트도 잘 견뎠었는데 시간이 지나면 알수없는 에러들이 툭툭 튀어나왔다. 채널 중심으로 바꾸면서 안정성이 올라갔다.

다음은 내 선택들의 근거가 되었던 이야기이다. 천천히 풀어보겠다.
발표 예제에 쓰인 뮤텍스 vs 채널 성능비교한 코드이며
아래는 뮤텍스로 동기화를 시도해본 예제코드이다.
```golang
func main() {
	runtime.GOMAXPROCS(runtime.NumCPU())
	var data = []int{}
	var mutex = new(sync.Mutex)

	go func() {
		for k := 0; k< 1000000; k++ {
			mutex.Lock()
			data = append(data, k)
			mutex.Unlock()
		}
	}()

	go func() {
		for k := 0; k< 1000000; k++ {
			mutex.Lock()
			data = append(data, k)
			mutex.Unlock()
		}
	}()

	go func() {
		for k := 0; k< 1000000; k++ {
			mutex.Lock()
			data = append(data, k)
			mutex.Unlock()
		}
	}()

	go func() {
		for k := 0; k< 1000000; k++ {
			mutex.Lock()
			data = append(data, k)
			mutex.Unlock()
		}
	}()

	time.Sleep(500 * time.Millisecond)
	fmt.Println(len(data))
}
```
- runtime.GOMAXPROCS(cpu갯수)를 이용해 모든 cpu를 다 사용한다.  
(go 1.9버전기준 default는 max이기때문에 안해줘도 상관없다)
- data라는 int형 슬라이스를 만든다.
- data slice에 특정값 k를 append하는 작업을 mutex로 보호한다.
- 이 작업을 100만번씩 4개의 고루틴을 실행한다.
- 500ms가 지난 후 data slice 길이를 확인한다.

 pc사양에 따라 결과가 다를 수 있지만 내 맥북에서는 **400만개가 모두 append되었다.**

아래 코드는 위와 같은 작업을 Unbuffered channel로 동기화한 예제코드:
```golang
func main() {
	runtime.GOMAXPROCS(runtime.NumCPU())
	var data = []int{}

	var c = make(chan int)

	go func() {
		for k := 0; k< 1000000; k++ {
			<-c
			data = append(data, k)
			c <- 1
		}
	}()

	go func() {
		for k := 0; k< 1000000; k++ {
			<-c
			data = append(data, k)
			c <- 1
		}
	}()

	go func() {
		for k := 0; k< 1000000; k++ {
			<-c
			data = append(data, k)
			c <- 1
		}
	}()

	go func() {
		for k := 0; k< 1000000; k++ {
			<-c
			data = append(data, k)
			c <- 1
		}
	}()

	c <-1

	time.Sleep(500 * time.Millisecond)
	fmt.Println(len(data))
}
```
- Unbuffered채널은 동기채널로 값을 들어오기전까지 블롹상태가 된다.
- 값이 수신하게된 녀석은 작업을 할수있는 제어권을 획득한 것으로 간주하고 append작업을 수행한다.
- append작업 이후에 1값을 채널로 넘겨줘서 다른녀석이 작업을 하도록 한다.

500ms 뒤에 결과값을 확인해보니 거의 **230만 ~ 240만** 정도의 결과가 나왔다.  
mutex에서 400만의 결과를 봤기에 눈이 높아진 나에게 mutex는 선택할수밖에없는 방법론이었다.


시간이라는 변수는 서버에서 중요하다.  
위 예제코드들은 500ms라는 짧은 시간동안 측정되었기때문에 순간적인 퍼포먼스만 측정되었지 안정성은 측정할 수 없었다. 서버는 long-running 프로세스이기 때문에 그 시간이라는 변수를 생각하게 된건
패닉메세지를 받아본 뒤였다.
정확한 원인을 알 수 없는 데드락을 만나면 내 머리도 패닉.

데드락때문에 골머리를 앓을 때쯤 주말에 카페에서 책을 보았는데 **미래를 품은 7가지 동시성모델** 책이었다. 그 책은 데팍(팀장)이 내가 개발 인턴일 때 사준 책인데 미래를 보셨는지 약 1년 반이 지나서 먼지를 털어냈다. 당시 맡은 실무는 HTML5게임이어서 싱글스레드 프로그래밍이었기에 동시성모델이 와닿지 않았다. 근데 그냥 그때는 배워보겠다고 우겨넣듯이 조금씩 읽어둔게 좌표가 되었다. 자바의 롹시스템 문제점 그리고 뮤텍스나 golang에 채널에 대한 이야기도 있었던 게 기억이 나서 다시 꺼내들었다. 이 책에서는 뮤텍스의 문제점들을 정확하게 짚어주었고 함수형 사고, 액터모델, CSP를 괜찮은 접근방식으로 추켜새웠다. 내가 골머리를 앓고있는 가변상태에 대해 부정적으로 이야기하면서 미래는 불변성이 지배할 것이라는 멋진 말이 적혀있다.

쌓았던 탑을 부시고 다시 채널 위주로 수정했다.
중간중간 팀장님한테 내 생각의 변화를 브리핑했고 컨펌이 떨어진뒤 작업을 진행했다. 최대한 테스트까지 돌리고나서 근거를 들고 이야기했기에 코어작업을 뒤엎는 이야기도 쉽게 진행되었다. 모든 의사결정에 있어서 진보와 보수의 균형은 꽤나 중요하다. 우린 서로 극단적이지 않았기때문에 적당한 타협점을 계속해서 매끄럽게 잘 찾았다. 에버노트에 모든 삽질과 그 결과를 스크린샷과 함께 꾸준히 기록한게 큰 도움이 된거같다.

채널중심으로 수정하고나니 안정성이 늘었다. 데드락이나 패닉 디버깅은 채널 중심으로 살펴보게 되어서 좀더 디버깅이 수월했다.

이쯤에서 좋은 내용이 담긴 유튜브 영상들을 소개한다.
golang을 만든 구글의 생각을 엿볼수있다.
[Concurrency is not Parallelism - Rob pike](https://www.youtube.com/watch?v=cN_DpYBzKso)  
[Google I/O 2013 - Advanced Go Concurrency Patterns](https://www.youtube.com/watch?v=QDDwwePbDtw)

아직 미완이지만 채널과 관련해 쓴 포스트도 공유한다.  
[Golang 채널 중심 프로그래밍](/code/golang/fib)

<br>
##### 2) 나의 실수를 얼른 찾자.(leak찾기, handlePanic)

**goroutine leak**

자원회수는 중요한 이슈이다. 불필요한 고루틴이 정해진 루틴에 회수되지않고 계속 돌고있으면 이는 시간이 지나서 큰 문제가 될 수 있다. 이런 회수되지 않는 고루틴을 goroutine leak이라고 부른다.
채널 중심으로 코딩을 하다보면 Goroutine leak을 찾기 편해진다.

뮤텍스에 의한 문제를 제외하고  
채널에 의해 동기화를 할때 goroutine leak은 두 부분을 중심으로 찾으면 된다.  

- Sender가 있는데, Receiver가 없다.
- Receiver가 있는데, Sender가 없다.

발표 때 제한된 시간관계상 자세히 다루지 않았지만 여기서는 간단한 예제로 두 케이스를 살펴보겠다.
Receiver가 없는 경우 :
```golang
func main () {
	s := make(chan int)

	// Sender (값을 보내는 녀석)
	s <- 3

	go func() {
		// Receiver (값을 받는 녀석)
		f := <-s
		fmt.Println(f)
	}()

	fmt.Println("보고싶다 이 로그")
	time.Sleep(3 * time.Second)
}
```

결과 :
![result](/gallery/screenshot.png)
3초 뒤에 메인프로세스를 종료되게 했음에도 불구하고
Sender부분에서 에러가 발생한다.
all goroutines are sleep - deadlock

Receiver가 goroutine으로 호출되었고 심지어 먼저실행도 되지않았기때문에
Golang의 메인프로세스는 보내는 시점에서 무한대기를 타게된다.

반대의 케이스도 보겠다:
```golang
func main () {
	s := make(chan int)

  // Receiver (값을 받는 녀석)
	f := <-s
	fmt.Println(f)

	go func() {
		// Sender (값을 보내는 녀석)
		s <- 3
	}()

	fmt.Println("보고싶다 이 로그")
	time.Sleep(3 * time.Second)
}
```
결과 :
![result2](/gallery/screenshot3.png)


이제 에이포용지를 펼치고 고루틴들의 관계도를 그린다.
채널을 중심으로 이때 받는녀석이 없나? 있나? 등의 짱구를 굴리면
여러분은 이런 릭포인트들을 최대한 줄일 수 있다.
Runtime의 trace 패키지를 통해 고루틴간 프로파일링을 쉽게 할수 있다는 정보를 접했다.
써보고 자세히 알게되면 포스팅할 예정이다.

**HandlePanic**  

defer와 recover메소드  
지연호출(defer)은 함수가 끝나는 시점에서 호출되는 코드이다.  
또 스택처럼 쌓이기때문에 defer문의 순서에 따라 호출순서를 조작할수 있다.  
Recover메소드는 panic의 상황에서 서버를 복구시켜준다.  
이 두녀석의 콜라보로 우린 서버다운이라는 심각한 상황을 막을 수 있다.

HandlePanic은 방어로직으로 앞서 이야기한 leak가능성이 있는 함수에 지연호출(defer)로 심어놨었다.
최악의 상황에 panic이 날 경우 서버를 recover하고 해당 에러스택을 카카오톡방으로 쏘도록 챗봇을 연동했다. 초기에 Mutex로 핵심로직을 짜고 부하테스트를 주말동안 켜놓은적이있는데
패닉메세지를 받았다. 팀장님도 같이있는 카톡방이어서 난처할 따름. 침착하게 '테스트입니다'로 일관했다.

<br>
#### Websocket Demo

https://github.com/aidanbae/websocket-example

이하 자세한 이야기는 SlideShare에서 확인할 수 있다.

---

#### 결과

위 내용에 의해 런칭 후 지금까지 한 번의 문제없이 서버는 잘 돌아가고있다.

#### 요약
 - Mutex 신용카드같아 너무 많이 사용하면 나중에 감당할 수 없는 값을 치를걸
 - share memory by Communicating (Channel을 잘 활용해보자)
 - 생산성, 퍼포먼스, 안정성 다 만족스럽다.

#### 받은 질문
```
Q: mutex가 안정적이지 않다는 말씀이신데 그럼 golang의 sync패키지에서 왜 mutex 지원하는거에요?
A: mutex는 전통적인 방법, 앞서 말씀드린거처럼 하드웨어쪽과 가깝기때문에 퍼포먼스측면에서 좋다.
그래서 제거할 이유도 없다고 생각한다. 나보다 구글이 대답을 잘할거같다. Go blog FAQ를 보시면 그 대답에 도움이 될 것 같다.

Q: Log System은 어떻게 관리하셨나요?
A: 예전에는 log rotation을 사용했는데, 마라톤으로 옮겨온뒤 json으로 쏴주기만한다.  
차기프로젝트에서는 logrus라이브러리를 사용할 것 같다.

Q: websocket 1006 error뭐에요?
A: 만나본거같긴한데 기억이 잘 안난다.

Q: 생산성이 좋다는건 다른언어에 비해 좋다는 건가요?
A: C++같은 고성능 언어로 서버를 만들어본적이 없어서 잘모르지만 제 기준 짧은 기간동안 한번의 장애도 없이
서비스했기때문에 높다고 생각한다.

Q: 로드밸런싱은 어떻게 한건가요?
A: 그냥 서버에 인원제한 걸어두고 만약 인원이 다찼다면 다른서버 가시라고 말해줬다.
```

<br>
#### 후기

부족한 발표를 다들 초롱초롱한 눈으로 들어주셔서 감사했다.  
맨날 반팔만 입다가 셔츠를 입으니 엄청더웠다.
golang korea에서 댓글 토론등을 보면서 내 생각을 정립했었기때문에 네임드분들을 만나뵙고 이야기를 해보고싶었는데 생각보다 세미나형식으로 진행되었다. 규모가 커지면 어쩔 수 없는 거 같다.  
갑자기 규모가 커진 밋업을 준비하느라 고생한 운영진분들께 감사하다.  

golang 디자인패턴을 같이 스터디했던 카카오 ai팀 사람들,
SOPT동아리를 같이 활동했던 지수랑 혜진,
토스로 이직한 Mason, Rebecca 반가운 얼굴들을 다시 볼 수 있어 좋았다.

발표준비를 하면서 발표자들의 땀을 알게되었다. 생각보다 많은 노력이 들어간 핵심자료들이 세상엔 많다. 그래서 요즘 빠르게 훑어보고 넘겼던 NDC발표들을 다시 천천히 보고있다.

---
title: "Golang 채널 중심 프로그래밍"
date: 2018-08-11T09:31:27+01:00

categories: ['Code', 'Golang']
tags: ['golang', 'algorithm']
author: "Aidan.bae"
noSummary: false
---

"사실 모든 학문은 연결되어있다. 사회학도 역사도 프로그래밍도"
#### Intro

사회학에서 산업혁명 하면 떠오르는 키워드는 `Division of labour`이다.
즉, 한국어로 `분업`이다. 일을 처리하는데 분업에 따른 전문성 증가, 기계와 같은 파이프라인 등은 우리 사회에 큰 영향을 미쳤다.
이전에 없던 엄청난 생산량(공장 시스템)에 사회는 급성장한다.  
빠른 성장만큼 노동문제와 같은 각종 사회문제가 터져나왔고 그런 생산량이 전투력에 악용되어 제2차 세계대전에서는 수많은 인류의 죽음을 우린 목격해야했다.

![찰리채플린과 모던타임즈](/code/golang/fib/chalie.png)
<p align="center">_찰리채플린과 Modern times(산업혁명 블랙코미디 영화)_</p>

프로그래밍에서도 cpu가 늘고 분업과 관련된 동시성 프로그래밍들이 발전하면서 lock기법이 발전했고
race condition, deadlock등 다양한 문제들이 속출했다.  

Golang으로 멀티쓰레드 프로그래밍을 쉽게 접하게되면서
컴퓨터라는 세계에 있어서 이놈의 분업을 큰 문제 없이 잘 활용할 수 있게 되었다.  
당신과 나의 맥북의 소장된 **8개의 CPU** 는 현대식 합법적 노예다.(물론 더 적을 수도 있어요~)

Golang을 1년 정도 만지면서 느낀 핵심포인트는
`채널 중심의 생각전환`이다.
저번에 아는동생이 kakao 무슨TF에서 golang을 쓰고있어서 살짝 보았는데
Golang의 특성을 죽이고 자바쓰듯이 사용하고 있어서 아쉬운마음이 컸다. 물론 결과물만 잘나오면 된다고들 하지만 한국의 고퍼들에게 좀더 재미있게 코딩할 수 있도록 도움을 주고싶어 이렇게 글을 남긴다.

---
#### Why

채널중심의 생각전환으로 지루하던 코딩을 재미있게

---

#### Prerequisite
다음과 같은 수준의 개발자를 예상독자로 생각하고 글을 적었습니다.  

- golang 관련 책을 한권 이상 읽어봤다.  
- golang에서 뜻하는 channel을 알고 있다.

사람마다 디자인 설계패턴이나 프로그래밍 철학이 있지만  
golang을 접하는 순간, **자신의 철학을 조금은 버려두길 권한다.**  
천천히 따라와 보시라.

#### Basic

golang은 **CSP(Communicating Sequential Process) 수학이론** 을 기반으로 발전된 언어이다. 순차 프로세스 통신이라고 불리는 csp는 프로세스간의 통신을 동시성프로그래밍의 핵심으로 보고있다.
golang은 이 통신의 통로이자 수단으로 `channel`이라는 개념을 추가한다.
채널은 golang에서 `일급 객체`로 값처럼 전달할 수 있다.

![result1](/code/golang/fib/screenshot.png)

즉, 함수형 프로그래밍에서 순수함수를 마치 값처럼 사용한 것처럼
고랭에서는 채널을 값으로 전달할 수 있다! 고랭은 채널을 특별히 아낀다. range, close 등만 봐도 알 수 있다.  

다음은 channel에 들어온 데이터를 range로 뽑아내는 간단한 프로그램이다:

```golang
func main () {
	input := make(chan int)

	go func () {
		for data := range input {
			fmt.Println("고루틴에 들어온 녀석을 처리했다 :", data)
		}
		fmt.Println("여기는 언제 찍힐까요?")
	}()

	input <- 5
	input <- 10
	input <- 12

	time.Sleep(1 * time.Second)
}
```

- input이라는 int형 채널을 만든다.
- 고루틴에서 input이라는 채널을 range문으로 뽑아내면서 반복문을 돌린다.
이 때 range문은 input이 drained상태가 될때까지 실행된다.(보통은 close된 상태) 반복문이 끝나면 "여기는 언제 찍힐까요" 로그를 찍도록한다.
- 1초 슬립을 걸어서 메인프로세스의 다운을 늦춘다.(그저 확인용)

결과:  

![result2](/code/golang/fib/screenshot2.png)
"여기는 언제 찍힐까요"라는 로그는 찍히지 않았다.
메인프로세스가 죽는 1초후까지 해당 고루틴의 프로세스 포인트는 range문 안에서 살아 있었기 때문이다. range문의 종료는 input채널이 drained상태일때 발생한다. (쉽게 생각해서 closed)

그러면 input 채널에 12라는 값을 넣기 전에
close를 해보자.

```golang
func main () {
	input := make(chan int)

	go func () {
		for data := range input {
			fmt.Println("고루틴에 들어온 녀석을 처리했다 :", data)
		}
		fmt.Println("여기는 언제 찍힐까요?")
	}()

	input <- 5
	input <- 10
	close(input)
	input <- 12

	time.Sleep(1 * time.Second)
}
```

결과:

![result3](/code/golang/fib/screenshot3.png)
5와 10이라는 값이 찍힌 후,  
"여기는 언제 찍힐까요"라는 로그가 찍혔다.
input채널을 닫아 range문을 빠져나온 고루틴의 행적을 알 수 있다.
panic 메세지는 닫힌 채널에 데이터를 넣었다는 메세지를 뿜어내며
프로그램을 종료시켰다.

이제 우리는 이 range와 channel을 활용해 간단한 알고리즘을 풀어보자.  

---
#### Fibonacci 수열

간단하게 기초 알고리즘 중 하나인 `피보나치 수열`을 해보자.  
다음과 같은 점화식으로 피보나치 수열을 정의할 수 있다.  

![result4](/code/golang/fib/screenshot4.png)

> 제0항을 0, 제1항을 1로 두고, 둘째 번 항부터는 바로 앞의 두 수를 더한 수로 놓는다. 1번째 수를 1로, 2번째 수도 1로 놓고, 3번째 수부터는 바로 앞의 두 수를 더한 수로 정의하는 게 좀더 흔하게 알려져 있는 피보나치 수열이다. 이 둘 사이에는 시작점이 다르다는 정도를 빼면 사실상 동일하다.


해당 수 이하의 피보나치 수들을 보여주는 함수를 만들어보자:
```golang
func fib(n int) chan int {
	c := make(chan int)
	go func(){
		for i, j:= 0,1; i < n; i,j = i+j,i {
			c <- i
		}
		close(c)
	}()
	return c
}
```
- fib라는 이름의 함수는 `n값`을 인자로 받고 `int형 채널`을 결과로 반환하는 함수이다.
- 함수 내부에서 채널을 만들고 피보나치 작업을 수행할 고루틴을 만들어낸다.
- for문에서 해당 점화식을 해결하고 결과값을 c라는 채널에 전달한다.
- 작업이 다끝나면 해당 채널을 닫는다.

메인 프로세스 :
```golang
func main() {
	for i := range fib(1000) {
		fmt.Println("피보나치 수 :",i)
	}
}
```
메인 프로세스에서는 `fib함수`가 반환해주는 채널을 range문으로 돌린다.
close될 때까지 프로세스포인트는 range문을 돈다.

결과:

![result5](/code/golang/fib/screenshot5.png)
How beautiful!
1000 이하의 피보나치 수가 쭈루룩 이쁘게 찍힌 모습을 볼 수 있다.  

#### Point
fib함수를 다시 유의해서 봐보자.

1. fibonacci라는 task의 결과물 방출을 위한 out채널을 만들고
2. 해당 task작업을 하는 고루틴을 실행시키고
3. 작업 종류 후 out채널을 닫는다.

이것이 하나의 작업 파이프이다.

이제 거대한 분업 시스템을 구축하기 위한 채널 중심 프로그래밍이 여러분의 머릿속에 각인되었다.  
다음시간에는 필자는 이쯤에서 만족하지않고 여러분을 더 독려해보겠다.  
이런 작업 파이프들을 연결해서 현대판 공장 파이프라인을 구축해보는 것이다. 두근거리죠?제발ㅇ_ㅇ

#### PostScript
이렇게 말해놓고 간단한 함수를 짤 때 나도 slice를 애용한다ㅎㅎ... 습관이 참 무섭다.

---
##### next: [Pipeline Pattern](https://aidanbae.github.io/code/golang-design/pipeline)

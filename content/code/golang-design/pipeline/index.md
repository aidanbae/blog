---
title: "Golang Design Pattern - Pipeline"
date: 2018-08-12T09:31:27+01:00

categories: ['Code', 'Golang']
tags: ['golang', 'design pattern']
author: "Aidan.bae"
noSummary: false
---

저번 시간에 채널 중심의 생각전환으로 하나의 작업 파이프를 만드는 방법을 배웠습니다.  
이번엔 여러가지 작업 파이프들을 이어서 하나의 작업 파이프라인을 구축해봅시다.  
혹시 이전 채널 중심 프로그래밍이라는 포스트를 놓치셨다면 보고오시길 추천드립니다.


#### Why

작업 파이프라인을 구축해서 고루틴과 채널 활용을 극대화합니다.  


![sc](/code/golang-design/pipeline/screenshot.png)
---

#### What

어떤 작업 파이프라인을 구성해볼까요?  
이해하기 쉬운 설명을 위해 쉬운 작업들 위주로 예제를 구성해보았습니다.

>- 0에서 99까지 랜덤한 수를 특정 횟수만큼 만들어내는 task
- 해당 랜덤한 수를 출력하는 task
- 숫자들을 제곱하는 task
- 마지막으로 그 제곱된 수들을 모두 더하는 task

총 4개의 작업이 있습니다. 1번부터 3번까지 파이프라인을 구축해보겠습니다.
4번 작업은 메인 프로세스에서 진행할 것이어서 파이프라인을 굳이 만들지 않았습니다.

---
#### Make Pipe Function
해야될 작업들을 분명하게 쪼갰으니 이제 코딩을 시작해야겠죠.
우선 0에서 99까지 랜덤한 수를 특정 횐수만큼 만들어내는 파이프 함수를 구축해봅시다:  
```golang
func factoryRand(count int) (out chan int) {
	out = make(chan int)
	go func() {
		for i := 0; i < count; i ++ {
			out <- rand.Intn(100)
		}
		close(out)
	}()
	return out
}
```
- 인자로 특정 횟수값을 받고 결과로 int형 채널을 반환합니다.  
- rand는 내장 라이브러리인 [math/rand](https://golang.org/pkg/math/rand/)를 사용했습니다.  
- rand.Intn(n)은 **0 <= n < 100** 사이의 interger형 난수를 반환합니다.

이제 해당 수를 출력하는 파이프함수를 만들어보죠:
```golang
func printNumber(in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		for n := range in {
			fmt.Println("print: ", n)
			out <- n
		}
		close(out)
	}()
	return out
}
```
- `<-chan`은 받기 전용 채널을 의미합니다.(일종의 제약을 걸어두는 역할입니다.) 만약 해당 채널에 값을 보내는 행위를 한다면 컴파일 에러가 발생합니다.
- 인자값으로 받은 채널을 range문을 통해 뽑아내면서 받은 값들을 `fmt.Println`로 찍어냅니다.

제곱하는 함수입니다:
```golang
func squareNumber(in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		for n := range in {
			out <- n * n
		}
		close(out)
	}()
	return out
}
```
- 제곱된 결과값을 out 결과채널로 전송하고 자신의 역할을 마칩니다.

최종적으로 메인 프로세스에서 해당 값들을 더합니다.
메인 프로세스에서는 성능 측정까지 하기위해 작업시간을 구하는 로직도 추가했습니다.
파이프 함수들에서 나오는 채널들을 이어봅니다.
```golang
func main() {
	// 작업시작시간 기록
	start := time.Now()

	c0 := factoryRand(100000)
	c1 := printNumber(c0)
	c2 := squareNumber(c1)

	// 최종적으로 더하는 작업을 메인 프로세스에서 진행합니다
	sum := 0
	for n := range c2 {
		sum += n
	}
	fmt.Printf("Total Sum of Squares: %d\n", sum)

	// 작업 종료 후 시간기록
	elapsed := time.Since(start)
	fmt.Println("작업소요시간: ", elapsed)
}
```
- factoryRand의 인자값으로 10만을 줍니다. factoryRand는 10만 개의 랜덤 숫자를 만들어냅니다.
- factoryRand에서 추출된 out채널 c0을 printNumber에 인자로 전달합니다.
- printNumber에서 추출된 out채널 c1을 squareNumber에 인자로 전달합니다.
- 최종적으로 전달된 c2라는 채널에서 데이터를 뽑아내 더하는 작업을 진행합니다.

결과:

![result](/code/golang-design/pipeline/screenshot2.png)

10만개의 랜덤한 숫자 생성, 프린트, 제곱, 더하기 이 4가지 작업을 하는데 223ms!
좋다. 제곱해서 모두 더한값은 3억이 조금 넘게 나왔네요.

이제 마지막으로 리팩토링해보겠습니다.  
모든 함수들이 채널을 결과값으로 반환하기 때문에!
요렇게 할당없이 바로바로 전달할 수 있습니다:
```golang
func main() {
	// 작업시작시간 기록
	start := time.Now()

	sum := 0
	for n := range squareNumber(printNumber(factoryRand(100000))) {
		sum += n
	}
	fmt.Printf("Total Sum of Squares: %d\n", sum)

	// 작업 종료 후 시간기록
	elapsed := time.Since(start)
	fmt.Println("작업소요시간: ", elapsed)
}
```

---
#### Summary

만들고 싶은게 있어요?  

    우선 작업(Task)를 나누세요.  
    goroutine에게 작업을 시키고 결과를 채널로 공유하세요  


#### PostScript
따라오시느라 수고하셨습니다.  
나름 이해하기 쉽게 설명한다고 구성했는데 어떻게 느껴졌을지 모르겠네요.  
다음 시간에는 이 파이프라인에 fan-in fan-out패턴을 적용해보겠습니다.

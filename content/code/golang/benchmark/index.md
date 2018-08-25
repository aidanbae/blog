---
title: "Golang Benchmark Test 사용하기"
date: 2018-05-13T09:31:27+01:00

categories: ['Golang']
tags: ['benchmark test', 'golang']
author: "Aidan.bae"
noSummary: false
---

#### Intro

코딩을 하다보면 내가 짠 함수의 성능이 좋은지 안좋은지 궁금할때가 많다.
Golang에서는 testing 라이브러리를 기본적으로 제공하면서 유닛테스트를 지원하면서도 성능을 측정할 수 있는 벤치마크 테스트도 제공한다.
게임 쪽을 개발하다보면 최적화의 과정이 필요하기 때문에 benchmark test는 유용하게 사용된다.

---
#### Why
golang benchmark test 사용법을 익혀 자신 스스로와 팀원들을 설득할 수 있다.

---
#### Usage

간단하게 두 수를 더하는 함수를 짜고 성능을 측정해보겠다.
```golang
package main

import (
	"testing"
)

func Sum(a, b int) int {
	return a + b
}

func BenchmarkSum (b *testing.B) {
	for i:= 0; i< b.N; i++ {
		Sum(1, 2)
	}
}
```

벤치마크를 수행하는 함수는 몇몇 규칙을 꼭 지켜야한다.

- Test함수는 Benchmark로 시작하는 이름을 가진다.
- Benchmark뒤에는 대문자가 옵니다. ex) BenchmarkSum
- *testing.B 타입의 매개 변수를 받는다.

테스트는 b.N에 정의된 값 만큼 for 반복문을 실행한다.  
반복문 안에서는 성능을 측정할 함수를 호출하면 된다.  

테스트 명령문은 해당 프로젝트 디렉토리에 가서 아래를 실행하면 된다:
```bash
$ go test -bench=.
```

환경:

		golang 1.10  
		MacBook Pro darwin/amd64  
		2.6 GHz Intel Core i7  

결과:
```
KAKAOui-MacBook-Pro-5:benc2 aidan$ go test -bench=.
goos: darwin
goarch: amd64
BenchmarkSum-8   	2000000000	         0.69 ns/op
PASS
ok  	_/Users/aidan/go/src/reborn/benc2	1.472s
```

해당 반복문을 몇회 실행해서 어느정도의 평균 성능결과값이 측정되었는지 볼 수 있다.
20억번의 반복문을 실행해서 평균적으로 1번의 작업에 0.69나노초가 걸렸다.
나노초라는 작업단위는 정말 빠르기때문에 훌륭한 함수구만 하고 넘어가면 된다.

#### 응용

함수 하나를 성능 테스트하기보다 비교해서 테스트할때 우린 더 눈이 초롱초롱해질 수 있다.  
서로 다른 Sum함수 2개를 비교해보자.

```golang
func Sum1(a, b int) int {
	return a + b
}

func Sum2(a, b int) (result int) {
	result = a + b
	return result
}
```
- 1번 더하기 함수는 아까와 같다.
- 2번 더하기 함수 Golang의 특성을 이용해 결과값 변수를 먼저 선언하는 방법을 활용했다.

```golang
func BenchmarkSum1 (b *testing.B) {
	for i:= 0; i< b.N; i++ {
		Sum1(1, 2)
	}
}

func BenchmarkSum2 (b *testing.B) {
	for i := 0; i < b.N; i++ {
		Sum2(1, 2)
	}
}
```

기대된다.

결과:
```
KAKAOui-MacBook-Pro-5:benc2 aidan$ go test -bench=.
goos: darwin
goarch: amd64
BenchmarkSum1-8   	2000000000	         0.67 ns/op
BenchmarkSum2-8   	2000000000	         0.34 ns/op
PASS
ok  	_/Users/aidan/go/src/reborn/benc2	2.137s
```

결과값을 먼저 선언한 함수가 0.34나노초 밖에 안걸렸다.  
(나도 이번 예제를 준비하면서 처음 알았다...)  
우리는 코드가 길어져서 팀장님 코드리뷰 때 까일 수 있는 2번째 함수를 이제 이 자료를 보여주며 당당하게 주장할 수 있다. **테스트 해봤는데 이게 더 좋습니다**

##### **추가정보**  
`-benchmem` 옵션을 사용하면
작업당 메모리 사용량과 작업당 할당량을 확인할 수 있da.  
간단하게 붙여보자.
```
KAKAOui-MacBook-Pro-5:benc2 aidan$ go test -bench=. -benchmem
goos: darwin
goarch: amd64
BenchmarkSum1-8   	2000000000	         0.68 ns/op	       0 B/op	       0 allocs/op
BenchmarkSum2-8   	2000000000	         0.35 ns/op	       0 B/op	       0 allocs/op
PASS
ok  	_/Users/aidan/go/src/reborn/benc2	2.183s
```

작업당 바이트 수와 작업당 allocs수가 추가된 것이 보인다.

---
#### StopTimer, StartTimer

테스트를 만들다보면 구체적으로 테스트를 하고싶은 구간을 정하고싶을 때가 있다. `StopTimer`함수를 호출해 기초 세팅을 한 뒤 실제 테스트하고싶은 부분 앞에서 `StartTimer`함수를 호출하면 정확히 원하는 부분을 테스트할 수 있다.


**Example**

Map과 Slice는 Golang에서 쉽게 접할 수 있는 자료구조이다.
Golang의 Map과 Slice에는 capacity(용량)이라는 것이 있는데 앨런 도프먼의 the Go Programming 책을 보다보면 length를 늘릴때 capacity를 초과하면 기존의 것을 복사하고 재해시하는 등의 큰 작업이 있다는 것을 알 수 있다.

예제 아이디어 출처:(https://qiita.com/oywc410/items/ad8baee00f039705a5c0)

map용량을 설정하고 사용하는 것이 좋은지 테스트를 직접 돌려보자.
```golang
func test(m map[int]int) {
	for i:=0; i < 100000; i++ {
		m[i]= i
	}
}

func BenchmarkMap (b *testing.B) {
	for i:= 0; i< b.N; i++ {
		b.StopTimer()
		m := make(map[int]int)
		b.StartTimer()
		test(m)
	}
}

func BenchmarkCapacityMap(b *testing.B) {
	for i := 0; i < b.N; i++ {
		b.StopTimer()
		m := make(map[int]int, 100000)
		b.StartTimer()
		test(m)
	}
}
```
- test함수는 map에 10만개의 데이터 인풋을 행하는 함수이다.

 맵을 make함수로 선언 및 할당하는 행위자체가 테스트에 영향을 줄수 있기때문에 StopTimer로 테스트타이머를 멈춘 뒤 진행한다.
BenchmarkCapacityMap은 용량(capacity)을 미리 설정한 뒤 데이터인풋 테스트를 진행하는 벤치마크함수다.


테스트 결과:
```
KAKAOui-MacBook-Pro-5:benc aidan$ go test -bench=. -benchmem
goos: darwin
goarch: amd64
BenchmarkMap-8           	     200	   8811713 ns/op	 5765791 B/op	    4005 allocs/op
BenchmarkCapacityMap-8   	     300	   5050326 ns/op	  322626 B/op	    1678 allocs/op
PASS
ok  	_/Users/aidan/go/src/reborn/benc	4.885s
```

역시 우리의 예상대로 용량을 크게 먼저 준경우가 훨씬 빠르다.
100만 나노초가 1ms이므로  
505만 나노초는 5.05ms  10만개의 데이터 인풋에 5ms면 빠른 성능이다.

---
#### PostScript

성능을 올리는 것도 중요하지만 더 중요한걸 놓치고 있지 않을까 늘 생각해보자

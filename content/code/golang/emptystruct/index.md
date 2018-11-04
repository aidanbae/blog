---
title: "Golang - EmptyStruct 빈구조체"
date: 2018-10-18T11:31:27+01:00

categories: ['Code', 'Golang']
tags: ['golang', 'empty struct', 'struct{}{}']
author: "Aidan.bae"
draft : false
noSummary: false
---

```
done <- struct{}{}
```

고랭관련 깃을 돌아다니면서 훌륭한 레파지토리(Docker, Gin..)들을 내부를 구경하다보면  
위처럼 빈 구조체를 채널에 던지는 경우를 종종 볼 수 있다.

`struct{}{}`

the empty struct

나는 이게 {}{} 이 리터럴 모양이 뭔가 지저분해보여서  
간단한 종료 시그널을 보내는 채널은 대부분 bool, int 채널을 사용했었다.  

bool 값보내는 거 보다 값이 적은가?..  
나는 구조체라고 생각해서 더 크다고만 생각했는데  
찾아보니 놀랍게도 struct{}{}는 사이즈가 `0`이다(기본기가 부족한 나만 놀란거일수도 있다;)


사이즈를 확인할 수 있는 내장 메소드가 있을까 찾아보니  
unsafe.Sizeof 메소드는 해당 변수의 크기값을 반환한다.  
예를들어 스트링변수라면 golang 1.10버전 기준으로 16을 반환한다.

```
stringVariable := "하하하"
fmt.Println(unsafe.Sizeof(stringVariable)) // 16
```

그렇다면 빈 구조체와 부울변수를 찍어보자:
```
emptyStruct := struct{}{}
fmt.Println(&emptyStruct)   // &{}
fmt.Println(unsafe.Sizeof(emptyStruct)) // 0

boolVariable := true
fmt.Println(&boolVariable)  // 0xc42001a088
fmt.Println(unsafe.Sizeof(boolVariable)) // 1
```

빈 구조체는 주소값조차 없으며 사이즈는 0이다.  
값 할당 자체가 일어나지 않는 변수라는 것을 알 수 있다.  

할당하고 안하고의 퍼포먼스 차이를 한번 알아보자
```
func GetBool() bool {
	return true
}

func GetEmptyStruct() struct{} {
	return struct{}{}
}

func BenchmarkGetBool (b *testing.B) {
	for i:= 0; i< b.N; i++ {
		GetBool()
	}
}

func BenchmarkGetStruct (b *testing.B) {
	for i := 0; i < b.N; i++ {
		GetEmptyStruct()
	}
}
```

GetBool은 true라는 값을 반환하는 함수이고  
GetEmptyStruct는 struct{}{}를 반환하는 함수이다.  

```
KAKAOui-MacBook-Pro-5:benc2 aidan$ go test -bench=. -v
goos: darwin
goarch: amd64
BenchmarkGetBool-8     	2000000000	         0.63 ns/op
BenchmarkGetStruct-8   	2000000000	         0.31 ns/op
```

두배의 차이가 있다.

---
#### 마무리

엄청 작은 차이이지만
단순히 시그널 용이라면 빈 구조체를 활용하는게 좋다.

---

##### 참고
stackoverflow에 나 같은 물음을 한 사람은 역시 존재했다.  
EmptyStruct에 대해서는 꽤나 깊은 이야기들이 존재하고있었다.  
https://stackoverflow.com/questions/20793568/golang-anonymous-struct-and-empty-struct
https://dave.cheney.net/2014/03/25/the-empty-struct

코딩에서 어떤 프로그래머를 리스펙하는건 꽤나 중요한 일이다.  
Dave Cheney는 내가 덕질하는 Gopher중 한명인데,  
그의 블로그를 살펴보면 고랭의 로우레벨까지 깊은 이해를 엿볼 수 있다.  
(context의 오용과 관련한 포스트 추천!)  
Golang 디자인을 총괄하고있는 Russ cox도 해당 글에 직접 댓글을 달아주었다.

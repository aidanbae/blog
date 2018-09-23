---
title: "Golang - gin middleware와 handler활용"
date: 2018-09-11T09:31:27+01:00

categories: ['Code', 'Golang', 'Server']
tags: ['golang', 'middleware', 'gin.handler', 'gin']
author: "Aidan.bae"
draft : false
noSummary: false
---

gin프레임워크를 사용하다보면 기존의 http.Handler와 함수 형태가 다르기때문에 wrapping을 해줘야하는 경우가 많다.

우선 go에서 제공하는 `net/http`패키지의 http.handler를 살펴보자.

```golang
func main() {
	http.HandleFunc("/hello", helloWorldHandler)
	http.ListenAndServe(":8000", nil)
}

func helloWorldHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprint(w, "hello world \n")
}
```

- helloWorldHandler는 첫번째 인자로 http.ResponseWriter를 받고 두번째 인자로 Request의 포인터 변수를 받는다.

단순한 마이크로 서비스라고 해도 요청된 경로나 요청박식에 따라 다른 핸들러로 요청을 라우팅하는 기능이 필요하고
위 코드는 간단하게 `/hello`라는 path로 들어온 요청에 대해 `helloWorldHandler`로 라우팅하는 것을 보여준다.
Go에서 이것은 ServerMux의 인스턴스인 DefaultServeMux 메서드에 의해 처리된다.
자세한 내용은 net/http패키지의 ServerMux Doc를 참고하자.  
또다른 방식으로 http.Handle()메서드를 사용할 수도 있다.
두 방식 모두 ResponseWriter와 *Request 값을 인자값으로 받는 인터페이스를 만족한다.

gin에서는 라우팅하는 함수의 인자가 독특하다.

```golang
r := gin.New()
r.GET("/hello", func(c *gin.Context){
		c.String(200, "hello world")
	})
r.Run(":8080")
```

r.GET, r.POST 등의 메서드로 쉽게 요청 방식과 요청경로를 지정할 수 있다.  
위의 예제코드처럼 첫번째 인자는 경로패턴이 들어가게되고 두번째 인자로 핸들러 함수가 들어간다.

gin.handler를 들여다보면
```
type HandlerFunc func(*Context)
```
gin.Context 포인터변수를 인자값으로 받는 형태이다.
이 형태에 맞추어 우리는 미들웨어를 만든다던지 추가적인 handler를 생성 및 삽입할 수 있다.

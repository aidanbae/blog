---
title: "Golang 결제검증서버 구현 - Validator 디자인"
date: 2018-09-07T09:31:27+01:00

categories: ['Code', 'Golang', 'Server']
tags: ['golang', 'interface', 'validate']
author: "Aidan.bae"
draft : false
noSummary: false
---
#### Intro
**결제** 와 관련된 정보는 고객 입장에서도 회사 입장에서도 **매우 중요한 정보**이다.  
타사(구글, 페이스북, 샘성)의 결제모듈을 활용한다면 그 절차 사이사이의 검증과 기록은 결제검증서버가 해야할 필수적인 부분-
이번에 페이스북으로 글로벌 게임서버 런칭을 진행하면서 페이스북 결제를 활용해 돈방석에 앉아보기로 팀방향이 결정되었고(제발~)  
우리 서버팀은 그 돈방석의 영수증 역할을 할 결제검증서버를 구현하게되었다.

이전에 불확실성을 줄이기위해 방명록을 간단하게 만들어보면서 orm프레임워크, mariadb 등을 확인했었고
그 연장선에서 이번 검증서버 프로젝트는 `xorm`과 web framework는 `gin`을 활용하기로 했다.


---
#### Point

결제 쪽에 자신감이 넘치는 팀장님이 명확한 로직을 제시해주었기때문에 코딩은 어렵지 않았다.  
서버가 커질수 있는 **확장성과 인수인계, 가독성등을 고려했을 때 어떤 식으로 설계하는 것이 올바를지에 대해 고민** 을 하는 시간에 좀더 비중을 두었다.

##### 고려해야할 상황

- 요청별 구조체가 다양하다.
- 여러 구조체에서 검증요소가 중복된다.
- 로그를 남기기 편하도록 검증에러값을 가장 중요하게 실어나른다.
- DB 접근을 최소화해야한다.

---

##### 접근법

- 책임연쇄 패턴처럼 검증되는 요소들을 명시적으로 모아두고 쭉 돌려서 검증통과여부를 확인한다.
- 각각의 필드는 자신을 검증하고 pass여부와 에러를 반환한다.
- context를 활용해 DB에 한번 접근했을때 중요정보를 context에 저장해둔다.

이 접근법을 활용하면 코드의 재사용성을 확보하면서 중복되는 부분을 철저하게 제거할 수 있다.  
확장성과 가독성은 덤이다.


##### Validator

```golang
type Validator interface {
	Check(c *gin.Context) (bool, error)
}

func Validate(c *gin.Context, val ...Validator) (bool, error) {
	//ctx, cancel := context.WithCancel(c)
	var isPass bool
	var err error
	for _, vali := range val {
		if isPass, err = vali.Check(c); !isPass {
			return isPass, err
		}
	}
	if err != nil {
		return true, err
	}
	return true, nil
}
```
- `Validator`는 검증과 관련된 interface이다.
	- 정의해둔 Check메소드만 가지고 있다면 Validator에 만족한다.
- 여기서 나오는 첫번째 인자 `gin.Context`는 `context.Context`와 비슷한 역할을 한다.
 	- 나는 gin을 사용하기때문에 gin.Context를 활용하는 쪽으로 가닥을 잡았다. context.Context인터페이스를 만족하기 때문에 주석처럼 커스터마이징해서 사용할 수도 있다.  
- Validate 함수는 검증 통과여부와 에러값을 반환한다.
	- 컨텍스트를 첫번째 인자로하고 그다음부터는 검증할 녀석들을 spread operater 인자로 쭉 받는다.

##### Validator 예시: player_id
```golang
type PlayerId string
func (p PlayerId) Check(c *gin.Context) (bool, error) {
	if v, exists := c.Get("player_id"); exists {
		if userId, ok := v.(string); ok {
			if userId == string(p) {
				fmt.Println("아이디체크통과: ", userId, p)
				return true, nil
			}
			return false, errors.New("userId가 다릅니다. 해킹이 의심됩니다")
		}
		return false, errors.New("저장된 player_id가 string이 아닙니다.")
	}
	return false, errors.New("저장된 player_id가 없습니다")
}
```

player_id 필드값은 string이다. 해당 값을 PlayerId라는 타입으로 한번 감싸기만하면 Check함수를 사용할 수 있다.  
검증 이전에 context에 저장한 player_id값과 비교하는 내용이다.
만약 gin을 사용하지 않고 context패키지를 사용한다면 c.Get이 아닌 c.Value("player_id") 메소드를 통해 값을 가져올 수 있겠다.

이제 우리는 검증할 요소가 생긴다면 위처럼 Check 메소드가 구현된 필드 타입을 만들면된다.


##### 검증하는 부분

```golang
pass, err := Validate(ctx,
      PlayerId(decodedData.PlayerId), // 플레이어 아이디 검증
      ExpiredTime(requestStruct.IssuedAt), // 시간 만료 검증
	  //...
      )
if err != nil {
	c.Error(err)
	c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
	return
}
```

마치 컴포넌트처럼 재사용할 수 있는 검증 로직이 완성되었다.  
검증해야될 필드가 추가된다면 그에 따른 Validator를 추가하고
Validate에 인자값으로 추가해나가기만하면 된다.  

> **context를 왜 첫번째 인자로??**  
Golang에서 가장 큰 github 프로젝트라고 한다면 **Docker**를 꼽을 수 있다.
도커 레파지토리를 잘 관찰해보면 그들의 인터페이스 활용은 google이 제시하는 가이드라인에 철저히 맞추어 표현하고있다. 그들의 인터페이스의 공통점은 context를 모든 인터페이스들의 첫번째 인자로 받는다는 것이다.
이는 각각의 자식 context들이 만들어내는 복잡한 고루틴 로직들을 확실하게 제어하도록 도와주면서 사이드 이펙트를 최소화한다. 설계 가이드라인은 go blog: context를 자세히 보면 알 수 있다.  

>At Google, we require that Go programmers pass a Context parameter as the first argument to every function on the call path between incoming and outgoing requests. This allows Go code developed by many different teams to interoperate well. It provides simple control over timeouts and cancelation and ensures that critical values like security credentials transit Go programs properly.

>Server frameworks that want to build on Context should provide implementations of Context to bridge between their packages and those that expect a Context parameter. Their client libraries would then accept a Context from the calling code. By establishing a common interface for request-scoped data and cancelation, Context makes it easier for package developers to share code for creating scalable services.

>By Sameer Ajmani(Google)

---
title: "빅오 표기법(Big O notation)과 자바스크립트"
date: 2018-09-20T09:31:27+01:00

categories: ['Javascript', 'Algorithm']
tags: ['javascript', 'algorithm', 'big o', '빅오표기법', '자바스크립트']
author: "Aidan.bae"
noSummary: false
draft: true
---

안녕하세요. 아이단입니다.  
오늘은 컴퓨터과학의 꽃, 알고리즘의 인트로에 늘 나오는 **빅오표기법**에 대해서 이야기해보려고합니다.  

대학생 때, 삼성SCSS 덕분에 컴퓨터과학을 늦게 접하면서 시립대에서 김진석 교수님 알고리즘 수업을 야간에 들었었는데 `컴퓨터과학 전공자와 비전공자를 나누는 기준이 알고리즘`이며 `알고리즘의 효율성을 판별하는 빅오표기법은 매우 중요해요`라고 콕 찦어 말하신게 기억에 남네요.

스타트업에 있을 때는 되게하는데 바빠 빅오표기법을 고민한 적이 없었고,
현업에 들어와서 게임프로그래밍을 하며 알고리즘의 효율성, 뎁스를 줄이기 위한 고민을 했을 뿐 정확하게 빅오표기법으로 얼마다하며 팀장님을 설득한 적은 없는 거 같습니다.

그냥 프로젝트들을 이어서 하다보니
느낌이 쌓여서 이건 slice로 해야겠다. map으로 하는게 낫겠다는 생각을 하게되었고 게임서버를 하면서 예측되지않는 유저들의 커넥션을 관리할때 map으로 해야 탐색이 빠르겠는데 정도의 느낌만 가져갔었습니다.

서비스에 최적화된 능력은 키웠지만 알고리즘은 늘 저에게 아직 약한 부분이었고, 내년 구글코드잼(Google CodeJam)을 준비하면서
프로그래머스라는 사이트에서 몇몇개를 공부삼아 풀면서 결과검사는 통과하는데
효율성 검사에서 시간초과가 나는 경우가 있어 빅오표기법과 자바스크립트를 정리하게 되었습니다.


---

#### 빅오(Big O)표기법

빅오표기법은 알고리즘이 걸리는데 필요한 시간의 수학적인 표현이라고 할 수 있습니다.  
일반적으로 **최악의 시나리오를 상정했을 때의 복잡도**를 의미합니다.


#### 왜 자바스크립트?

물론 알고리즘 쪽에서는 c++이나 java가 가장 대중적이다.  
아는 동생이 카카오 블라인드 테스트에서 나에게 질문한게 자바스크립트,  
자바스크립트는 나의 주력 언어 중 하나,  
동적언어의 매력, 너무 편해서  
builtin method(내장 함수)들을 많이 알고 있다.

#### 알고리즘

서비스 쪽에 익숙해지다보면 편한 함수들이 많아  
효율성보다 가독성으로 아름답게 문제를 해결하고자 하는 경우가 생긴다.

조금 극단적인 경우를 보겠다.  
아래는 동아리 동생이 이번 카카오 블라인드 문제를 풀면서  
시간초과가 난다며 나에게 문의한 코드이다.

```
record.map((txt) => {
        const states = {
            action: txt.split(' ')[0],
            userId: txt.split(' ')[1],
            userNick: txt.split(' ')[2],
        };

        if(states.action === 'Enter') {
            addUser(states.userId, states.userNick);    
        }
        if(states.action === 'Change') {
            changeUserNick(states.userId, states.userNick);
        };

        newRecord.push(states);
    });
```
일단 어떤문제인지 모르는 상태에서 보아도 불편한 부분이 조금 보인다.  
##### 1. map이라는 도구를 잘못 활용했다.

record라는 array를 builtin method `map`으로 호출한다.  
map은 es6에 추가된 array메소드로  
**내부를 순환**하면서  
**콜백함수에서 리턴된 값**으로  
**새로운 배열을 반환**한다.  

위 코드는 return값이 없는 콜백함수를 품고 있고 forEach문을 통해 충분히 할 수 있다.
새로 만들어진 array를 활용하고 있지도 않다.
만약 map을 활용한다면 아래처럼 새로운 배열을 사용하는 로직이어야한다:
```javascript
var newRecord = record.map((txt) => {
  //...
  return states
  })
```

##### 2. string.split()

string의 split은 builtin 메소드로 내부적으로 살펴보면 string을 순회하면서 공백으로 분리하는 작업을 하고 있을 것이다.
전체적으로 보면 map이라는 반복문 안에 split이라는 반복문이다.
심지어 변수 하나당 한번씩 3번 호출하고있다.

수정:
```
const splited = txt.split(' ')
const states = {
           action: splited[0],
           userId: splited[1],
           userNick: splited[2],
       };
// ...
return states
```
##### 3. addUser, changeUserNick 함수가 뭔일을 할까?

벌써 두렵다.

---
title: "Javascript Console 활용"
date: 2018-01-24T09:31:27+01:00

categories: ['Code', 'Javascript']
tags: ['javascript', 'console.log', 'console.time', 'console.timeEnd', 'console']
author: "Aidan.bae"
noSummary: false
---

#### Intro
자바스크립트를 쓰면서 웹어플리케이션을 만들다보면 Debug를 위해 console객체를 자주 사용하게됩니다. 간혹 잘못 사용해서 잘못된 정보를 본다면... 삽질의 시작이죠
#### Why
Console 객체의 주의해야할 점과 좋은 활용법을 터득해 즐거운 디버깅을 해봅시다.

---
### console.log
자바스크립트 개발자라면 너무 친숙한녀석이죠.  

    console.log(obj1 [, obj2, ..., objN]);  
    console.log(msg [, subst1, ..., substN]);

보시는 것처럼 객체와 메세지 등 다양한 인자값을 출력하는데 사용합니다.

주의할점은 객체를 로깅할때입니다.
우선 string 변수를 로깅해봅시다.

```javascript
var sangik = "상익아 뭐하니"
console.log(sangik); // 상익아 뭐하니
sangik = "what are you doing? aidan"
console.log(sangik); // what are you doing? aidan
```

바로바로 변한 값이 찍히네요.
이번엔 객체를 찍어볼까요. 객체 리터럴로 선언하겠습니다.
```javascript
// 아침의 상익
var sangik = {
  'name': 'aidan',
  'age': 28,
  'sex': 'male',
  'state': '심란하다아주'
}
console.log(sangik)
// 시간이 흘러.. 밤의 상익
sangik['state'] = '졸리당'
console.log(sangik)

```
**결과**

![result1](/code/javascript/console/screenshot.png)

잘나왔네요. 문제 없어보이네요 이제 펼쳐볼까요 (크롬 콘솔창입니다.)

![result2](/code/javascript/console/screenshot2.png)

`심란하다아주`를 펼쳤더니 `졸리당` state가 나오네요  
객체를 펼쳐볼때 해당 녀석의 **메모리를 찾아가기때문에**  
state가 최근값인 '졸리당'이 되어버린 상황  

실제 서비스에서는 객체의 뎁스가 깊기에 후자의 방식으로 펼쳐보는 경우가 대부분입니다.  
어쩌죠. 객체를 펼친 당신은 잘못된 데이터를 가지고 원인추론을 시작합니다.  
**삽질의 시작입니다.**

---

개발자가 원하는 로그는 해당 로그가 위치한 곳이 실행될때의 객체 상태일텐데...
##### 그렇다면 어떻게 제가 원했던 객체녀석의 상태를 읽을까요?

쉬운 방법 2가지가 있습니다.

- JSON.stringify(obj) 활용
- console.table(obj) 활용

<br>
##### JSON.stringify

JSON.stringify를 활용하면 그때당시의 객체상태를 JSON문자열로 바꿔서 볼 수 있습니다.

```javascript
...
console.log(JSON.stringify(sangik))
// 시간이 흘러
sangik['state'] = '졸리당'
console.log(JSON.stringify(sangik))
```

**결과**
![result3](/code/javascript/console/screenshot3.png)

##### console.table
Table을 사용하면 펼쳐볼 필요도없이 눈으로 쉽게 보라고 테이블을 그려줍니다.  

```javascript
...
console.table(sangik)
// 시간이 흘러
sangik['state'] = '졸리당'
console.table(sangik)
```

**결과**
![result4](/code/javascript/console/screenshot4.png)

---

### console.time과 console.timeEnd  
- '이 반복문 다도는데 얼마나 걸릴까?'  
- '서버랑 통신하는데 어느정도 걸리는거지?'  

런타임에서 시간을 체크해야하는 경우가 자주 생깁니다.  
그때 요긴하게 사용할 수 있는 녀석이 요 두녀석입니다.

인자값으로 `라벨링`을 할수 있어서 복잡한 코드 속에서도 체크할 수 있습니다.

    console.time(label)
    console.timeEnd(label)

`console.time`은 타이머를 호출하는 함수입니다.  
한페이지에 최대 10000개의 타이머를 호출할 수 있습니다.  
`console.timeEnd`는 해당 타이머를 중지하고 elapsed time을 출력하는 함수입니다.


100만번 도는 반복문을 만들어서 나의 크롬창과 반복문의 퍼포먼스를 시험해봅시다:
```javascript
console.time('aidan loop 1')
var sum = 0;
for (var i = 0; i< 1000000; i++) {
  sum += i;
}
console.log(sum)
// 499999500000
console.timeEnd('aidan loop 1')
// aidan loop 1: 47.418701171875ms
```

백엔드 서버와 통신 딜레이를 측정하는 경우 Promise의 then, catch에 묶어서 사용해도 되겠네요.

##### 추가정보
window의 내장객체인
`performance 객체`를 사용하는 것도 좋은 방법입니다.

가벼운 예제:

```javascript
var t0 = performance.now();
var sum = 0;
for (var i = 0; i< 1000000; i++) {
  sum += i;
}
console.log(sum)
// 499999500000
var t1 = performance.now();
console.log("Call to doSomething took " + (t1 - t0) + " ms");
// Call to doSomething took 47.4000000001397 ms
```
(performance객체와 크롬 DevTool을 활용하면 좀 더 정밀한 측정, 프로파일링이 가능합니다)  
HTML5게임처럼 고프레임을 유지해야하며 프레임별 데이터가 필요한경우
`performance.mark()`와 `performance.measure()`를 활용해서
타임라인을 볼수 있습니다.  

구글이 워낙 설명을 잘해놔서 자세한 내용은 아래 사이트에서 확인하실 수 있습니다.  
https://developers.google.com/web/tools/lighthouse/audits/console-time?hl=ko  
https://www.html5rocks.com/en/tutorials/webperformance/usertiming/

---
#### 마무리
디버깅 고고

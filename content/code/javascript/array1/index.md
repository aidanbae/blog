---
title: "Javascript Array - forEach, map, filter"
date: 2017-07-11T09:31:27+01:00

categories: ['Code', 'Javascript']
tags: ['javascript', 'array', 'forEach', 'map', 'filter']
author: "Aidan.bae"
noSummary: false
---

HTML5 게임들을 만들면서
가장 유용하고 자주 사용한 자료구조  `배열`입니다.  
`push` `pop` `shift` `unshift`등의 기본 함수들을 사용해
손쉽게 데이터를 조작할 수 있습니다.    
더 나아가 멋지게 배열들을 다룰 수 있는 중급 내장함수들을 만나보겠습니다.

> 기본으로 제공되는 함수를 배움으로서 생산성이 향상됩니다.  
자바스크립트는 Array객체를 제공합니다.  
배열을 자유자재로 다룰 수 있다면 효율적이고 손쉬운 코딩이 가능합니다.  
전역객체인 Array를 쉽게 다루기 위한 **표준내장함수** 들을 공부해봅시다.


### forEach
해당 배열의 요소 하나하나를 순회하면서 반복문을 돕니다.
```
array.forEach(callback [, thisArg])
```
첫번째 인자 `callback(item, index, array)`  
두번째 인자 `thisArg` // 콜백함수 사용시 this로서 사용되는 값

```javascript
var fruits = ['사과', '바나나', '배']
fruits.forEach(function (item, index, array) {
  console.log(item, index);
});
// 사과 0
// 바나나 1
// 배 2
/* ES6 */
fruits.forEach((item, index, array) => { console.log(item, array[index]) })
// 사과 사과
// 바나나 바나나
// 배 배
```
---
### map
배열 내의 모든 요소 각각에 대해 제공된 콜백함수를 호출하고. 그 결과를 모아 _새로운 배열을 반환_ 합니다
```
array.map(callback [, thisArg])
```
첫번째 인자 `callback(item, index, array)`  
두번째 인자 `thisArg` // 콜백함수 사용시 this로서 사용되는 값  
해당 배열에 따라 새로운 맵을 형성하므로 활용 범위가 넓습니다.

```javascript
var userArray  = [{id: 1, name: '상익'}, {id: 2, name: '동혁'}, {id: 3, name: '지현'}]

var newUserArray = userArray.map(function(obj) {
  var user = [];
  user.push(obj.id)
  user.push(obj.name)
  return user
});
console.log(userArray)
console.log(newUserArray)
// userArray : [{id: 1, name: '상익'}, {id: 2, name: '동혁'}, {id: 3, name: '지현'}]
// newUserArray : [[1, "상익"], [2, "동혁"], [3, "지현"]]
```
_서버로 부터 userArray를 받아 내가 원하는 2중 배열로 데이터 형태를 바꾼모습_

새로운 배열을 생성해서 리턴하기때문에 메모리관리가 중요한 게임같은 장르에서는 사용이 성능을 잡아먹을 수 있습니다.
하지만 고성능이 필요하지 않은 웹서비스의 경우, 매우 유용한 데이터 변환 함수입니다.
다음은 Array의 map함수와 Function객체의 call을 응용한 예제입니다.

```javascript
var map = Array.prototype.map
var a = map.call('Hello Aidan', function(x) {
  return x.charCodeAt(0)
})
// string객체 역시 배열객체이기 때문에 인자값
// 단어 하나하나에 대한 Byte의 아스키코드값을 요소로가지는 배열을 얻는 방법
// [72, 101, 108, 108, 111, 32, 65, 105, 100, 97, 110] // 공백의 바이트코드는 32
```
**참고자료**  
[MDN 자바스크립트 String.prototype.charCodeAt](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt)  
[MDN 자바스크립트 Function.prototype.call](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

---
### filter

VueJS로 댓글플랫폼을 만들면서 가장 많이 사용한 함수입니다.  
`MVVM`으로 상태관리를 하다보니 관리받고 있는 DataList를 조작해
제가 원하는 리스트로 필터링할 함수가 필요했는데요.
Filter는 그에 딱맞는 기능을 제공해줍니다.

```
var newArray = array.filter(callback[, thisArg])
```

인자로 들어가는 콜백함수에는 `테스트함수`가 들어갑니다.
테스트함수를 통과한 요소가 있는 새로운 배열을 반환해줍니다
아무튼 테스트함수에 `리턴값이 true인 요소들에 대해서만 필터` 됩니다.  
간단한 예제:

```javascript
function isSmallEnough(value) {
  return value <= 100;
}
var filtered = [120, 500, 80, 130, 44].filter(isSmallEnough);
console.log(filtered) // [80, 44]
```
`isSmallEnough라는 함수는 100이하의 값에 대하여 true를 반환합니다.`
filter함수는 isSmallEnough라는 테스트함수를 사용하여 필터링을 진행,
테스트를 통과한 요소들로 구성된 새로운 배열을 반환합니다.
결과값으로 100이하의 값으로 구성된 [80, 44]가 filtered에 담긴것을 확인가능!

```javascript
ES6를 활용해 한줄로 간단히 내가쓴 댓글목록만 걸러냅니다.
// username 클라이언트에 저장되어있는 사용자이름을 댓글작성자 이름과 비교해 필터링
this.commentList = this.commentList.filter((comment) => comment.author === username)
```

---
## 마무리
자, 여러분들은 모르시겠지만 이미 강해졌습니다.  
각자의 프로젝트에 잘 적용해보시길!  

---
title: "Golang과 websocket을 활용한 서버 프로그래밍 (장애 없는 서버 런칭 도전기)"
date: 2018-07-30T09:31:27+01:00

categories: ['Code', 'Golang']
tags: ['golang', 'websocket', 'server']
author: "Aidan.bae"
noSummary: false
---

2018, 7월 25일 제1회 golang meetup이 열렸다.  

그간 golang korea 커뮤니티에서 눈팅으로 이득을 취한 나...
장애없이 런칭한 Golang 스낵게임서버를 개발할 때에도 수많은 golang 블로그에서 도움을 받았기때문에
이번 meetup에서 그분들을 만나뵙고 이야기를 나눌 수 있으면 좋겠다 생각해서 참가를 신청했다.
발표자가 없어 존폐위기까지 거론하셔서 발표를 하는 방향으로 결정

작은 토즈방에서 20명정도 모여서 소근소근 토론하는 자리가 될줄 알았는데 160명이나!!  
개발인생 첫 발표인데 많은 사람이 와주셔서 감사한 마음으로 준비했다.

---
<iframe src="//www.slideshare.net/slideshow/embed_code/key/IAOoU055tM9yPS" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/SangikBae/golang-websocket-109095156" title="golang과 websocket을 활용한 서버프로그래밍 - 장애없는 서버 런칭 도전기" target="_blank">golang과 websocket을 활용한 서버프로그래밍 - 장애없는 서버 런칭 도전기</a> </strong> from <strong><a href="https://www.slideshare.net/SangikBae" target="_blank">Aidan</a></strong> </div>


발표시간 40분, 질의응답시간 10분 (총 50분)  
첫 meetup에 어울리는 주제가 무엇일까 생각했는데
`Concurrency`가 Golang의 디자인철학과 가장 밀접하다고 판단, Build나 배포 pipeline구축 경험을 공유하는 것보다
개발과정에서 Concurrency와 관련해 방황했던 경험을 공유하는 쪽으로 가닥을 잡았다.
그리고 마무리는 `websocket을 활용한 demo`를 보여주는 시나리오를 구성했다.

#### 요약
 - Mutex 너무 많이 사용하면 데드락을 만날거야.
 - share memory by Communicating (Channel을 잘 활용해보자)
 - 생산성, 퍼포먼스, 안정성 다 만족스럽다.

받은 질문
```
Q: mutex가 안정적이지 않다는 말씀이신데 그럼 golang의 sync패키지에서 왜 mutex 지원하는거에요?
A: mutex는 전통적인 방법, 앞서 말씀드린거처럼 하드웨어쪽과 가깝기때문에 퍼포먼스측면에서 좋다.
그래서 제거할 이유도 없다고 생각한다. 나보다 Go blog FAQ를 보시면 그 대답에 도움이 될 것 같다.

Q: Log System은 어떻게 관리하셨나요?
A: 예전에는 log rotation을 사용했는데, 마라톤으로 옮겨온뒤 json으로 쏴주기만한다.  
차기프로젝트에서는 logrus라이브러리를 사용할 것 같다.

Q: websocket 1006 error뭐에요?
A: 잘 모르겠다.

Q: 생산성이 좋다는건 다른언어에 비해 좋다는 건가요?
A: C++같은 고성능 언어로 서버를 만들어본적이 없어서 잘모르지만 제 기준 짧은 기간동안 한번의 장애도 없이
서비스했기때문에 높다고 생각한다.

Q: 로드밸런싱은 어떻게 한건가요?
A: 그냥 서버에 인원제한 걸어두고 만약 인원이 다찼다면 다른서버 가시라고 말해줬다. 
```

<br>
#### 후기

golang korea에서 활동하시는 네임드분들을 만나뵙고 인사드릴려고 했는데 따로 네트워킹 타임이 없어서 아쉬웠다. 하지만
golang 디자인패턴을 같이 스터디했던 카카오 ai팀 사람들, 
SOPT동아리를 같이 활동했던 지수랑 혜진, 
토스로 이직한 Mason, Rebecca 반가운 얼굴들을 다시 볼 수 있어 좋았다.
좋은 경험을 좋은 곳에 공유해서 만족한다.
발표준비를 하면서 발표자들의 땀을 알게되었다. 생각보다 많은 노력이 들어간 핵심자료들이 많은 것 같았다.
그래서 요즘 빠르게 훑어보고 넘겼던 NDC발표들을 다시 천천히 보고있다.

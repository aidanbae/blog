---
title: "Docker client와 Docker server"
date: 2018-08-09T09:31:27+01:00

categories: ['Server', 'Docker']
tags: ['docker', 'infra', 'gocd', 'docker client', 'docker daemon']
author: "Aidan.bae"
noSummary: false
---

#### **Why**
GoCD에서  AWS ECR로 이미지 빌드, 푸시 pipeline을 구축하면서  
Docker의 구조에 대해서 좀더 자세히 알게된 점을 정리한다.  
아키텍쳐를 이해함으로서 도커 통신과 관련된 이슈를 좀더 정확하게 해결할 수 있다.

---

Docker 공식문서는 자주보았는데 `Docker Overview`는 그냥 슭 훑고 지나갔었다.  한국어로 치면 **개요(?)** 정도 이겠다. 그 곳에 있는 핵심내용을 정리해본다.

#### Basic

로컬에서 Docker를 사용할때 우리는 아주 손쉽게 docker ps, docker images 등의 명령어를 쳤기때문에
Docker 엔진의 작동방식을 잘 모르고 지나가는 경우가 많다. 간단하게 살펴보자.  

##### Docker Client-Server Architecture

![hw12](/code/docker/docker-overview/screenshot.png)

즉, Docker Client와 Docker Server로 나뉘어있고 그 사이에 REST API로 소통을 한다.
결국 client는 요청을 할뿐 build, run, push등은  실질적인 작업은 다 데몬(server)이 수행한다.
한창, gocd-agent를 구성하면서 빌드 파이프라인을 만들때 docker-client쪽 환경변수에 목메어있었는데
daemon이 docker 작업의 핵심이라는 당연한 사실을 뒤늦게 깨달은 멍청이가 바로 나다.

---


##### Docker 클라이언트
일반적으로 우리가 치는 docker 명령어의 주인이다.  
우리가 흔히말하는 명령행 인터페이스로 생각하면 편하다. CLI이다.  
Docker 서버과 대화를 하려고 늘 노력한다.  

##### Docker 데몬 (dockerd)
지속적으로 running되면서 docker cli의 요청을 기다리고 docker 프로세스들을 관리한다.  
데몬은 Docker를 관리하기 위해 다른 데몬과 통신할 수도 있다.


##### Docker 통신방법
Docker는 데몬과 클라이언트 간의 통신을 할때 로컬에서는 `유닉스 소켓`을 사용하고,
원격에서는 `TCP소켓`을 사용한다. HTTP REST 형식으로 API가 구현되어 있다.

![hw123](/code/docker/docker-overview/screenshot2.png)

그림 속 **선**들에 주목해보자.  
결국 원격 레즈스트리에 통신하는 것도, 이미지들을 관리하는 것도,
접근하는 것도 핵심은 Host머신의 Daemon이다.

---
다음 명령어들을 쳐보자:

    $ docker version

![hw](/code/docker/docker-overview/screenshot3.png)

> 클라이언트와 서버가 어떤 버전인지 **어떤 API version** 으로 통신하는지 이쁘게 나온다. 혹여나 Version이 1.x버전이라면 제발 업그레이드하길 추천드린다. 이전 버전으로하다가 알수 없는 문제들로 수명이 줄어든다. 나는 실제로 줄어든거같다.

    $ docker info

![hw1234](/code/docker/docker-overview/screenshot4.png)

>docker info를 통해 system-wide information을 훔쳐볼 수 있다.
현재 docker daemon이 어떤 방식으로 storage를 관리하는지
debug 모드는 켜져있는지. 보안옵션은 무엇인지 등등 확인가능하다.
지금 내 로컬 데몬은 overlay2방식으로 storage를 관리하며 Debug mode를 켜두었다.

#### Tip

서버개발자라면 docker daemon의 server
**Debug Mode (server)**를 켜놓는 것을 추천한다.
에러가 났을때 그 메세지를 직접 확인해서 빠르게 원인파악을 할 수있다.


#### Summary
축하한다.  
도커의 아키텍쳐를 이해했다면 도커를 다루다 만나는 에러, 이슈를 좀 더 빠르게 해결할 수 있다.  

---
참고자료  
docker doc overviow https://docs.docker.com/engine/docker-overview

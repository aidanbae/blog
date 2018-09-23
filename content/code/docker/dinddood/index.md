---
title: "DinD(docker in docker)와 DooD(docker out of docker)"
date: 2018-09-03T12:31:27+01:00

categories: ['Server', 'Docker', 'DevOps']
tags: ['docker', 'infra', 'gocd', 'docker-client', 'aws-cli']
author: "Aidan.bae"
noSummary: false
draft: false
---

##### Docker의 Client-Server아키텍쳐
단순히 도커를 이용해 이미지를 만들고 띄우는 개발자라면 docker client와 docker daemon이 분리되어 있는 것을 인지하지 못할 수 있다.
반면 CI(Continuous Integration/ 지속적 통합)툴을 사용하며 agent를 세팅하는 DevOps쪽 개발자라면 docker관련 task를 수행하면서 이 사실을 쉽게 접하게된다.

![hw123](/code/docker/docker-overview/screenshot2.png)
그림처럼 docker시스템 유닛은 크게 3개로 분리되며,
Client, Host(Daemon), Registry가 그것이다.
자세한 내용은 이전 포스트 [Docker client와 Docker server](/code/docker/docker-overview)를 참고.


대부분의 현대 CI 도구들(travis, circle, gocd, jenkins)등이 agent를 통해 docker관련 Task를 수행을 하기 때문에 docker daemon은 호스트머신에서 동작하면서 컨테이너로 동작하는 agent들이 docker-client역할을 하는 경우가 많다. 그래서 데브옵스 개발자들은 쉽게 daemon과 client의 분리를 고려하며 docker container에서 agent가 호스트 머신에 위치한 docker daemon에게 어떻게 도커 명령을 전달해야할지 고민하게된다.

---
##### Docker는 docker 위에서 docker를 사용하는 것을 권장하지 않는다.

Docker측에서는 Docker 컨테이너 위에서 도커명령을 실행하는 것을 권장하지 않았다.  
그리고 더 나아가 client를 분리해서 사용하는 것에 대해서 통합하는 방향으로 가닥을 잡았다.
https://docs.docker.com/install/linux/docker-ce/centos/  
Get Docker CE for CentOS 등 docker설치가이드 문서들을 살펴보면  
설치 전에 old version을 uninstall할 것을 오더하고있다.
old version은 docker-client와 docker-engine이 확실하게 분리되어 있었다.

![hw123](/code/docker/dinddood/screenshot.png)
<docker-ce로 통합된 최신 도커를 사용할 것을 권장하고있는 공식 doc>

---
#### Docker in Docker (DinD)
도커 안에 도커는 도커 바이너리를 설정하고 컨테이너 내부의 격리된 Docker 데몬을 실행하는 작업을 의미한다.
즉, 도커데몬이 2개가 뜨는 것이다. CI측면에서 접근한다면 Task를 수행하는 Agent가 Docker Client와 Docker Daemon역할까지 하게되어 도커 명령들을 수행하는데 문제가 없어진다. 이렇게 말로만 들으면 아름답고 문제가 없어보이지만 이 접근에는 큰 단점이 존재한다.

호스트 도커 컨테이너가 privilieged mode로 실행되어야 한다.
```
$ docker run --privileged --name dind1 -d docker:1.8-dind
```
privilieged 플래그를 사용한다면 호스트컨테이너가 호스트머신에서 할 수 있는 거의 모든 작업을 할 수 있게 된다.  
이는 컨테이너를 실행하는데 큰 보안 위험을 초래할 수 있다.

jptazzo는 ci에서 사용되는 dind에 관해 부정적인 생각을 포스트로 잘정리해두었다.
http://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/

DinD 사용법과 원리에 대해 잘 정리된 포스트   
https://sreeninet.wordpress.com/2016/12/23/docker-in-docker-and-play-with-docker/

---
#### Docker Out of Docker (DooD)

도커 외부의 도커는 호스트의 docker socket을 에이전트 컨테이너에 볼륨 세팅을 통해 공유하고  
호스트의 도커 데몬을 이용해 CI의 도커 명령을 실행한다.

에이전트를 띄우는 명령문 예시:
```
$ docker run -v /var/run/docker.sock:/var/run/docker.sock ...
```

-v옵션으로 호스트의 docker socket을 빌려사용할 수 있다.
소켓통신을 통해 에이전트 컨테이너는 호스트의 daemon에 docker 명령을 전달한다.

이 방식은 그나마 Docker측에서 DinD보다 권장하는 방향이다.

---
#### 빌려써도 문제
DooD 방향으로 agent를 구성함에 있어서  
내가 사용하고 있는 CI/CD 도구인 GoCD에서 agent가 호스트의 docker명령을 전달함에 권한 문제에 부딪혔었다.
소켓을 공유했는데 왜 permission관련 에러가 뜨는거지?...  
sudo로 docker 명령이 동작하게 만들어 놓긴했었는데 임시방편이라는 느낌을 안고있다가
최근에 aws ecs 배포 파이프라인을 구축하면서 함께 해결하게되었다.

![g23ddd](/code/docker/dinddood/screenshot4.png)
sudo와 -E 옵션으로 어거지로 동작하게 만든 build job

이는 리눅스 그룹문제였다.  
agent는 동작할때 go라는 그룹세팅을 하게되고 해당 유저권한으로 모든 작업을 실행하게된다.

https://docs.docker.com/install/linux/linux-postinstall/

에이전트가 동작하는 컨테이너 속에서 도커 소켓이 있는 곳을 한번 ls -l로 자세하게 들여다보자
![hw123](/code/docker/dinddood/screenshot2.png)
srw-rw— 1 root 499
499가 핵심이다.
499는 리눅스에서 그룹아이디인데
이 번호는 어디서 왔을까 한다면 호스트머신의 docker 그룹아이디이다.

이제 데몬이 설치되어있는 호스트머신에서 /var/run 디렉토리를 들여다보자
![hw123](/code/docker/dinddood/screenshot3.png)

이건 호스트에서의 docker.sock 그룹명이 docker인 것이 보인다.

왜 컨테이너 안에서는 499일까?  
컨테이너 안에서는 docker 라는 그룹이 없기 때문이다!  
저 그룹에 속하면 docker를 sudo 없이 사용할 수 있다.

오케이 그럼 내 agent docker 파일을 수정해보자  
```
RUN groupadd -g 499 docker //499를 docker로
RUN usermod -aG docker go // docker에 go 유저를 추가해준다!
```
go그룹핑은 gocd에만 해당되는 내용이지만 다른 agent도 같은 원리로 추가해주면 될 것이다.

최신버전의 도커를 사용한다면
```
# RUN groupadd -g 499 docker -> groupmod로 변경(최신도커버전 CE로 전환때문에)
RUN groupmod -g 499 docker
```

여력이 된다면 최근버전의 도커를 사용할 것을 권한다.  
나(회사 인프라)는 docker 1.xx버전을 사용하고 있었기 때문에
나의 agent는 고대 유물인 docker-client를 설치해서 운영하고있었다.  
낮은 버전 탓에 짜잘한 문제들이 있긴 했지만 꾸역꾸역 잘 동작하게 만들어놨었다.  

문제는 외부 레파지토리인 aws ecr에 이미지를 push하는 파이프라인을 구축할 때부터였다.  
회사를 2년정도 다니면서 암걸릴뻔한 적이 두번 정도 있는데 한번이 이때다.  
`~/.docker/config.json`에 aws ecr 로그인 인증받은 데이터가 멀쩡히 존재하고 있는데도  
aws-cli는 push할때면 no basic auth credentials를 뱉었다.  
태어나서 처음으로 stackoverflow에 글도 써봤다.  

알고보니 Docker머신의 낮은 버전때문에  
cert데이터를 aws와 docker가 다른 곳을 바라보고있어  
몇 일을 삽질로 날렸다.(물론 리눅스나 도커를 더 자세히 공부하게 되었지만 힘들었다. 코딩이 그리운 데브옵스)  
도커와 관련되어 이해가 안되는 부분이 생긴다면 [Docker Debug모드](/code/docker/docker-debug) 를 켜놓고 로그를 분석하는 것을 추천한다.

jenkins나 다른 쪽은 워낙 오래되어 사용자 풀이 많아 플러그인으로 쪽 해버리는데,
정보가 적은 gocd는 aws ecs가 유료플러그인이다.
아무튼 구축했으니 연 500만원의 지출을 막았다.

최종스크립트
```
FROM mdock.daumkakao.io/gocd/gocd-agent-centos-7:v17.10.0
MAINTAINER  Aidan <aidan.qs@kakaocorp.com>

# curl 설치
RUN yum install -y curl
# docker-client 제거 최신버전으로 업그레이드 (v1.7)
RUN yum clean all
RUN yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
RUN yum install -y docker-ce
RUN yum install -y sudo
RUN echo -e "\n## GoCD\ngo	ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# go install
RUN curl -LO https://storage.googleapis.com/golang/go1.9.linux-amd64.tar.gz
RUN sudo tar -C /usr/local -xzf go1.9.linux-amd64.tar.gz
ENV PATH $PATH:/usr/local/go/bin


# dockerclient를 사용하는데 sudo를 사용해야하는 상황을 제거하기위해 (v1.4)
# RUN groupadd -g 499 docker -> groupmod로 변경(최신도커버전 CE로 전환때문에)
RUN groupmod -g 499 docker
RUN usermod -aG docker go

# go bashrc
# RUN chown -R go:go /go-support
ENV GOPATH /usr/local/go/

# npm install p.s https://tecadmin.net/install-latest-nodejs-and-npm-on-centos/
RUN yum install -y gcc-c++ make
RUN curl -sL https://rpm.nodesource.com/setup_8.x | sudo -E bash -
RUN yum install -y nodejs npm

# AWS cli 설치
RUN curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
RUN unzip awscli-bundle.zip
RUN ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
RUN aws --version

# glide install
RUN curl https://glide.sh/get | sh
```

이 스크립트를 빌드하고 dood 방식으로 agent를 볼륨세팅해서 띄우면
npm, aws-cli, glide, go, docker를 사용할 수 있는 agent를 만나게된다.

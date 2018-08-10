---
title: "Docker Debug 모드"
date: 2018-08-09T12:31:27+01:00

categories: ['Server', 'Docker']
tags: ['docker', 'infra', 'gocd', 'docker client', 'docker daemon']
author: "Aidan.bae"
noSummary: false
---

#### **Why**

Docker의 디버그모드를 활용해서 트러블슈팅을 빠르게 진행할 수 있다.

---

##### Docker Debug Mode를 켜보자.


**1. 데몬이 사용할 Configuration파일을 편집해야한다.**  

/etc/docker에 위치한 daemon.json파일 없다면 직접 만들어봅시다.

 - `/etc/docker/daemon.json` on Linux
 - `C:\ProgramData\docker\config\daemon.json` on Window


**2. 아래 컨텐츠를 추가합니다**

```json
{
  "debug": true
}
```

ps. 만약 파일이 이미 존재했다면 debug 프로퍼티를 추가하고 true값을 적어줍니다.

**3. HUP signal을 데몬에게 보내서 configuration을 다시 reload하도록 합니다.**

리눅스에서는 다음 명령어를 사용합니다:  

    $ sudo kill -SIGHUP $(pidof dockerd)

윈도우에서는 도커를 재시작하세요.

<br>

---
##### Docker Log를 확인해보자

디버그모드를 켰다면 수집된 디버그 로그들을 확인해야겠죠.  
로그는 운영체제별로 지정된 위치에 쌓이게됩니다.

![hw](/code/docker/docker-debug/screenshot.png)

저는 리눅스 환경이기때문에 해당 명령문으로 쉽게 확인했습니다.

    sudo cat /var/log/messages | grep dockerd

    // order: 메세지로그에서 dockerd가 적힌 녀석들을 보여달라

결과는 아래와같습니다.

```sh
...
Aug  8 13:53:12 dkos-snackgame-dev-slave-5 dockerd: time="2018-08-08T13:53:12.272841054+09:00" level=debug msg="Calling GET /_ping"
Aug  8 13:53:12 dkos-snackgame-dev-slave-5 dockerd: time="2018-08-08T13:53:12.506705735+09:00" level=debug msg="Calling POST /v1.37/images/332448781195.dkr.ecr.eu-west-1.amazonaws.com/uto/push?tag=65"
Aug  8 13:53:12 dkos-snackgame-dev-slave-5 dockerd: time="2018-08-08T13:53:12.954950018+09:00" level=debug msg="hostDir: /etc/docker/certs.d/332448781195.dkr.ecr.eu-west-1.amazonaws.com"
Aug  8 13:53:12 dkos-snackgame-dev-slave-5 dockerd: time="2018-08-08T13:53:12.955971961+09:00" level=debug msg="Trying to push 332448781195.dkr.ecr.eu-west-1.amazonaws.com/uto to https://332448781195.dkr.ecr.eu-west-1.amazonaws.com v2"
...
```

---

##### Expected
저의 경우, 해당 로그들을 통해 ecr push작업간에 이녀석이 어디서 인증데이터를 가져오는지 이슈트래킹이 가능했습니다.  디버그 모드가 여러분의 트러블슈팅에 숨통을 트여주길 기대합니다.

---
참고자료
https://docs.docker.com/config/daemon/

---
title: "Hugo로 github page에 블로그 배포하기"
date: 2018-04-23T09:31:27+01:00

categories: ['Blog', 'Hugo']
tags: ['hugo', 'github', 'deploy', 'tutorial']
author: "Aidan.bae"
noSummary: false
---

#### Why?

Hugo 공식사이트를 보다보면 뭔가 친절한듯 하지만 영어다.  
테마 설정때문에 에러를 겪으시는 분들이 많아  
쉽게 디렉팅해서 여러분들의 시간을 아껴주고싶다.

##### hugo 설치하기

[Install hugo](https://gohugo.io/getting-started/installing/)

##### hugo quickstart

[QuickStart](http://gohugo.io/getting-started/quick-start/)
이거 따라하시면된다.

여기 Step3를 보면 `git clone` 대신 `git submodule`을 사용중이다.
님은 둘중 하나를 선택할 수 있는데 submodule을 쓰는 편이 암에서 해방되는 길이긴하다.
하지만 나처럼 이미 clone을 선택해서 테마를 받았다면 `themes` 디렉토리를 `.gitignore`에 추가해야한다.  

```sh
echo 'themes/' >> .gitignore
```

##### config.toml파일을 수정하자

config.toml 파일에서 baseURL을 수정하자:

```toml
baseURL = "https://<USER-NAME>.github.io/"
```

USER-NAME은 여러분의 ``깃허브 username`이다.

##### 로컬에서 실행해보기
```sh
hugo server
```
`localhost:1313` 에 접속해서 이쁜 당신의 블로그를 확인한다.

##### Build 해보자.
다음 명령어를 쳐보자:
```sh
hugo
```

Hugo가 당신의 웹사이트를 public 디렉토리에 빌드해준다.

##### Upload 해보자
우선 public폴더를 .gitignore에 추가해야한다.
```sh
echo 'public/' >> .gitignore
```
그리고 public 디렉토리로 가서 git세팅을 한다.
```sh
cd public
git init
git remote add origin https://github.com/<USER-NAME>/<USER-NAME>.github.io
```

git push를 하게되면 당신의 정적 사이트는 업로드된다.  
https:<USER-NAME>.github.io로 가서 확인한다.

조금이라도 도움이 되었다면 좋겠당.

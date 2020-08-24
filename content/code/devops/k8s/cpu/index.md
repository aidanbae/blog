---
title: "kubernetes CPU와 docker CPU"
date: 2020-06-05T19:01:14+09:00

author: "Aidan.bae"
categories: ['Server', 'DevOps']
tags: ['cpu', 'k8s', 'debug']
noSummary: false
draft: true
---

리소스에 대한 이해를 토대로 안전한 클러스터 운영

cpu와 메모리는 시스템 리소스의 주요 지표중 하나이다. 클라우드 환경에서는 호스트머신들의 리소스를 나눠쓰기때문에 과도한 리소스 사용은 다른 application에도 영향을 줄 수 있다.

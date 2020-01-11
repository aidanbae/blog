---
title: "envoy와 x-forwarded-proto 헤더 이슈"
date: 2019-12-05T19:01:14+09:00

author: "Aidan.bae"
categories: ['Server', 'DevOps']
tags: ['istio', 'x-forwarded', 'header']
noSummary: false
draft: false
---

게임서버를 할땐 x-forwarded 헤더에 대해 깊게 다룰일이 없었는데
프론트단부터 거치는 프록시 서버들이 많은 시스템을 운영하다보니 접속 클라이언트의 정보가 담긴 x-forwarded 헤더를 공부할 기회가 생겼다.

 `x-`로 시작하는 헤더는 사용자가 임의로 정의한 헤더를 나타냅니다. 근데 nginx등 오픈소스들이 워낙 많이 쓰이다보니 사실상 표준이 된 케이스가 `x-forwarded-*` 시리즈임.


### x-forwarded-for, x-forwarded-proto, x-forwarded-host

위 삼총사는 de-facto standard 헤더로 사실상 표준이 되어버린 헤더임. 이 삼총사는 한세트 묶음으로 보통 전달되며 클라우드 업계에서도 표준으로 자리잡고있음.
https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Forwarded

예를들면 아래처럼 사용될 수 있습니다.

- x-forwarded-for : {client-ip}, {proxy1_ip}, {proxy2_ip}
- x-forwarded-proto : http or https
- x-forwarded-host : toss.im

최초의 접속된 클라이언트에 대한 정보를 나타냄.

제가 운영하는 클러스터는 앞단의 프록시서버에서 https를 처리하기때문에 내부 클러스터안에서는 http로 통신하면서 퍼포먼스상의 실을 최소화하는 구조입니다.

### 이슈

istio-proxy(envoy)에서 파드로 내려주는 헤더 중 x-forwarded-proto가 http로 되어 있어 스프링 부트에서 redirect를 수행하며 절대 경로로 해버려서 https 요청을 http로 변경시켜버림.


#### 해결책


우선 `x-forwarded-proto`가 http로 들어오고 있었다.
istio는 앞단의 프록시 서버가 있다는것을 알지못하기 때문에 이를 명시적으로 클러스터 운영자가 세팅해주어야한다.


*x-forwarded-for만 들어오게해주고 x-forwarded-proto를 없애버리기*

없애버리면 되겠네? 삼총사가 세트라는 사실을 잘 모르고 있었기 때문에 스프링에서 x-forwarded-proto가 없다면 상대경로로 작동한다는 사실때문에 없애보기로했다.
(앞단 프록시 서버에서는 x-forwarded-for만 넘겨주고있었는데, x-forwarded-proto를 istio가 세팅해주는 듯했다.)

istio의 리소스중 하나인 virtual service에서 request헤더와 response해더를 조작할 수 있는데 예를 들면 이런식이다.

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: application-vs
  namespace: default
spec:
  hosts:
  - "app.toss.im"
  gateways:
  - gateway
  http:
  - route:
    - destination:
        host: application.namespace.svc.preprod.kubernetes
        port:
          number: 80
    headers:
      response:
        remove:
          - {header_name}
```
detail은 isto 공식문서를 통해 확인가능하다.
https://istio.io/docs/reference/config/networking/virtual-service/


하지만 x-forwarded-proto를 지우는 VirtualService 세팅을 했음에도
x-forwarded-proto는 http로 들어왔다:

```
curl -X GET devops-test-server.alpha.test.bz -H x-forwarded-proto:https

...map[Accept:[*/*] Content-Length:[0] User-Agent:[curl/7.29.0] X-B3-Parentspanid:[dd1c14896c025516] X-B3-Sampled:[0] X-B3-Spanid:[5f31320997d82578] X-B3-Traceid:[c7ff5be60bae87c8dd1c14896c025516] X-Envoy-External-Address:[~] X-Forwarded-For:[~] X-Forwarded-Proto:[http] X-Request-Id:[f9d6ec34-6aff-41ec-9b93-aa74e2ace010]]
```
보안상 x-envoy-external-address와 x-forwarded-for의 ip주소를 지웠습니다 ㅎㅎ;


안지워지기 때문에 envoy측에서 특수한 방어로직이 있거나 generate한다는 것을 짐작가능

여기는 envoy의 http커넥션과 관련된 레파지토리 소스코드이다.
https://github.com/envoyproxy/envoy/blob/master/source/common/http/conn_manager_utility.cc
관련 로직을 찾아보면 93번째줄!부터 쭉! (2019/12/3 기준, 소스코드는 변경될수 있으므로 직접들어가셔서 확인해야해요)

```
if (config.useRemoteAddress()) {
    single_xff_address = request_headers.ForwardedFor() == nullptr;
    // If there are any trusted proxies in front of this Envoy instance (as indicated by
    // the xff_num_trusted_hops configuration option), get the trusted client address
    // from the XFF before we append to XFF.
    if (xff_num_trusted_hops > 0) {
      final_remote_address =
          Utility::getLastAddressFromXFF(request_headers, xff_num_trusted_hops - 1).address_;
    }
    // If there aren't any trusted proxies in front of this Envoy instance, or there
    // are but they didn't populate XFF properly, the trusted client address is the
    // source address of the immediate downstream's connection to us.
    if (final_remote_address == nullptr) {
      final_remote_address = connection.remoteAddress();
    }
    if (!config.skipXffAppend()) {
      if (Network::Utility::isLoopbackAddress(*connection.remoteAddress())) {
        Utility::appendXff(request_headers, config.localAddress());
      } else {
        Utility::appendXff(request_headers, *connection.remoteAddress());
      }
    }
    // If the prior hop is not a trusted proxy, overwrite any x-forwarded-proto value it set as
    // untrusted. Alternately if no x-forwarded-proto header exists, add one.
    if (xff_num_trusted_hops == 0 || request_headers.ForwardedProto() == nullptr) {
      request_headers.setReferenceForwardedProto(
          connection.ssl() ? Headers::get().SchemeValues.Https : Headers::get().SchemeValues.Http);
    }
  } else {
    // If we are not using remote address, attempt to pull a valid IPv4 or IPv6 address out of XFF.
    // If we find one, it will be used as the downstream address for logging. It may or may not be
    // used for determining internal/external status (see below).
    auto ret = Utility::getLastAddressFromXFF(request_headers, xff_num_trusted_hops);
    final_remote_address = ret.address_;
    single_xff_address = ret.single_address_;
  }
  // If the x-forwarded-proto header is not set, set it here, since Envoy uses it for determining
  // scheme and communicating it upstream.
  if (!request_headers.ForwardedProto()) {
    request_headers.setReferenceForwardedProto(connection.ssl() ? Headers::get().SchemeValues.Https
                                                                : Headers::get().SchemeValues.Http);
  }
```

내부에 x-forwarded-proto가 없는경우 생성하는 방어로직이 여러 곳에 들어가있다.
엔보이의 소스코드를 건드려서 새로 컴파일하지 않는이상 파드로 떨궈지는 x-forwarded-proto 헤더는 운명이다.

그러므로 x-forwarded-proto를 삭제하려는 행위보다 윗단에서부터 잘 내려주도록 세팅해야한다.

이를 프록시로 인지하도록하는 방법은 여러가지가 있는데
istio의 pilot컨피그를 수정해도되고 `--set pilot.env.PILOT_SIDECAR_USE_REMOTE_ADDRESS`  
envoyfilter를 이용해 `use_remote_address`, `xff_num_trusted_hops`를 조작할 수도 있다.
이를 통해 이스티오는 자신보다 앞단에 프록시서버가 있다는 것을 인지할 수 있다.


```
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: xff-trust-hops
  namespace: istio-system
spec:
  workloadSelector:
    labels:
      istio: ingressgateway
  configPatches:
  - applyTo: NETWORK_FILTER
    match:
      context: ANY
      listener:
        filterChain:
          filter:
            name: "envoy.http_connection_manager"
    patch:
      operation: MERGE
      value:
        typed_config:
          "@type": "type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager"
          use_remote_address: true
          xff_num_trusted_hops: 1
```

엔보이필터를 활용하면 윗단에서 내려주는 x-forwarded-proto 헤더를 파드에게 전달할 수 있다.

앞단 프록시 서버에서 x-forwarded-proto를 내려주는 세팅을 하면!

```
[root@pransible resources]# curl -X GET devops-test-server.alpha.test.bz -H x-forwarded-proto:https
[GIN] 2019/12/03 - 11:29:27 | 200 |      47.873µs |    172.16.70.25 | GET      /

map[Accept:[*/*] Content-Length:[0] User-Agent:[curl/7.29.0] X-B3-Parentspanid:[1ae62ab0db06b9d8] X-B3-Sampled:[0] X-B3-Spanid:[59241862fb823195] X-B3-Traceid:[f2d9ff306bb45b351ae62ab0db06b9d8] X-Envoy-External-Address:[~] X-Forwarded-For:[~] X-Forwarded-Proto:[https] X-Request-Id:[e7ccda65-e493-49a5-8b00-b8986d2c5f04]]
```

잘들어온다!

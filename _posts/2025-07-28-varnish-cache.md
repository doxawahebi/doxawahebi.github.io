---
title: varnish server cache
description: 
author: doxawahebi
date: 2025-07-17 15:39:00 +0900
categories: [web]
tags: [web, varnish, reverse-proxy, cache]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
---


## Overview
---
varnish는 HTTP 가속기라 불리는 리버스 캐싱 프록시이다. 작동 방식은 다른 웹 캐시들과 비슷하다. (이전 web cache 또는 nginx 글 참조)

아래 명령어를 통해 varnish를 시작할 수 있다.
```shell
sudo varnishd -f /etc/varnish/default.vcl
```

## VCL
---
Varnish Cache의 핵심은 VCL(Varnish Congifuration Language)의 유연성에 있다.
VCL을 사용하여 정책을 작성하고 요청과 응답 프로세스의 모든 측면을 제어할 수 있습니다.  VCL를 실행하여 얼마나 오래 캐시할 지 해당 콘텐츠가 어느 백엔드에 있는 지 등등을 결정합니다.
VCL은 C로 변환히고 SO(공유 오브젝트)로 컴파일되고 Varnish에 직접 로드되기 때문에 재시작없이 구성되며 빠릅니다.

### VCL 문법
---
`set`과 `unset`을 통해 변수를 설정하거나 지울 수 있다. 그리고 변수에도 여러 타입이 존재한다.

```
if (req.url == "/mistyped_url.html") {
    set req.url = "/correct_url.html";
    unset req.http.cookie;
}
```

그 이외에도 if문, re 등을 사용할 수 있다.

아래에서 VCL의 문법을 배울 수 있다.
https://varnish-cache.org/docs/7.7/reference/vcl.html

이 링크에서는 변수들에 대한 정의가 있다. 이 변수들을 통해 request나 response의 헤더 등을 수정할 수 있다. 
https://varnish-cache.org/docs/7.7/reference/vcl-var.html

참고로 변수 중 bereq는 원래 요청으로부터 생성된 백엔드 요청이다.


### Varnish Processing States
---
클라이언트와 백엔드 요청에 대한 처리는 상태 머신으로 구현되어 있다. 상태에 진입할 때마다 
C함수를 호출합니다. 이 함수는 해당 단계에서 요청이나 응답을 처리하기 위해 적절한 Varnish core code 함수를 합니다. 대부분의 상태에서 코어 코드는 VCL에서 컴파일된 상태별 함수인 VCL 서브루틴도 호출합니다.

아래 링크에 다음 그래프를 참조하면 상태 머신이 어떤 것들 참조하고 어떤 흐름으로 작동하는 지 알 수 있습니다. 서브루틴들이 어떤 순서로 작동하는지 파악할 수 있다.

https://varnish-cache.org/docs/7.7/reference/states.html#reference-states

아래 링크에서 좀 더 보기 쉬운 그래프를 볼 수 있고 좀 더 쉽게 문법들을 배울 수 있다. 화살표의 색깔에 따른 의미도 여기서 볼 수 있다.
https://www.varnish-software.com/developers/tutorials/varnish-configuration-language-vcl/

### VCL Steps
---
`vcl_recv`와 같은 것들이 코어 코드가 상태에 따라 호출하는 빌트인 서브루틴입니다. 

각 서브루틴 특정 키워드를 `return()`으로 부르는 것으로 종료하고 다음 상태로 넘어갈 수 있다.

`pass`를 사용하여 pass 모드로 전환되면 현재 요청이 캐시를 사용하지 않고 응답을 캐시에 넣지 않도록 만들어집니다. 흐름은 `vcl_pass`로  넘깁니다.

`vcl_hash`는 요청에 대한 해시값을 만들고 이 해시값을 바니시에서 객체를 찾는 키로 사용합니다. `hash_data()`를 사용해 어떤 변수를 해시키로 사용할 지 결정할 수 있다.
`lookup` 키워드를 통해 cache가 hit인지 miss인지 등을 판단합니다. `hash`를 통해 cache status를 정하기 때문에 그래프에서 위 화살표들이 hash까지는 경로가 같은 것이다.

https://varnish-cache.org/docs/7.7/reference/vcl-step.html#id7

## X-Cache
---
다음을 설정하면 클라이언트로 전해지는 응답을 통해 캐시의 상태를 알 수 있다.

```
sub vcl_deliver {
    if (obj.hits > 0) {
        set resp.http.X-Cache = "HIT";
    } else {
        set resp.http.X-Cache = "MISS";
    }
    return (deliver);
}
```

## Cookie
---
Varnish는 `Set-Cookie` header가 있는 응답을 캐시하지 않습니다. 또한 **클라이언트가 `Cookie` 헤더를 보낸다면 Varnish는 캐시를 bypass**할 것 입니다.

(아래링크에서는 필요없는 쿠키를 제거하는 방법을 배울 수 있다.)
https://varnish-cache.org/docs/7.7/users-guide/increasing-your-hitrate.html#cookies
https://www.varnish-software.com/developers/tutorials/removing-cookies-varnish/
## Vary header
---
`Vary` 헤더가 존재하면 캐시되어 있고 해시값이 같은 다른 요청이 존재해도 클라이언트에게 그 캐시된 응답이 반환되지 않을 수 있다.

다음을 통해 응답에서 `Vary` 헤더를 제거하면 문제가 해결된다.
```
sub vcl_backend_response {
  unset beresp.http.Vary;
  ...
}
```

https://ma.ttias.be/varnish-hash-different-results-check-vary-header/

## varnish logging
---
아래와 같은 명령으로 로그를 볼 수 있다.

```shell
varnishlog -b -g request | grep -e 'timestamp' -e 'BereqURL' -e 'BereqMethod'
```

로그는 자체 형식이나 NCSA Common log format 형식을 따른다.

구체적으로 알고 싶다면 다음을 참고하자
https://varnish-cache.org/docs/7.7/reference/varnishlog.html#varnishlog
## Reference
---
https://en.wikipedia.org/wiki/Varnish_(software)

https://varnish-cache.org/docs/7.7/tutorial/introduction.html
https://varnish-cache.org/docs/7.7/users-guide/index.html#users-guide-index

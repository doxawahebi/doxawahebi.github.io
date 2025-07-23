---
title: nginx server cache
description: 
author: doxawahebi
date: 2025-02-25 15:39:00 +0900
categories: [web]
tags: [web, nginx, reverse-proxy, cache]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
---


## Web cache
---
웹 캐시 (서버)는 오리진 서버(백엔드 서버)와 클라이언트 사이에 존재하는 시스템(서버)입니다. 클라이언트가 정적 자원을 요청하면 일단 캐시 서버로 요청이 전송된다. 만약 캐시 서버가 리소스의 사본을 갖고 있지 않다면(cache miss)
요청은 오리진 서버로 갑니다. 오리진 서버가 요청을 처리하고 응답을 캐시 서버로 전송합니다. 
캐시 서버는 미리 설정된 룰에 맞춰 **응답을 저장**할지 말지 결정합니다. 
이후 클라이언트가 요청을 보냈을 때 캐시에 저장된 자원을 요청했다면 백엔드 서버에 요청을 보내지 않고 클라이언트에 저장한 콘텐츠를 직접 리턴한다(cache hit). 
이로 인해 요청받을 때마다 백엔드 서버가 매번 페이지를 생성할 필요가 없기에 애플리케이션 서버를 더 효율적으로 사용하고 퍼포먼스를 향상시킬수있다.

웹 브라우저와 애플리케이션 서버 사이에는 클라이언트의 브라우저 캐시, 중간 캐시, CDN, 로드 밸런서 또는 리버스 프록시와 같은 여러 캐시가 있습니다.


## cache key
---
캐시는 HTTP 요청을 받았을 때 캐시된 응답이 있는지 확인하기 위해 캐시 키를 사용합니다. 캐시 키는 HTTP 요청의 요소들로 만들어집니다. 전형적으로 URL path와 query parameter로 만들어지지만 **다른 헤더들(특히 vary)이나 컨텐츠 타입을 포함하기도 합니다.**
만약 요청의 캐시 키가 이전 요청의 캐시 키와 일치하다면, 캐시는 클라이언트에게 캐시된 응답을 보낼 것입니다.


## cache rules
---
캐시룰은 무엇을 캐시할지 얼마나 오래 캐시할지 정하는데 사용합니다. 캐시룰들은 쉽게 변하지 않고 자주 재사용되는 정적 리소스를 저장하도록 종종 설정합니다. 동적 콘텐츠들은 사용자마다 개인화되어있고 민감한 정보들을 포함하고 있을 확률이 높으므로 캐시하지 않고 직접 백엔드 서버한테서 데이터를 받아오도록 설정을 한다. 

Web cache deception 공격은 설정된 캐시 규칙을 악용한다. 일반적으로 아래와 같이 설정된 룰을 이용한다.

- 정적 파일 확장자 룰 - 요청된 리소스의 파일 확장자와 일치하는지로 판단(예를 들어, 스타일시트(`.css`)나 자바스크립트 파일(`.js` ))
- 스태틱 디렉토리 룰 - 특정 접두사로 시작하는 URL와 매치시킨다. 예를 들어, `/static`, `/assets`와 같이 정적 리소스들만 저장한 특정 디렉토리를 타겟으로 하는데 종종 사용된다.
- 파일 이름 룰 - 특정 파일 이름과 매치되면 캐시한다. 예를 들어, 다음과 같은 파일을 자주 캐시한다. (`robots.txt`, `favicon.ico`)

## How to Set Up and Configure Basic Caching
---
`proxy_cache_path`와 `proxy_cache` 두 가지만 있다면 캐시를 사용할 수 있다.

`proxy_cache_path`는 캐시의 path와 configuration를 설정하는 디렉티브고 `proxy_cache`는 캐시를 활성화하는 지시어입니다.

```nginx
proxy_cache_path /path/to/cache levels=1:2 keys_zone=my_cache:10m max_size=10g 
                 inactive=60m use_temp_path=off;

server {
    # ...
    location / {
        proxy_cache my_cache;
        proxy_pass http://my_upstream;
    }
}
```

proxy_cache_path에 대한 parameter들은 다음 설정을 정의한다.

- **/path/to/cache/** 는 캐시를 위한 로컬 디스크 디렉토리이다.
- 위처럼 설정하면 levels는 **/path/to/cache/** 아래에 2레벨 디렉토리 계층을 설정합니다. (**/path/to/cache/0/11/**) 한 디렉토리에 너무 많은 파일이 있으면 파일 엑세스가 느려질 수 있으므로 일반적으로 2레벨을 권장한다. 
- `keys_zone`는 캐시 키나 usage timers와 같은 메타데이터를 저장하는 공유 메모리 영역을 설정합니다. 디스크가 아닌 메모리에 키의 복사본이 있으면 요청이 HIT인지 MISS인지 빠르게 판단할 수 있다. 1MB 정도면 대략 8000개 정도의 키에 대한 데이터를 저장할 수 있다.
- `max_size`는 캐시 사이즈에 대한 한도를 설정합니다. 이를 설정하지 않으면 캐시가 사용가능한 모든 디스크 공간을 사용합니다. 캐시 사이즈가 한도에 달하면 캐시 매니저라는 프로세스가 제한이하로 줄이기 위해 최근 사용되지 않는 파일들을 제거합니다.
- `inactive`는 항목이 액세스되지 않고 캐시에 남아있을 수 있는 시간을 설정합니다. 예시에서는 60분이 지나면 expired 여부와 관계없이 cache manager process가 캐시에서 삭제하도록 되어있습니다. nginx는 cache control 헤더에 의해 정의되고 만료된 콘텐츠를 자동으로 삭제하지 않습니다. (예를 들어, `Cache-Control:max-age=120`)  오직 inactive가 지정한 시간 동안 콘텐츠가 접근되지 않은 경우에만 삭제가 됩니다. 그리고 만료된 콘텐츠가 접근되면 nginx는 그 콘텐트를 원래 서버를 통해 리프레쉬하고 `inactive` 타이머를 리셋합니다.
- nginx는 먼저 캐시에 저장될 파일을 임시 저장소에 작성합니다. `use_temp_path=off`디렉티브는 임시 저장소에 따로 작성하지 않고 파일을 원래 캐시될 디렉토리에 작성하도록 만든다. 불필요한 데이터 복사를 방지하기 위해 off로 설정하는 것을 추천한다.
`proxy_cache`는 parent location block(예시에서는 `/`이다.)의 URL와 매치되는 모든 콘텐츠를 캐시하도록 활성화하는데 사용합니다. 쉽게 말해 cache할 콘텐츠가 있는 location block 내부에 작성하면 된다. nginx의 상속이 어떻게 이루어지는 알고 있다면 당연한 내용이지만 server에 `proxy_cache`를 작성하면 자체적으로 `proxy_cache`를 가지고 있는 location block를 제외한 모든 location block에 서버에서 작성한 `proxy_cache`가 그대로 적용된다.


## Cache-Status
---
아래 디렉티브를 추가하면 cache status(MISS, HIT 등)를 클라이언트가 응답에 헤더로 볼 수 있다.
```
add_header X-Cache-Status $upstream_cache_status;
```


## How Does NGINX Determine Whether or Not to Cache Something?
---
NGINX는 백엔드 서버가 보낸 response가 (미래에 대한) 날짜와 시간이 포함된 `Expires` 헤더나 (0으로 설정되어 있지 않은) `max-age`를 가지고 있는 `Cache-Control` 헤더를 포함한 경우에만 response를 캐시합니다. 

기본적으로, NGINX는 `Cache-Control` 헤더는 다른 디렉티브를 존중합니다. `Private`, `No-Cache`, or `No-Store` 디렉티브가 헤더에 포함되면 response를 캐시하지 않습니다. 
**`Set-Cookie`가 포함된 응답은 캐시되지 않습니다.**
기본적으로 GET, HEAD method에 대한 response만 캐시합니다. `proxy_cache_methods`를 통해 override할 수 있습니다.


### proxy_ignore_headers
---
```nginx
location /images/ {
    proxy_cache my_cache;
    proxy_ignore_headers Cache-Control;
    proxy_cache_valid any 30m;
    # ...
}
```

proxy_ignore_headers 디렉티브를 사용하면 
`Cache-Control` Headers를 무시할 수 있으며 이를 통해 아까 말한 nginx 캐시 정책을 무시할 수 있다. 여기서 **중요한 점은 `Cache-Control` headers를 무시하면  `proxy_cache_valid` 디렉티브를 꼭 작성해줘야한다.** 이 디렉티브는 캐시된 데이터에 expiration(만료일)를 강요한다. nginx는 expiration이 없는 파일들은 캐시하지 않는다. 

valid와 proxy_ignore_headers에 관한 내용은 아래 링크에 나와있다.
https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_valid
### (관찰에 따른 추측)
---
ignore이면 정책에 영향을 주지 않는다는 의미일뿐 unset(해당 헤더를 삭제)으로 만드는 것은 아닌듯하다.

이 키워드를 사용하면 위에 캐시 정책 (`Set-Cookie`는 캐시하지 않음 등등)을 무시하는데 사용할 수 있다. 

백엔드 서버의 응답에 vary 헤더가 있으면 다르게 캐시하는 것 같다. vary와 같이 캐시키를 설정하는데 영향을 주는 헤더들이 존재하는 것 같다.

web cache deception 취약점을 구현하고 싶다면 아래와 같이 설정하면 좋다. `vary` 헤더나 그 이외에 캐시키에 영향을 주는 헤더들을 무시하고 response를 캐시하도록 만들 수 있다.

`proxy_ignore_headers Expires Set-Cookie Vary;` 

(위 설정을 하면 다른 브라우저나 다른 IP에서 접근해도 같은 캐시를 반환하도록 만들어진다.)
## What Cache Key Does NGINX Use?
---
nginx가 생성하는 키들의 기본 형태는 `scheme$proxy_host$request_uri`를 MD5로 해시시킨 것와 비슷합니다.(아마 같지는 않을 것 입니다. 좀 더 복잡하게 헤더의 영향이나 알고리즘이 존재하는 것 같습니다) (URI가 같아도 다른 브라우저로 들어가보면 같은 URI로  한 브라우저에서 cache hit된 콘텐츠가 응답되지 않는다)

```nginx
location / {
        proxy_cache my_cache;
        proxy_pass http://my_upstream;
}
```

위 설정을 예로 들면, `http://www.example.org/my_image.jpg`에 대한 캐시키는 다음으로 계산될 것 입니다.
 `md5(“http://my_upstream:80/my_image.jpg”)`

캐시키에 대한 룰을 바꾸고 싶다면 `proxy_cache_key` 디렉티브를 사용하면 된다.




## cache
---

```
location ~* \.(css|png)$ {
    proxy_cache nginxcache;
    proxy_pass http://server:80;
    proxy_cache_valid 200 302 1d;
    proxy_cache_valid 404 1m;    
}
```

`location ~* \.(css|png)$` : `.css` 또는 `.png`로 끝나는 요청을 처리하는 location block

`proxy_cache nginxcache` : `nginxcache`라는 이름의 프록시 캐시 영역을 사용하겠다는 의미

`proxy_pass http://server:80` : 
request를 `http://server:80`으로 넘긴다.

`proxy_cache_valid 200 302 1d` : 응답 코드가 `200` 또는 `302`이면 해당 응답을 1일 동안 캐시한다.
## Character
---

### Time complexity
---


## Application
---



## Reference
---
https://blog.nginx.org/blog/nginx-caching-guide
https://docs.nginx.com/nginx/admin-guide/content-cache/content-caching/
https://portswigger.net/web-security/web-cache-deception

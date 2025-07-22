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
캐시 서버는 미리 설정된 룰에 맞춰 응답을 저장할지 말지 결정합니다. 
이후 클라이언트가 요청을 보냈을 때 캐시에 저장된 자원을 요청했다면 백엔드 서버에 요청을 보내지 않고 클라이언트에 저장한 콘텐츠를 직접 리턴한다(cache hit). 
이로 인해 요청받을 때마다 백엔드 서버가 매번 페이지를 생성할 필요가 없기에 애플리케이션 서버를 더 효율적으로 사용하고 퍼포먼스를 향상시킬수있다.

웹 브라우저와 애플리케이션 서버 사이에는 클라이언트의 브라우저 캐시, 중간 캐시, CDN, 로드 밸런서 또는 리버스 프록시와 같은 여러 캐시가 있습니다.

## nginx cache
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




## Reference
---



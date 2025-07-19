---
title: nginx server
description: 
author: doxawahebi
date: 2025-02-25 15:39:00 +0900
categories: [web]
tags: [web, nginx, reverse-proxy, cache]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
---


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



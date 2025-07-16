---
title: nginx server
description: 
author: doxawahebi
date: 2025-02-25 15:39:00 +0900
categories: [web]
tags: [web, nginx]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
---

## `proxy_set_header`
---
proxy_set_header 디렉티브는 특정 헤더를 재정의하거나 전달하기 위해 사용한다.

nginx가 request를 보낼 떄 아래와 같이 두 헤더를 기본적으로 재정의한다.
```
Host: $proxy_host
Connection: close
```

다음과 같은 방식으로 정의할 수 있다. proxy_set_header를 location 블록에 상위에 작성하면 해당 헤더 필드들은 location에 상속되어 전달된다.
```
proxy_set_header foo foo;
proxy_set_header bar bar;

location /some/path {
	# `foo`와 `bar` 헤더가 상속되어 전달된다.
	proxy_pass http://localhost:8000;
}
```

이때 주의할 점이 있는데 다음과 같이 상위, 하위 모두에 `proxy_set_header`를 사용하면 상위에 정의된 내용은 무시되고 하위에 정의된 헤더만 적용된다는 것이다.
```
# 상위에 정의된 헤더
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;

location /some/path {
	# 하위에 정의된 헤더만 적용됨
	proxy_set_header foo foo;
	proxy_pass http://localhost:8000;
}
```



## nginx가 Request를 처리하는 방법
---

nginx는 server block을 선택할 때 request 헤더의 `Host` 필드를 가지고 판단한다. Host의 값이 server_name에 일치하지 않거나 Host field가 존재하지 않는다면 메시지는 default_server로 전송된다.

``
```
In this configuration nginx tests only the request’s header field “Host” to determine which server the request should be routed to. If its value does not match any server name, or the request does not contain this header field at all, then nginx will route the request to the default server for this port.
```
(https://nginx.org/en/docs/http/request_processing.html)

## `$host`
---
`$host`는 다음 우선순위로 값이 정해진다. `request line` -> `request header의 Host field` -> `request에 매칭되는 server name`

```
in this order of precedence: host name from the request line, or host name from the “Host” request header field, or the server name matching a request
```
(https://nginx.org/en/docs/http/ngx_http_core_module.html)

ex) 
```
## request line
GET http://www.example.com/page HTTP/1.1
...

## Host field
...
Host: www.example.com
...
\r\n\r\n
```

다음은 server name으로 정해지는 경우이다.
nginx에서 다음과 같은 설정이 되어 있다고 하자.
```
server {
        listen 80 default_server;
		server_name www.example.com;

        location / {
			proxy_pass http://localhost:3000;
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Forwarded-Proto $scheme;
        }
}
server {
        listen 80 default_server;
		server_name www.example2.com;

        location / {
			proxy_pass http://localhost:5000;
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Forwarded-Proto $scheme;
        }
}
```

아래와 같이 Host 헤더가 없이 요청을 보내면 메시지가 default_server로 설정된 `http://localhost:3000`로 라우팅된다.
```
GET / HTTP/1.1
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
...
\r\n\r\n
```







## Reference
---
https://sculove.github.io/post/nginx-reverse-proxy/
https://ohgyun.com/537

https://nginx.org/en/docs/http/request_processing.html
https://nginx.org/en/docs/http/ngx_http_core_module.html



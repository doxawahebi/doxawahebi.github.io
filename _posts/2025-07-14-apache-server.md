---
title: apache server setting
description: 
author: doxawahebi
date: 2025-02-25 15:39:00 +0900
categories: [web, apache_server]
tags: [web, apache_server]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
---

## port 변경
---
```shell
vi /etc/apache2/ports.conf
```

```shell
vi /etc/apache2/sites-enabled/000-default.conf
```
## log 확인

### access.log
---
서버에 request를 보낸 사용자들을 알고 싶을 때 사용한다.
```shell
tail -f /var/log/apache2/access.log
```

### error_log


## reverse proxy
---


### reverse proxy vs redirect
---


Location tag를 이용하면 다음과 같이 간단하게 설정할 수 있다.
```xml
<Location /myapp>
	ProxyPass http://127.0.0.1:5000/
	ProxyPassReverse http://127.0.0.1:5000/
	RequestHeader set X-Forwarded-Proto http
	RequestHeader set X-Forwarded-Prefix /myapp
	Require all granted
</Location>
```

- ProxyPass : 
- RequestHeader

- `RequestHeader set X-Forwarded-Proto http`

RequestHeader를 사용하려면 `mod_headers` 모듈을 활성화해야한다.

```
a2enmod headers
systemctl restart apache2
```

클라이언트가 Apache에 접속한 **프로토콜(HTTP or HTTPS)** 을 백엔드에 전달합니다.
백엔드 서버는 이 값을 보고 리디렉션, URL을 생성할 시 `http://` 또는 `https://`를 결정합니다.


`X-Forwarded-Prefix`를 설정해야하는 이유는 다음과 같다.

백엔드에게 프록시 경로의 접두어를 알려줍니다. 이를 통해 백엔드가 링크, 리다이렉트 등을 만들 때 설정한 prefix(ex. `/myapp`)를 prefix로 붙인다.

`/myapp/question/list/`라는 페이지 요청했다고 했을 떄 다음과 같은 링크가 온다고 하면 `X-Forwarded-Prefix`를 설정했을 떄는 다음과 같이 설정된다.
```html
<a href="/myapp/question/create/" class="btn btn-primary">질문 등록하기</a>
```

설정하지 않으면 다음과 같이 온다.
```html
<a href="/question/create/" class="btn btn-primary">질문 등록하기</a>
```

차이점을 보면 알 수 있겠지만 `/myapp`이 prefix로 붙지 않기에 링크를 타거나 할 때 문제가 발생한다.

### 주의할 점
---
백엔드 서버에서 앞에 프록시 서버가 있다는 것을 알려줘야한다.

flask의 경우 다음과 같이 설정한다.
```python
from werkzeug.middleware.proxy_fix import ProxyFix

app.wsgi_app = ProxyFix(app.wsgi_app, x_proto=1, x_prefix=1)

```

## Application
---

## ubuntu apache2 디렉토리 이해하기
---
  
/etc/apache2

설정파일 위치

  

/etc/apache/apache2.conf

cent os 에서는 httpd.conf를 설정팡일로 했으나 우분투는 apache2.conf를 쓴다.

  

/etc/apache2/conf-available  :  사용 가능한 구성 파일

/etc/apache2/conf-enabled  :  사용되는 구성파일

/etc/apache2/mods-available :  사용가능 모듈 디렉토리    가령 )   a2enmod  모듈명.conf  하면 해당모듈이  로드된다.

/etc/apache2/mods-enabled :  로드될 모듈

/etc/apache2/sites-available  : **Apache HTTP 서버**의 기본 가상 호스트 설정 파일입니다. Ubuntu 및 Debian 계열 리눅스 시스템에서 사용됩니다.

/etc/apache2/sites-enabled :  실제 로드될 파일

/etc/apache2/envvars   : apache2ctl 환경설정 파일

/etc/apache2/magic : 파일의 MIME TYPE결정,가급적 수정하지말것 

/etc/apache2/ports.conf : 서비스 포트설정 파일

  

apache2 에서는 이렇게 모듈화 되서 굉장히 편한느낌이다

  

a2ensite test.conf    <-- virtual host 생성 및 module 설정
a2dissite test.conf    <--  비활성화하기


  

디렉토리 중   a2명령어로  실행~

명칭-available :  소스파일들 !

명칭-enabled :  심볼릭 ! 실제 로드될때 실행된다
https://hotwebtech.tistory.com/8

### directives
---
`VirtualHost 172.0.0.1:80` : 172.0.0.1의 80번 포트로 들어오는 요청만 처리한다. `*`를 사용하면 제한없이 받아들인다.

`Require all granted` : 
### command
---
`sudo a2ensite 000-default` 설정 활성화 (enabled 디렉토리에 링크 생성)
`sudo a2dissite 000-default` 설정 비활성화
`sudo systemctl reload apache2` 설정 변경 후 재적용
`sudo apache2ctl configtest` 설정 파일 문법 검사
a2enmod 모듈명   < -- 모듈활성화
a2dismod 모듈명   < -- 비활성
## Reference
---


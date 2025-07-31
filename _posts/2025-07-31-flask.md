---
title: flask-login
description: 
author: doxawahebi
date: 2025-07-17 15:39:00 +0900
categories: [web]
tags: [web, login, flask-login]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
---


## Overview
---
Flask-Login은 플라스크에서 유저 세션을 관리하기 위한 기능을 제공한다. 

다음과 같은 것이 된다.
- Flask Session에 활성화된 user's ID를 저장하고 쉽게 로그인하거나 로그아웃할 수 있게 합니다.
- 로그인한 사람(또는 로그인하지 않은)만 view를 볼 수 있도록 제한할 수 있다. (`login_required`)
- 까다로운 로그인 상태 유지(remember me) 기능을 처리한다. (브라우저를 닫아도 로그인이 유지되도록 설정한다는 이야기이다. 로그인할 때 종종 로그인 상태 유지라는 체크박스를 볼 수 있다.)
- 쿠키 도둑으로부터 사용자의 세션이 뺏기지 않도록 보호한다.

다음과 같은 것은 불가능하거나 직접 구현해야한다.
- 특정 DB를 사용하도록 강요하지 않으며 직접 유저 정보를 어떻게 로드할지 직접 설정해야한다. (`load_user`)
- `Flask-Login`은 사용자의 로그인 여부의 중점을 두었기에 로그인 과정에서 인증 방식(`id/pw`, `OTP` 등)은 직접 구현해야합니다.
- Role(역할)에 따른 authorization(권한 부여), Access Control(접근제어) 등은 직접 구현해야한다. (`Flask-Principal`, `Flask-Security`, `Flask-User`와 같은 확장 기능을 사용하여 구현할 수 도 있다.)
- 회원가입, 비밀번호 찾기, 이메일 인증 등의 기능은 직접 구현해야한다. (`Flask-User`, `Flask-Authlib`)




## Application 구성
---
Flask-Login을 사용할 때 가장 중요한 것은 `LoginManager` class이다.  다음과 같은 코드를 애플리케이션 어딘가에 작성해야한다.

```python
from flask_login import LoginManager
login_manager = LoginManager()
```

```python
login_manager.init_app(app)
```

ID로부터 사용자를 로드하는 방법, 

Flask-Login은 기본적으로 인증을 위해 세션을 사용한다. 이를 다르게 말하면 `secret key`를 설정해야한다는 의미이다. 
(`app.secret_key = b'_5#y2L"F4Q8z\n\xec]/'`)




## user_loader()
---
세션에는 user 객체가 아닌 user ID가 저장되어 있기 때문에 객체를 리로드하기 위해서 `user_loader` callback을 작성해줘야한다.

`user_loader()`를 통해
`session['user_id']`의 값을 통해 `load_user(user_id)`를 호출해서 User 객체를 복원하고 `current_user`에 할당합니다.

Example)
```python
@login_manager.user_loader
def load_user(user_id):
    return User.get(user_id)
```

이때 ID가 유효하지 않으면 `None`을 리턴해야한다.


### 내부 코드
---

```python
class LoginManager:
	...
	def user_loader(self, callback):
        """
        This sets the callback for reloading a user from the session. The
        function you set should take a user ID (a ``str``) and return a
        user object, or ``None`` if the user does not exist.

        :param callback: The callback for retrieving a user object.
        :type callback: callable
        """
        self._user_callback = callback
        return self.user_callback
        @property
    def user_callback(self):
        """Gets the user_loader callback set by user_loader decorator."""
        return self._user_callback



```

(아마 다 알고 있겠지만 데코레이터는 함수를 파라미터로 받는 함수를 syntax sugar로 작성한 것이다.)

user_loader를 통해 `_user_callback`을 설정하며 `user_callback` property를 통헤 설정한 콜백을 가져올 수 있다.


`login_user(user)`는 사용자를 로그인 상태로 만듭니다. 내부적으로 `session['user_id'] = user.get_id()`를 통해 사용자 ID를 세션에 저장합니다. (User 객체를 전달해야하지만 User 객체는 저장되지 않는다.)



## login_

## current_user
---
```python
def _load_user(self):
	...
	        # Load user from Flask Session
        user_id = session.get("_user_id")
        if user_id is not None and self._user_callback is not None:
            user = self._user_callback(user_id)
    ...
```

## remember me
---
```
login_user(user, remember=True)
```

`remember=True`로 설정하면 브라우저를 닫아도 로그인 상태를 유지하게 만든다. 
`remember_token`이라는 쿠키를 브라우저에 저장한다. 

`"user_id|timestamp|hash"`

`SECRET_KEY`

세션은 사라져도 클라이언트는 `remember_token` 쿠키를 서버에 보낸다.

`user_loader`가 세션에 사용자 정보가 없으면 `token_loader`를 통해 사용자 인증을 시도한다.

쿠키가 유효하면, Flask-Login은 `login_user()`를 내부적으로 호출해서 다시 로그인시킴.

토큰은 암호화되고 서명됨.
`app.config['REMEMBER_COOKIE_DURATION'] = timedelta(days=7)`

## Application
---



## Reference
---
https://flask-login.readthedocs.io/en/latest/



---
title: howdy로 얼굴 인식 기능 사용하기
description: 
author: doxawahebi
date: 2025-02-13 20:26:00 +0900
categories: [linux, ubuntu]
tags: [linux, ubuntu, howdy, face]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
---

## Overview
---

## install howdy
```shell
sudo apt install howdy
```

```shell
sudo howdy add
```

```shell
sudo apt install python3-numpy python3-opencv python3-dlib
```

```shell
lsusb
```

```shell
ls /dev | grep "video."
```

```shell
sudo apt install ffmpeg
```

```shell
ffplay /dev/video0
```

```shell
sudo howdy config
```
`device_path`를 `/dev/video0`을 기입한다.

프로필 추가하기
```shell
sudo howdy add
```

```shell
sudo howdy list
```

sudo 명령어
`/etc/pam.d/sudo`

로그인 화면
`/etc/pam.d/gdm-password`
최상단에 기입하여 howdy 모듈 추가
```
auth sufficient pam_python.so /lib/security/howdy/pam.py 
```

`certainly`가 높을수록 정확도가 떨어짐.
## Reference
---
https://www.linuxtechmore.com/2023/10/how-to-set-up-facial-authentication-on-linux.html
https://support.system76.com/articles/setup-face-recognition/
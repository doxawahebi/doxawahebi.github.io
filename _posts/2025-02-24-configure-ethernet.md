---
title: ubuntu에서 이더넷 설정하기
description: 
author: doxawahebi
date: 2025-02-13 20:26:00 +0900
categories: [linux, ubuntu]
tags: [linux, ubuntu, ethernet, c-to-ethernet]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
---

## Overview
---
노트북에 USB-C to Ethernet을 이용해 이더넷 설정을 하는 방법을 알아보겠습니다.

## 디바이스가 감지되어 있는 지 확인
---

다음 명령어를 통해 어댑터가 감지되어 있는 지 확인합니다.
```shell
lsusb
```

adapter가 감지되지 않는다면 드라이버 설치를 해주셔야합니다.

## 드라이버 설치
---
해당 제품의 공식 홈페이지에 들어가면 드라이버가 있습니다. 다운받아줍니다.

tar 형식에 파일일텐데 파일에 압축을 풀면 나오는 디렉토리가 있습니다.
그 디렉토리에 다음 명령어를 쳐줍니다. 
```shell
make
```
드라이버가 컴파일될텐데 이 드라이버를 설치해줍니다.
```shell
sudo make install
```

드라이버 로드를 한다.
```shell
sudo modprobe ax_usb_nic
```

드라이버가 제대로 설치되었는 지 확인한다.
```shell
modinfo ax_usb_nic 
```

<!-- make -C /lib/modules/$(uname -r)/build M=$(CUR_DIR) -->

## 네트워크 인터페이스 상태 확인
---

다음 명령어로 네트워크 인터페이스의 상태를 확인합니다.
```shell
ip link
```
`NO-CARRIER`로 표시되어있다면 다음 명령어를 쳐줍니다.

```shell
ethtool -s eth0 speed 100 duplex full
```

Settings -> Network에 들어가보면 Wired 부분에서 이더넷이 활성화된 것을 볼 수 있다.

## Reference
---
https://askubuntu.com/questions/497850/eth0-no-carrier-ifconfig-shows-no-ip-address

---
title: Blog 꾸미기
description: 
author: doxawahebi
date: 2025-02-12 21:26:00 +0900
categories: [Jekyll]
tags: [github-pages, jekyll, chirpy-theme]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
---


## 댓글 추가하기
---
댓글 기능을 위해 사용할 수 있는 외부 댓글 서비스는 `disqus`, `utterances`, `giscus` 정도 있다.

### giscus를 이용하는 방법

이 [사이트](https://giscus.app/ko) 들어가보면 대충 알 수 있다.

간단하게 설명하면
giscus를 사용하기 위해서는 일단 저장소의 discussion을 생성해야됩니다.
discussion을 생성할려면 enable을 해야되는데

enable하는 방법은 Repository에 들어가서
Setting -> General -> Features를 보면
Discussions라는 항목이 있는데 이것을 체크해주면 enable 됩니다.

Discussions 탭에 들어가서 New discussion을 만들어줍니다.
(만들 때 Discussions은 Announcements 유형을 권장합니다.)


giscus를 [설치](https://github.com/apps/giscus)합니다. 저는 설치할 때 한 저장소에만 적용시켰습니다.

![Desktop View](/assets/img/posts/20250213/install-giscus-1.png){: width="700" height="300" }

![Desktop View](/assets/img/posts/20250213/install-giscus-2.png){: width="700" height="300" }

설치한 다음에 이 [사이트](https://giscus.app/ko)에서 설정 부분 따라하면 다음과 같이 코드가 나오는데 이걸 복사헤줍니다.

![Desktop View](/assets/img/posts/20250213/giscus-script.png){: width="700" height="300" }

대충 적용하는 방법이 2개가 있는데 `_layouts`폴더의 `post.html`에 가장 밑에 추가하거나 `_config.yml`을 수정하는 방법이 있습니다.
취향껏하시면 된는는데 개인적으로 내장으로 존재하는 `_config.yml`에 추가하는 게 좋아서 다음과 같이 적용했습니다.

![Desktop View](/assets/img/posts/20250213/config-giscus.png){: width="700" height="300" }

### utterances을 이용한 방법

이 [사이트](https://utteranc.es/?installation_id=60940545&setup_action=install)를 들어가보면 Configuration이라 되어있는 부분이 있는데 그 부분을 따라하면 된다.

간단히 얘기하면 [utterances app](https://github.com/apps/utterances)에 들어가서 install하고 동의하고 해당 코드를 `_layouts`폴더의 `post.html`에 가장 밑에 추가하거나 `_config.yml`을 수정하면 됩니다.

```HTML
<script src="https://utteranc.es/client.js"
        repo="doxawahebi/doxawahebi.github.io"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```

![Desktop View](/assets/img/posts/20250213/insert-utt-script.png){: width="700" height="300" }

<br/>

## 아바타 수정하기


## 언어 설정 좀 더 알아보기


## Markdown preview
vscode에서는 `Ctrl + K`를 누른 후 `V`키를 누르거나 밀에 사진에 있는 빨간상자 부분을 누르면
![Desktop View](/assets/img/posts/20250213/preview-button.png){: width="700" height="300" }

다음과 같이 preview가 사이드에 나온다.
![Desktop View](/assets/img/posts/20250213/preview-markdown.png){: width="700" height="300" }


## 구글 애널리틱스 설정
구글 애널리틱스로 가서 계정을 만들어줍니다.

(대충 구글 애널리틱스 관련된 이미지들)

웹 스트림 세부정보가 나타나면 `측정 ID`를 복사한 다음 `_config.yml` 파일의 `analytics.google.id`의 값으로 넣어준다.

## Reference
- https://jason9288.github.io/posts/github_blog_4/#%EA%B5%AC%EA%B8%80-%EC%95%A0%EB%84%90%EB%A6%AC%ED%8B%B1%EC%8A%A4-%EB%93%B1%EB%A1%9D

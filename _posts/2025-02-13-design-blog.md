---
title: Blog 커스터마이징 하기
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

![Desktop View](/assets/img/posts/2025/02/install-giscus-1.png){: width="700" height="300" }

![Desktop View](/assets/img/posts/2025/02/install-giscus-2.png){: width="700" height="300" }

설치한 다음에 이 [사이트](https://giscus.app/ko)에 설정 섹션 읽으면서 하시면 다음과 같이 코드가 나오는데 이걸 복사헤줍니다.

![Desktop View](/assets/img/posts/2025/02/giscus-script.png){: width="700" height="300" }

대충 적용하는 방법이 2개가 있는데 `_layouts`폴더의 `post.html`에 가장 밑에 추가하거나 `_config.yml`을 수정하는 방법이 있습니다.
취향에 맞춰하시면 되는데 개인적으로 내부적으로 존재하는 `_config.yml`에 추가하는 게 좋아서 다음과 같이 적용했습니다.

![Desktop View](/assets/img/posts/2025/02/config-giscus.png){: width="700" height="300" }

### utterances을 이용한 방법

이 [사이트](https://utteranc.es/?installation_id=60940545&setup_action=install)를 들어가보면 Configuration이라 되어있는 부분이 있는데 그 부분을 따라하면 됩니다.

간단히 오약하면 [utterances app](https://github.com/apps/utterances)에 들어가서 install하고  해당 코드를 `_layouts`폴더의 `post.html`에 가장 밑에 추가하거나 `_config.yml`을 수정하면 됩니다.

```HTML
<script src="https://utteranc.es/client.js"
        repo="doxawahebi/doxawahebi.github.io"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```

![Desktop View](/assets/img/posts/2025/02/insert-utt-script.png){: width="700" height="300" }

<br/>

## 언어 설정 좀 더 알아보기
---
언어설정와 관련된 파일은 `_data/locales`에 모두 들어 있습니다.
이 파일들이 사이트의 레이아웃 텍스트를 정하며 `_config.yml`의 lang 부분이 여기에 있는 파일들과 연결되어 있습니다.

예를 들어 lang의 값을 `en`이 아닌 `ko-KR`로 설정하면 사이트의 사이드바에 있는 텍스트들이 다음과 같이 바뀔 겁 니다.

```text
Home -> 홈
Categories -> 카테고리
Tags -> 태그
Archives -> 아카이브
About -> 정보
```

직접 다르게 수정하거나 지원하지 않는 다른 언어를 사용하고 싶으면
`_data/locales`의 `[language Name].yml` 파일을 만들어서 직접 작성하면 될 것 같습니다.


## 아바타 수정하기
---
아바타 수정은 `_config.yml`에서 그림과 같이하면 됩니다.

![Desktop View](/assets/img/posts/2025/02/config-avatar.png){: width="700" }


## 왼쪽 사이드바 배경 이미지 넣기
---
`_sass/layout/_sidebar.scss`에서 `background` 부분을 주석처리 해주고
밑에 저 3줄을 추가해준다. 그리고 url()에 자신의 배경 화면 이미지의 경로를 넣어준다.
```scss
#sidebar {
  ...

  // background: var(--sidebar-bg);
  background: url('/assets/img/sidebar/bg-orora.jpg');
  background-size: 100% 100%;
  background-position: center;
}
```

## 사이드 바에 다른 SNS 계정 추가하기
`_data/contact.yml`에 있는 부분을 수정해주면 됩니다.

만약 페이스북 계정을 추가하고 싶다면 다음과 같이 적습니다..

```yml
- type: facebook
  icon: 'fab fa-facebook'
  url: 'https://www.facebook.com/'
```

`fab fa-facebook`는 무료 오픈 소스인 [fontawesome](https://fontawesome.com/)에 있는 폰트입니다.

## 블로그 타이틀과 서브타이틀 폰트/색상 바꾸기
---
사이드 바에 있는 블로그 이름과 설명이 타이틀과 서브타이틀입니다.
`_sass/layout/_sidebar.scss`에서 수정할 수 있습니다.

```scss
  .site-title {
    @extend %clickable-transition;
    @extend %sidebar-link-hover;

    font-family: inherit;
    font-weight: 900;
    font-size: 1.75rem;
    line-height: 1.2;
    letter-spacing: 0.25px;
    margin-top: 1.25rem;
    margin-bottom: 0.5rem;
    width: fit-content;
    color: var(--site-title-color);
  }

  .site-subtitle {
    font-size: 95%;
    color: var(--site-subtitle-color);
    margin-top: 0.25rem;
    word-spacing: 1px;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
  }
```


## footer 수정하기
---
밑에 copyright 어쩌고 써져있는 부분이 footer인데
`_includes/footer.html`에서 수정해주시면 됩니다.

## About 수정하기
---
`_tabs` 폴더에 about.md를 수정해주면 됩니다.
아이콘 같은 것도 수정할 수 있습니다.

CATEGORIES, TAGS, ARCHIVES도 전부 여기서 수정해주면 됩니다.

## favicon
---
favicon은 인터넷 웹 브라우저의 주소창에 표시되는 웹사이트나 웹페이지를 대표하는 아이콘입니다.
다음과 같은 작은 이미지가 favicon입니다.
![Desktop View](/assets/img/posts/2025/02/explain-favicon.png){: width="700" height="300" }

이 [favicon 만들어주는 사이트](https://www.favicon-generator.org/) 들어가서 이미지 업로드 하면 이미지들을 주는데
`asset/img/favicons` 들어가서 이미지들 대체 해주면 됩니다..


## 구글 애널리틱스 설정
---
구글 애널리틱스로 가서 계정을 만들어줍니다. 그림처럼 만들어주면 됩니다.
![Desktop View](/assets/img/posts/2025/02/sign-up-ga-1.png){: width="700" height="300" }
![Desktop View](/assets/img/posts/2025/02/sign-up-ga-2.png){: width="700" height="300" }
![Desktop View](/assets/img/posts/2025/02/sign-up-ga-3.png){: width="700" height="300" }

웹 스트림 설정을 해줍니다.

![Desktop View](/assets/img/posts/2025/02/config-web-stream.png){: width="700" height="300" }

웹 스트림 세부정보를 보면 `측정 ID`라는 게 존재하면 이것을 복사한 다음 `_config.yml` 파일의 `analytics.google.id`의 값으로 넣어주면 됩니다.
![Desktop View](/assets/img/posts/2025/02/ga-stream-details.png){: width="700" height="300" }
![Desktop View](/assets/img/posts/2025/02/config-ga.png){: width="700" height="300" }

## 사이트 맵 설정
---


## 마무리
---
간단하게 여러가지 알아봤는데요.
만약 블로그를 좀 더 꾸미고 싶으면 `_`로 시작하는 디렉토리를 좀 더 탐색해보면서 이것저것 시도해보시시면 될 것 같습니다.

## Reference
---
- https://jason9288.github.io/posts/github_blog_4/#%EA%B5%AC%EA%B8%80-%EC%95%A0%EB%84%90%EB%A6%AC%ED%8B%B1%EC%8A%A4-%EB%93%B1%EB%A1%9D
- https://www.irgroup.org/posts/Chirpy-%ED%85%8C%EB%A7%88-%EC%BB%A4%EC%8A%A4%ED%84%B0%EB%A7%88%EC%9D%B4%EC%A7%95/

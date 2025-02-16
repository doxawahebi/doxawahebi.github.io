---
title: post 작성하기
description: 
author: doxawahebi
date: 2025-02-12 21:26:00 +0900
categories: [Jekyll]
tags: [github-pages, jekyll, chirpy-theme]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
---

저번글에서는 아주 간단하게 글 쓰는 방법을 알아봤다. (파일 이름명과 경로, frontmatter())



## frontmatter 좀 더 자세히 알아보기

```yaml
title: Text and Typography
description: Examples of text, typography, math equations, diagrams, flowcharts, pictures, videos, and more. (부제목목)
author: cotes
date: 2019-08-08 11:33:00 +0800
categories: [Blogging, Demo]
tags: [typography]
pin: true
math: true
mermaid: true
image:
  path: /commons/devices-mockup.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
```

### Author Information
기본적으로 작가에 대한 정보는 `social.name`과 `social.links`의 첫번째 항목에서 가져온다.
혹시 이것을 따로 오버라이드하고 싶으면 `_data/authors.yml`에 작가에 대한 정보를 추가해준다.

```YAML
<author_id>:
  name: <full name>
  twitter: <twitter_of_author>
  url: <homepage_of_author>
```
{: file="_data/authors.yml" }

그 후 작성할 포스트의 frontmatter에 다음과 같이 작가정보를 추가헤준다.
```yaml
---
author: <author_id>                     # 저자가 한 명인 경우
# or
authors: [<author1_id>, <author2_id>]   # 저자가 복수인 경우
---
```
참고로 공식 문서에 따르면 `_data/authors.yml`의 작가 정보를 읽는 것의 이점은 페이지가 `twitter:creator` 메타 태그를 가지게 되서
[Twitter Cards](https://developer.x.com/en/docs/x-for-websites/cards/guides/getting-started#card-and-content-attribution)를 풍부하게 하고 SEO에 좋다고 한다.

### Description
```yaml
---
description: 포스트에 대한 짧은 요약
---
```
다음 사진의 회색 글자가 description이다. 글이 어떤 내용인지 간단하게 보이고 싶으면 작성하면 된다.
![Desktop View](/assets/img/posts/2025/02/post-description.png){: width="700" }

### Table of Contents
목차는 오른쪽 패널에 기본적으로 항상 보이며
전역적으로 보이지 않게 하고 싶으면  `_config.yml`을 `toc` `false`로 수정하면 되고
특정 게시물만 안 보이게 할 거면 Front Matter에 다음과 같이 작성하면 된다.
```yaml
---
toc: false
---
```

### Comments
특정 포스트에 대해 comments가 안 보이게 할려면 다음과 같이 작성한다.
```yaml
---
comments: false
---
```
Comments 활성화에 관한 내용을 보고 싶으면 다음 [글](https://doxawahebi.github.io/posts/design-blog/#%EB%8C%93%EA%B8%80-%EC%B6%94%EA%B0%80%ED%95%98%EA%B8%B0)에서 참고하면 된다.


### Prompts
```markdown
{: .prompt-info }
```

## Markdown preview
---
vscode에서 마크다운을 작성을 하시는 분은 `Ctrl + K`를 누른 후 `V`키를 누르거나 밀에 사진에 있는 빨간상자 부분을 누르면
![Desktop View](/assets/img/posts/2025/02/preview-button.png){: width="700" height="300" }

다음과 같이 preview가 사이드에 나오니까. 참고해두면 좋을 것 같습니다.
![Desktop View](/assets/img/posts/2025/02/preview-markdown.png){: width="700" height="300" }

## commit error
요구하는 커밋 규칙을 지키지 않으면 다음과 같은 오류 발생한다.
**"subject may not be empty [subject-empty]" and "type may not be empty [type-empty]"**

이렇게 작성하거나
```shell
git commit -m "fix(core): add hasValue narrowing to ResourceRef (#59708)"
```

귀찮고 잘 모르겠으면 이렇게 작성하면 된다. (`--no-verify`한다는 이야기이다.)
```shell
git commit -n -m ""
```

잘 모르겠으면 필자가 작성한 [글](https://doxawahebi.github.io/posts/commit-message-style/)을 참고하자.



## Reference
---
- https://chirpy.cotes.page/posts/write-a-new-post/

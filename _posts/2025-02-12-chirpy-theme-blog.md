---
title: Github Pages로 blog 호스트하기
description: using chirpy-theme
author: doxawahebi
date: 2025-02-12 21:26:00 +0900
categories: [Jekyll]
tags: [github-pages, jekyll, chirpy-theme]     # TAG names should always be lowercase
pin: true
math: false
mermaid: false
# image:
  # path: /20250212/repo_name.png
  # lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  # alt: image
---



## jekyll 설치
---
jekyll를 설치하는 이유는 로컬에서 jekyll을 실행시켜서 포스팅 결과를 미리 보기 위해서입니다.

(Windows환경에서서 설명했습니다.)

### 1. 루비 설치하기

해당 [사이트](https://rubyinstaller.org/)에서 루비를 설치해줍니다.

그 다음 cmd을 실행시키고
다음 명령어를 통해 설치여부를 확인합니다.
```shell
ruby -v
gem -v
```

### 2. jekyll과 bundler를 설치하기

해당 명령어를 입력해줍니다.
```shell
gem install jekyll bundler
bundle install
bundle add webrick
```
bundle install은 젬파일에 특정 의존성들을 설치해주는 command입니다.
webrick 관련 오류가 뜨면 webrick도 추가해준다.

## 테마 설치하기
---
크게 두 가지 방법이 있다.

### 1. starter 사용하는 방법
특별히 수정없이 글쓰기에만 집중할 사람이면 공식에서도 추천하는
가장 쉬운 방법인 [starter](https://github.com/cotes2020/chirpy-starter)를 이용하도록 하자.

### 2. 테마를 포킹하거나 zip로 받아서 하는 방법
UI design을 수정하거나 Jekyll에 익숙하고
이 테마를 헤비하게 수정해서 사용할 예정이라면 두 번째 방법을을 추천한다.


필자는 zip로 받아서 하는 방법으로 했다.


## github repository 생성
---
github repository를 생성한다. 이때 repository name은 (github user name).github.io로 한다.

![Desktop View](/assets/img/posts/2025/02/repo_name.png){: width="700" height="300" }
필자는 이미 레포지토리가 존재해서 빨간색으로 뜨는 것일 뿐이다.

로컬 저장소와 원격 저장소를 연결한다.
```shell
git clone https://github.com/(github user name)/(github user name).github.io.git
```

다음 [사이트](https://github.com/cotes2020/jekyll-theme-chirpy/releases)로 가서 소스코드를 다운받고 로컬 저장소에 저장한다.

다음 명령으로 init.sh를 실행시킨다.
```shell
bash tools/init.sh
```
 <!-- 어떤 파일들이 삭제되는지... -->
아마 fork해서 레포지토리를 만든 게 아니면 해당 파일이 실행되지 않을 것 입니다.
파일을 수정해줘야 되는데 RELEASE_HASH나 필요없는 함수나 필요없는 git 명령어를 지워주면 됩니다..

<details>
<summary>수동으로 하실 분들 참고사항</summary>
<div markdown="1">

init.sh를 실행하지 않으실 분은 해당 [블로그](https://velog.io/@hashnsalt/Github-Blog-%EB%A7%8C%EB%93%A4%EA%B8%B0-2)를 참고해보는 것도 좋을 것 같습니다.

workflow 직접 생성하실분은
jekyll.yml에 [해당 이슈](https://github.com/cotes2020/jekyll-theme-chirpy/discussions/1809)가 발생할 수 있다.

이 이슈는 Build에서 Build with Jekyll 위에 다음 텍스트를 추가하면 해결된다.
```text
name: npm build
run: npm install && npm run build
```

</div>
</details>
<br/>

init.sh가 성공적으로 실행되었다면
Jekyll가 로컬 서버에서 실행되는 지 확인해봅니다.
```shell
bundle exec jekyll serve
```
무언가가 다 잘 지워진거 같으면 성공한 겁니다.

## _config.yml 수정
---
블로그의 기본 설정와 관련된 파일입니다.

- lang = 웹 페이지의 언어
- timezone = 시간대 = Asia/Seoul
- title = 메인 타이틀
- tagline = 블로그의 서브타이틀
- description = SEO Meta 및 Atom Feed에서 사용됨
- url = 블로그 주소 = https://doxawahebi.github.io
- github.username: = GitHub username
- social.name = 포스트의 기본 작성자로 보여지며 footer에 copyright 소유자로 보여짐
- social.email = 이메일 계정
- social.links = 다른 SNS 계정들들
- analytics = 웹 분석 세팅
- theme_mode = 테마 스킨에 관한 것 기본은 light이다.
- avatar = 블로그 왼쪽 상단에 보여줄 이미지 = /asset/img/example.png


## 포스팅 규칙
---
포스팅 작성에 관한 내용은 [해당 글](https://chirpy.cotes.page/posts/write-a-new-post/)에 잘 나와있습니다.

### 1. 파일명과 경로
The `_posts` 폴더는 블로그의 포스트들이 위치하는 곳입니다. 포스트들은 모두 이 폴더에 위치해있어야합니다.
포스트를 작성할 때는 **마크다운**이나 **HTML**로 작성합니다.

새로운 파일을 작성할 때는 파일명이 `YYYY-MM-DD-TITLE.EXTENSION`와 같은 형식으로 이름으로 지어야합니다.
`EXTENSION`은 **md**이나 **markdown**이어야합니다.
파일명에 공백이 들어가서는 안 되며 공백 대신 하이픈(`-`)을 사용해야합니다다.

### 2. Frontmatter
모든 블로그 포스트들은 layout이나 다른 meta date를 설정하기 위해서는 파일 맨 위에 [front matter](https://jekyllrb.com/docs/front-matter/)를 작성해야합니다.
아래 같은 형식으로 작성해주면 되는데 이에 대해서는 다음 시간에 좀 더 알아봅사더,
```yaml
---
title: TITLE
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [TOP_CATEGORIE, SUB_CATEGORIE]
tags: [TAG]     # TAG 이름은 항상 소문자이여야합니다.
---
```
<br/>
포스트 파일 작성법이 귀찮으면 [jekyll-compose](https://github.com/jekyll/jekyll-compose) 플러그인을 사용해보는 것도 좋습니다.


## 원격 저장소에 소스 올리고 배포하기
---
```shell
git add . &&
git commit -m "message about update" &&
git push
```

깃허브에 들어가서 배포가 성공적으로 되었다면 깃허브 블로그(https://(username).github.io/)에 들어가서 확인해봅니다.

## 마무리
---
간단하게 블로그를 만드는 방법을 살펴보았습니다.
참고로 살펴본 내용들은 대부분 다른 jekyll theme에 적용되는 내용들입니다.
더 많은 내용이 궁금하면 [jekyll 공식 docs](https://jekyllrb.com/docs/themes/)를 읽어보는 것도 좋을 것 같습니다.

## Reference
---
- https://chirpy.cotes.page/posts/getting-started/
- https://chirpy.cotes.page/posts/write-a-new-post/
- https://www.irgroup.org/posts/jekyll-chirpy/
- https://kiffblog.tistory.com/233

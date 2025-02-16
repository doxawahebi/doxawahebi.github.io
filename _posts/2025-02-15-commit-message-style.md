---
title: git commit message 작성 규칙
description: 
author: doxawahebi
date: 2025-02-15 14:30:00 +0900
categories: [git]
tags: [git, commit, commit-message]     # TAG names should always be lowercase
pin: false
math: false
mermaid: false
---

## Overview
---
git commit message을 작성하는 규칙에 대해서는 다양한 종류가 있는데 많은 사람들이 일반적으로 어떠한 방식으로 작성하는지 알아보자.

## Message Structure
---
커밋 메세지는 공백으로 된 한 줄을 기준으로 3가지 영역으로 구성되어 있습니다. 
header 부분은 무조건 작성해줘야합니다.
메제지의 각 줄은 100자를 넘어서는 안 됩니다.
- header
- optional body
- optional footer
header은 message의 type과 scope, subject로 구성되어 있다.

다음과 같은 형식으로 이루어져있다는 이야기입니다.
```php
<type>(<scope>): <subject>

<body>

<footer>
```

### Header
---
#### Type

변경의 type을 기술하며 일반적으로 다음과 같은 것들이 존재하고 그 중 하나이여야합니다.
- feat: 새로운 기능
- fix: 버그 픽스
- docs: Documentation만 변경
- style: 코드의 동작에 영향을 주지 않는 변화 (공백, 포맷팅, 세미콜론, etc).
- refactor: 기능 추가도 버그 수정도 아닌 코드 수정
- perf: 퍼포먼스 향상을 위한 코드 수정
- test: 누락된 테스트 추가
- chore: 빌드 프로세스나 패키지 매니저 설정, 보조 도구 등등 수정. 생산 코드는 변경 없음.

#### Scope
scope는 옵션이며 커밋으로 변경된 장소를 가리킨다. (`mac`, `ui`, `auth`, etc.)

#### Subect
subject는 변경 사항에 대한 짧은 설명을 기술한다.
작성할 때 다음 규칙을 지켜주세요.

- 명령형, 현재 시제 어조로 작성하기 (changed나 changes가 아닌 change를 사용하기)
- 첫글자 대문자로 시작하지 않기
- 마지막 (.) 붙이지 않기



### Body
마찬가지로 명령문, 현재 시제로 작성하고 변경한 이유와 변경 전과의 차이점을 설명합니다.

### Footer
footer는 **Breaking Changes**(주요 변경 사항)과 커밋과 관련된 Github 이슈를 적습니다.


**Breaking Changes** 섹션을 적을 때는 `BREAKING CHANGE:`로 시작하며 주요 변경 사항에 대한 자세한 설명과 마이그레이션 지시, 빈 줄을 다음과 같은 형식으로 적습니다. 그리고 관련된 `<issue number>`도 적어줍니다.

```
BREAKING CHANGE: <breaking change summary>
<BLANK LINE>
<breaking change description + migration instructions>
<BLANK LINE>
<BLANK LINE>
Fixes #<issue number>
```

프로젝트에 따라서 `BREAKING CHANGE:`이외에도 `Deprecation `와 같이 다른 섹션을 정한 경우도 존재한다.



다음과 같은 형식으로 [커밋](https://github.com/angular/angular/commit/6c92d653493404a5f13aa59cde390bcbed973fb6)을 작성하면 된다.

```
fix(core): add hasValue narrowing to ResourceRef (#59708)

`hasValue` attempts to narrow the type of a resource to exclude `undefined`.
Because of the way signal types merge in TS, this only works if the type
of the resource is the same general type as `hasValue` asserts.

For example, if `res` is `WritableResource<string|undefined>` then
`.hasValue()` correctly asserts that `res` is `WritableResource<string>` and
`.value()` will be narrowed. If `res` is `ResourceRef<string|undefined>`
then that narrowing does _not_ work correctly, since `.hasValue()` will
assert `res` is `WritableResource<string>` and TS will combine that for a
final type of `ResourceRef<string|undefined> & WritableResource<string>`.
The final type of `.value()` then will not narrow.

This commit fixes the above problem by adding a `.hasValue()` override to
`ResourceRef` which asserts the resource is of type `ResourceRef`.

Fixes #59707

PR Close #59708
```


대충 head만 작성할거면 이렇게 작성하면 된다.
```shell
git commit -m "fix(core): add hasValue narrowing to ResourceRef (#59708)"
```



## 마무리
---
본문과는 차이점이 꽤꽤 있는데 Udacity에서 제시한 [가이드](https://udacity.github.io/git-styleguide/)도 한 번 읽어보시는 것도 나쁘지 않습니다.

## Reference
---
- https://kcong0505.tistory.com/54
- https://udacity.github.io/git-styleguide/
- https://gist.github.com/develar/273e2eb938792cf5f86451fbac2bcd51
- https://github.com/angular/angular/blob/main/CONTRIBUTING.md#commit-message-header
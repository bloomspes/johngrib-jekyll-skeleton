# Vimwiki + Jekyll + Github.io

# 책 읽는 프로그래머의 위키

이 블로그는 [기계인간 이종립]()님의 블로그 스켈레톤으로 만들어졌습니다.

## 설치하기

루비가 설치되어 있지 않을 경우 루비를 설치해 주세요. 여기에서는 rvm으로 설치하는 방법을 소개해 드립니다. 다른 방법으로도 루비를 설치할 수 있으니, 다른 방법으로 하셔도 됩니다.  

루비 버전은 GitHub Pages Dependency versions을 보면 GitHub Pages에서는 2.7.4버전을 사용하고 있으니 해당 버전을 설치해 줍니다.

```
# See also https://rvm.io/rvm/install
$ gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
$ curl -sSL https://get.rvm.io | bash
$ rvm install 2.7.4
$ rvm use 2.7.4
```

bundle install을 실행하여 의존성들을 설치합니다.

```
$ bundle install
```

### Git hooks 추가하기

새로운 글을 등록하면 메타 데이터를 업데이트해 주어야 합니다. 커밋하기 전에 이를 자동으로 될 수 있도록 Git Hooks를 추가해야 합니다.  

```
$ cp tool/pre-commit ./.git/hooks
```

### 노드 모듈 설치하기

메타데이터 생성을 위해서 generateData.js를 실행해야 합니다. 이를 실행하기 위해서 yamljs 의존성을 설치해야 합니다.

```
$ npm install
```

## 실행하기

```
$ jekyll serve
```

## 글 작성하기

### 새로운 카테고리 만들기

카테고리가 있는 글을 작성하고 싶을 때는 카테고리를 먼저 만들어야 합니다. /_wiki/category-name.md같이 파일을 만들고 내용에는 다음을 추가해야 합니다.  

이때 layout속성은 category가 되어야 합니다.  

```
---
layout  : category
title   : 제목을 입력합니다.
summary : 
date    : 2022-10-06 00:00:00 +0900
updated : 2022-10-06 00:00:00 +0900
tag     : 
toc     : true
public  : true
parent  : index
latex   : false
---

* TOC
{:toc}
```

### 위키에 글 등록하기

위키를 작성할 때는 `/_wiki` 폴더 아래에 마크다운으로 파일을 작성합니다. 만약 카테고리 아래에 글을 작성하고 싶을 경우에는 카테고리 이름으로 폴더를 만들고 파일을 추가합니다. 예를 들어 `/_wiki/category-name/document.md`로 만들 수 있습니다. `layout`은 `wiki`가 되어야 합니다. `parent`는 상위 카테고리 이름을 작성해야 합니다.  

만약 상위 카테고리가 없을 경우에는 `parent`에 `index`를 입력합니다.

```
---
layout  : wiki
title   : 제목을 적습니다
date    : 2022-10-08 11:23:00 +0900
updated : 2022-10-08 11:23:00 +0900
tag     : 
toc     : true
public  : true
parent  : category-name
latex   : false
---

* TOC
{:toc}

내용을 적습니다.
```
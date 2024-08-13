---
title: git 명령어 정리 (내가 보려고)
date: 2024-08-07 06:30:00 +0900
categories: [git, commands]
tags: [git]
---

# git 커맨드 종류별 모음



## repo 가져오기

Git repository를 클론해온다.

```shell
git clone https://github.com/[계정]/[repo-name].git
```

git remote, push 등 원격으로 여러 동작을 수행하기 위해서는 ACCESS TOKEN이 필요하다. 이 경우, ACCESS TOKEN을 통해서 clone도 가능하다.

```shell
git clone https://[ACCESS_TOKEN]@github.com/[계정]/[repo-name].git
```



## 원격 연결하기

### 계정 등록

기본적으로 원격으로 연결해서 `commit`하고 `push`하려면 계정 등록하고, 본인의 token을 넣어야 한다.

```shell
git config --global user.name [계정]
```

### 원격 연결

`origin`은 일반적인 원격 연결 저장소의 이름으로 사용한다. 굳이 origin이 아니어도 된다.

```shell
git remote add origin https://github.com/[계정]/[repo-name]
```

이 경우에도 ACCESS TOKEN을 통해서 연결할 수 있다.

```shell
git remote add origin https://[ACCESS_TOKEN]@github.com/[계정]/[repo-name]
```

### 원격 연결 삭제

```shell
git remote remove origin
```



## 파일 업로드

### 파일 추가

```shell
git add .
git add -A
git add -p #patch
```

### 인덱스 파일 삭제

```shell
git restore --staged [파일명]
```

### 커밋하기

나는 commit 방식이 첫 번째가 편하다. 첫 번째 방식대로 commit하면 `vim editor`가 열리고, 첫 줄은 `commit title`이고 한 줄 띄고 그 이하는 `commit content`으로 자동인식한다.

```shell
git commit
```

```shell
git commit -m "[커밋 내용]"
```

### 업로드

```shell
git push -u origin [브랜치 이름]
```



## 브랜치

### NEW Branch

`이름`은 새롭게 만들 branch의 이름이고, 분기지점은 분기할 branch의 이름이다.

```shell
git branch [이름] [분기지점]
```

### DELETE Branch

```shell
git branch -D [이름]
```

### 이름 변경

```shell
git branch -m [기존 이름] [새 이름]
```

### Branch 이동

```shell
git checkout [이름]
```

### Branch 상태

```shell
git branch -v
```

```shell
git branch --merged
```

```shell
git branch --no-merged
```


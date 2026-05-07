---
title: Git에서 submodule 추가/꼬임 해결
date: 2026-05-03 11:00:00 +0900
categories: [git, command]
tags: [git, command, submodule]
---

# Git에서 submodule 추가/꼬임 해결

Git에서 submodule로 프로젝트를 관리하다 보면, 꼬임 현상이 발생하기도 한다. 따라서, 해당 꼬임 현상을 해결해야 하는 경우가 많다.
이 경우에는 submodule의 각 repo의 commit hash가 꼬여서 발생하는 경우가 많다.
이 문제를 해결하거나(1부터 정독), submodule로 프로젝트 관리를 시작하고자 하는 사람들이 submodule을 적절히 추가할 수 있도록 그 방법을 소개하고자 한다.

## 1. 기존 캐시 삭제

먼저, 우리는 `googletest` module로 test를 한다고 가정하자.

기존 캐시부터 삭제해주자.

```bash
git submodule deinit -f --all
git rm -f --cached <YourPath>/googletest
rm -rf third_party/googletest
rm -rf .git/modules/third_party/googletest
rm -f .gitmodules
```

## 2. 새로운 서브모듈 작성

새롭게 서브모듈(`.gitmodules`) 파일을 작성해주자.

```
[submodule "<YourPath>/googletest"]
	path = <YourPath>/googletest
	url = https://github.com/google/googletest.git
```

## 3. 서브모듈 추가

아래를 입력하면, 새로운 새로운 모듈이 추가된다.

```bash
git submodule add https://github.com/google/googletest.git <YourPath>/googletest
```

그리고, 선택적으로 commit hash를 고정시킬 수 있다.
아래의 명령어를 실행하면 된다.

```bash
git -C <YourPath>/googletest checkout b514bdc898e2951020cbdca1304b75f5950d1f59
```

## 4. 서브모듈 저장

commit 및 git push 함으로써 서브모듈을 저장할 수 있다.

```bash
git add .gitmodules
git add <YourPath>/googletest

git commit -m "Add third-party submodules"
git push
```


## 5. 테스트

아래를 실행하면, 그 다음과 같은 결과가 나오면, 잘 추가된 것이다.

```sh
git ls-files --stage | grep 160000
```

```
160000 b514bdc898e2951020cbdca1304b75f5950d1f59 0	<YourPath>/googletest
```


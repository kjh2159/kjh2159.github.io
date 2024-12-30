---
title: 협업을 위한 github 명령어 모음 (w/ Termux)
date: 2024-12-26 00:00:00 +0900
categories: [github, git]
tags: [git, commands, cowork]
---

## 0. Access Token

Access Token은 github 협업에 있어 필수적으로 발급 받아야 한다. `Account Settings > Developer Settings > Personal Access Token > Token`에서 발급받을 수 있다. 이 Access Token은 한 번 발급 받으면, 다신 볼 수 없기에 어디 잘 적어 놓아야 한다.

## 1. 초기 설정

계정 등록

```bash
git config --global user.name <id> # git config --global user.name kjh2159
```

```bash
git clone <git-repository-address> # git clone https://github.com/kjh2159/llama.cpp.git
or
git clone https://<access-token>@github.com/<id>/<repository-name>.git 
```
위의 명령어는 둘 중 하나만 사용하면 된다. access token을 입력하지 않는 git clone 명령어는 나중에 pull, push 시에 username, password를 입력해야 한다. 이 때 username에는 본인의 계정 아이디, password에는 access token을 입력하면 된다.


## 2. Branch

### 브랜치 목록 확인
```bash
git branch
```
> -a 옵션: 로컬 및 원격 브랜치 모두 표시 \
> -r 옵션: 원격 브랜치만 표시
> ex) git branch -r
{: .prompt-info}

### 새로운 브랜치 생성
브랜치 이름은 보통 feature 이름으로 짓는다. checkout은 브랜치 생성 및 해당 브랜치 이동을 담당한다. 브랜치가 없으면 생성하고, 있다면 이동만 한다.
```bash
git checkout -b <branch-name> 
```

### 특정 포인트에서 브랜치 생성
새로운 브랜치를 특정 포인트에서 생성한다.
```bash
git branch <branch-name> <commit-hash>
```

### 브랜치 삭제
기본적으로 branch deletion을 하기 위해서는 merge가 되어야 한다.
```bash
git branch -d <branch-name>
```
> -D 옵션: merge된 branch가 아니어도 강제 deletion한다.
{: .prompt-info}


### 브랜치 이름 변경
현재 checkout되어 있는 브랜치의 이름을 변경한다.
```bash
git branch -m <new-branch-name>
```

## 3. Commit

Commit 관련해서는 [기본 명령어 모음 주소](https://kjh2159.github.io/posts/git-commands/) 이곳 참고하기.

## 4. PUSH

### Push 기본형
`remote-name`은 원격 브랜치 이름을, `branch-name`은 분기점 이름을 나타낸다. 
```bash
git push [-u] <remote-name> <branch-name>
```
> -u 옵션 : 로컬 브랜치와 원격 브랜치를 동기화시킨다. \
> 새 브랜치를 파고 이 옵션을 안 넣으면, 로컬과 원격 브랜치 간의 연동이 되지 않아 push할 때 항상 브랜치를 지정해주어야 함. 
{: .prompt-info}

### 삭제 Push
원격 브랜치나 태그를 삭제한다. (태그 push의 경우 딱히 안 쓸 것 같아서 넣지는 않음.) `branch-name/tag`에는 브랜치 이름이나 태그를 넣으면 된다.
```bash
git push <remote-name> --delete <branch-name/tag>
```

### 특정 Commit Push
로컬의 특정 Commit을 원격 저장소의 지정된 브랜치로 업로드할 수 있다.
```bash
git push <remote> <commit-hash>:<branch-name>
```
> `commit-hash`는 local의 commit hash 값이며, `branch-name`은 원격 저장소에 존재하는 원격 브랜치임을 주의하자.
{: .prompt-warning}

### Preview Push
Push 결과의 Preview를 볼 수 있다. 두 번째 명령어처럼 각 브랜치를 지정할 수도 있다.
```bash
git push --dry-run
git push --dry-run <remote-name> <branch-name>
```


## 4. PULL

### Pull 기본형
지정한 원격 저장소와 브랜치에서 최신 변경 사항을 가져오고 로컬 브랜치에 병합한다.
```bash
git pull <remote-name> <branch-name>
```

### Rebase
변경 사항을 병합하지 않고, 로컬 브랜치의 커밋을 원벽 변경 사항 위로 재정렬(rebase)한다. 병합 commit 없이 히스토리를 유지하는 데에 유용하다.
```bash
git pull --rebase <remote-name> <branch-name>
```

### Preview Pull
실제로 변경 사항을 가져오지 않고, 어떤 변경 사항이 적용될지 미리 확인할 수 있다.
```bash
git pull --dry-run <remote-name> <branch-name>
```

### 특정 파일만 Pull
git pull 자체는 commit된 branch를 기준으로 일어나므로 파일 단위 업데이트가 불가능하다. 따라서 `git fetch`와 `git checkout`으로 특정 파일만 업데이트 할 수 있다.
```bash
git fetch <remote-name> <branch-name> && git checkout FETCH_HEAD -- <files>
```
> ex) `git fetch origin main && git checkout FETCH_HEAD -- file1.txt file2.txt`
> FETCH_HEAD 옵션 : `git fetch` 명령어 실행 후 git이 원격에서 가져온 데이터를 임시로 저장하는 특수 참조이다.
{: .prompt-info}


### 충돌 시 병합 도구
로컬 변경 사항을 임시 저장(stash)하고 충돌 시 병합 도구를 사용할 수 있도록 준비한다.
```
git pull --rebase --autostash <remote-name> <branch-name>
```

## 5. MERGE
### Merge 기본형
현재 checkout된 branch에 `branch-name`의 변경 사항을 병합한다.
```bash
git merge <branch-name>
```

### 주요 옵션
```bash
git merge <branch-name> [-m "<message>"] [--no-commit]
```
> -m 옵션 : merge의 메세지를 지정
> --no-commit 옵션 : 병합 충돌시 commit을 수행하지 않음


## 6. Pull Request
가장 좋은 방법은 visual studio code를 사용할 수 있으면, `GitLens`라는 Extension을 설치하여 commit 이후에 `Create a Pull Request`하는 거라고 생각한다. 하지만, Termux처럼 힘든 경우에는 아래를 따라서 할 수도 있다.

### Github CLI 설치
[Github CLI 다운로드 사이트](https://cli.github.com/)에서 Github CLI를 설치해준다.

### Github CLI 인증
아래의 명령어를 치면, 로그인을 해야 하는데 Termux에서 할 경우 어차피 브라우저는 못 쓴다. 그냥 `Paste an authentication token`으로 Personal token 붙여넣어서 인증하는 게 가장 좋아 보인다. 
```bash
gh auth login
```

### Pull Request
Pull request하기 전에는 처음 pull request 하는 경우에는 pull request가 되어있는 repository를 아무거나 보고 이해한 후에 진행하면 도움이 될 것 같다.
```bash
gh pr create -B <target-branch> -H <source-branch> -t <title> -b <body-content> -r <reviewers> -a <assignees> 
```
> -B / --base 옵션 : Pull Request의 대상 브랜치를 지정한다. 기본적으로는 원격 repository의 main 브랜치가 사용된다. \
> -H / --head 옵션 : Pull Request의 소스 브랜치를 지정한다. 기본적으로 현재 checkout된 브랜치가 사용된다. \ 
> -t / --title 옵션 : Pull Request의 제목을 지정한다. 
> -b / --body 옵션 : Pull Request의 본문을 지정한다.
> -r / --reviewer 옵션 : 특정 계정을 reviewer로 지정한다. (이건 옵션이다. 없어도 된다. 여러명 지정해야 할 경우 comma로 구분한다.)
> -a / --assignee 옵션 : 특정 계정을 Pull Request의 담당자로 지정한다. (이건 옵션이다.)
{: .prompt-info}


## 7. 여러 에러 상황 해결

### git pull 충돌
아래의 명령어로 일단, 현재 본인의 로컬 브랜치의 commit 상태와 remote 브랜치의 commit 상태가 어떻게 다른지 확인한다. rebase를 사용하면 본인의 commit이 원격의 remote 브랜치의 최신 commit으로 되기 때문에 신중히 하자.
```bash
git diff <remote-name>/<branch-name>
git rebase <remote-name> <branch-name>
```

별 차이가 없다면, 아래를 수행해준다. `reset --hard`의 경우 로컬의 변경사항이 원격 저장소의 브랜치와 다르면 전부 삭제해버리므로 조심히 하자. 
```bash
git fetch --all
git reset --hard <remote-name>/<branch-name>
git pull <remote-name> <branch-name>
```
> Ref: [참고 사이트](https://frontdev.tistory.com/entry/GIT-Conflict%EC%B6%A9%EB%8F%8C-%EB%82%AC%EC%9D%84-%EB%95%8C-%EA%B0%95%EC%A0%9C%EB%A1%9C-Pull-%ED%95%98%EA%B8%B0)
{: .prompt-info}


### git push 충돌
git push가 에러나는 경우는 최신 commit이 아닌 commit에 push를 하려고 할 때 발생한다. 따라서 로컬 commit을 최신 원격 repository의 commit과 sync가 맞도록 해줘야 한다. 방법은 간단하다. `git pull` 해주면 된다.
```bash
git pull ...
```
...에 들어갈 것들은 옵션이며 옵션의 경우 위를 참고하자.



### git-error RPC-400
이 에러는 업로드 할 때 사용하는 Buffer가 넘쳐서 발생한다. Buffer의 크기를 최대로 늘려주면 된다.
```bash
git config --global http.postBuffer 524288000
```
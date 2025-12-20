---
title: Linux terminal(bash)에 git branch 띄우기
date: 2025-12-20 10:00:00 +0900
categories: [Linux, Terminal]
tags: [Linux, Terminal, Bash, Git, Prompt]
---

# Linux terminal(bash)에 git branch 띄우기

## 1. Terminal의 작동 원리

이 포스트에서는 bash terminal에 대해서만 다루겠지만, 대부분의 Unix 계열 terminal은 매우 유사한 구조로 동작한다.
일반적으로 우리가 사용하는 터미널 환경은 **터미널 에뮬레이터(예: GNOME Terminal)** 와  
**셸(shell, 예: bash)** 이 분리된 구조로 구성되어 있다.

리눅스 환경에서 터미널 창 자체는 `gnome-shell` 및 터미널 에뮬레이터 프로세스에 의해 관리된다.
사용자가 입력한 명령어는 해당 터미널을 통해 **셸 프로세스(bash)** 로 전달되고,
bash는 이를 해석하여 적절한 동작을 수행한다.

bash는 단순한 명령 실행기 이상으로, 자체적인 **명령어 분석 규칙**, **환경 변수 관리**, **파이프 및 리다이렉션 처리**,
그리고 **프로세스 생성(fork/exec)** 을 담당하는 핵심 컴포넌트이다.
즉, 우리가 터미널에서 입력하는 모든 명령어는 bash가 정의한 문법과 실행 규칙에 따라 해석되고 처리된다.

이러한 명령어 체계와 실행 환경은 사용자가 직접 커스터마이징할 수 있다.
대표적으로 bash는 실행 시점에 여러 초기화 스크립트를 로드하며, 그중 가장 널리 사용되는 것이 `~/.bashrc` 파일이다.

`bashrc`를 통해 우리는
- alias를 이용한 명령어 단축
- PATH 및 각종 환경 변수 설정
- 프롬프트(`PS1`) 커스터마이징
- 자동 완성, 히스토리 옵션 등 셸 동작 방식 정의
와 같은 설정을 구성할 수 있다.

결과적으로 터미널은 단순한 입출력 창에 불과하며,
실제로 사용자가 체감하는 “터미널 사용 경험”의 대부분은 bash가 제공하는 명령어 해석 규칙과 `bashrc`를 통해 정의된 사용자 환경에 의해 결정된다.
즉, `bashrc` 설정을 통해 터미널이 보여주는 정보를 지정할 수 있다.

## 2. 변수 및 함수 설정

이제 `bashrc`에 진입해야한다.
아래의 명령어를 통해 `bashrc`에 진입할 수 있다.
```bash
vim ~/.bashrc
```

그리고, 맨 아래에 다음 변수와 함수들을 적어준다.
```shell
GREEN="[\e[1;32m]" # bold
WHITE="[\e[0;37m]" # normal
BLUE="[\e[1;34m]"  # bold
RED="[\e[0;31m]"   # normal
RESET="[\e[0m]"

parse_git_branch() {
        git branch 2>/dev/null | grep '*' | sed 's/* (.*)/ (\1)/'
}
```

## 3. 적용

이제 위의 변수와 함수들을 사용해서 적용해주는 라인을 `bashrc`에 적어주어야 한다.
아래가 바로 그 프롬프트다.
```shell
export PS1="${GREEN}\u@\h${WHITE}:${BLUE}\w${RED}$(parse_git_branch)${RESET}$ "
```

이제 저장하고 나가서 적용하자.

```shell
source ~/.bashrc
```

그러면 이제 끝이다.
---
title: pyenv 명령어 정리 (내가 보려고)
date: 2024-09-07 09:00:00 +0900
categories: [pyenv, commands]
tags: [pyenv, python3, python, ubuntu, linux]
---

# pyenv 명령어 정리 (내가 보려고)

## 설치 (Ubuntu)

프로그램 파일 다운로드 받는다.
```bash
curl https://pyenv.run | bash
```

환경 변수 업데이트해준다.
```bash
vim ~/.bashrc
```
아래 내용을 맨 아래 삽입.
```
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
if command -v pyenv 1>/dev/null 2>&1; then
    eval "$(pyenv init -)"
fi
```
위가 안된다면 아래.
```
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

적용.
```bash
source ~/.bashrc
```

테스트
```bash
pyenv --version
```

## python 설치 (pyenv 이용)

Ubuntu용 업뎃이다. Mac은 `brew`이용하자.
```bash
sudo apt-get update
sudo apt-get install \
build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev \
libsqlite3-dev libncursesw5-dev xz-utils tk-dev libxml2-dev \
libxmlsec1-dev libffi-dev liblzma-dev
```

python 파일 다운로드.
3.12.5 대신 본인이 사용하고 싶은 버전 넣으면 됨.
```bash
pyenv install 3.12.5
```

현재 사용하는 python 버전 확인
```bash
pyenv versions
```

시스템 전역 python 버전 설정
```bash
pyenv global 3.12.5
```


## 추가 명령어

현재 directory 및 하위 directory에서의 python 버전 설정.
(해당 directory에서 .python-version 파일을 이용해 관리)
```bash
pyenv local 3.12.5
```

설정 python 버전으로 가상환경 생성
```bash
pyenv virtualenv 3.12.5 [Name of Virtualenv]
```

virtual environmnet 활성화
```bash
pyenv activate [Nme of Virtualenv]
```

virtual environmnet 비활성화
```bash
pyenv deactivate
```
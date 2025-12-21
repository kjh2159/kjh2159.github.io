---
title: Python 가상환경 초기 설정 정리
date: 2025-09-18 10:00:00 +0900
categories: [python, virtual environment]
tags: [python, conda, pyenv, venv, linux]
---

# Python 가상환경 초기 설정 정리

> Set-up 환경은 무조건 Linux Ubuntu 20.04 LTS 기준.
{: prompt-warning }

## 1. anaconda

### a. *Installer 다운로드*

[여기](https://www.anaconda.com/download)에서 anaconda installer 설치해준다.

```shell
sh Anaconda3-2020.04-Linux-x86_64.sh
source ~/.bashrc
```

### b. *가상환경 초기 설정*

터미널 열 때마다 conda 가상환경 열리는 기능을 꺼줄 수 있다.

```shell
conda config --set auto_activate_base false
```

anaconda를 키고 끄고 싶을 땐, 활성/비활성 아래 명령어를 쓰면 된다.

```shell
conda activate
conda deactivate
```

### c. *기본 명령어*

```shell
conda create -n python39 python=3.9
# conda create -n {venv-name} python={ver} - 새로운 가상환경을 만든다.

conda env list  
conda activate python39
```

## 2. pyenv

pyenv는 이미 이전 [포스트](https://kjh2159.github.io/posts/pyenv-commands/)에서 다룬 적이 있다. 해당 포스트를 참고하자.

## 3. python-venv

python-venv는 파이썬 표준 라이브러리 모듈이고, Debian/Ubuntu에선 버전별 패키지(python3.9-venv, python3.10-venv …)가 필요하다.
여기서는 python 3.10을 기준으로 보겠다. python3.10-venv는 python3.10 전용이므로, 다른 버전과는 호환되지 않는다.

### a. *Installation*

```shell
sudo apt-get update && sudo apt-get install python3.10 python3-distutils libpython3.10
sudo apt install python3.10-venv
```

### b. *기본 명령어*

pip는 다른 가상 환경의 pip와 충돌이 날 수도 있어서, 따로 추후 푸가 설치를 추천한다.

```shell
python3 -m venv MY_ENV --without-pip
# python3 -m venv {venv-name} --without-pip
source MY_ENV/bin/activate
python3 -m ensurepip --upgrade
```
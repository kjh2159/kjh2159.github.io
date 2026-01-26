---
title: Docker 세팅법 및 사용법 정리
date: 2026-01-17 10:00:00 +0900
categories: [Docker]
tags: [Docker, CI/CD, toolchain]
---

# Docker 세팅법 및 사용법 정리

## 1. 들어가며

Docker는 애플리케이션과 그 실행에 필요한 모든 의존성(라이브러리, 런타임, 설정 파일 등)을 하나의 **컨테이너(container)**로 패키징하여, 어떤 환경에서도 동일하게 실행될 수 있도록 해주는 컨테이너 기반 가상화 플랫폼이다. 전통적인 가상 머신(VM)이 운영체제 전체를 가상화하는 방식인 반면, Docker는 호스트 OS의 커널을 공유하면서 애플리케이션 단위로 격리된 실행 환경을 제공하기 때문에 훨씬 가볍고 빠르다. 이로 인해 개발 환경과 배포 환경 간의 차이로 발생하는 문제를 최소화할 수 있으며, CI/CD 파이프라인, 마이크로서비스 아키텍처, 재현 가능한 실험 환경 구축 등에서 사실상 표준 도구로 자리 잡았다. Dockerfile을 통해 실행 환경을 코드로 정의할 수 있다는 점 또한 인프라를 코드처럼 관리하는 현대적인 개발 방식과 잘 맞아떨어진다.

## 2. 설치

### A. Linux (e.g., Ubuntu)

Linux 환경에서는 Docker Desktop 없이도 Docker Engine을 직접 설치해 사용할 수 있다. 대부분의 배포판에서는 패키지 매니저를 통해 Docker를 설치할 수 있으며, 공식 저장소를 추가한 뒤 몇 가지 명령어만으로 바로 컨테이너 실행 환경을 구성할 수 있다. 이 방식은 불필요한 가상화 계층이 없기 때문에 성능과 자원 활용 측면에서 가장 효율적이다.

먼저, `apt`를 설정해주어서 내 terminal이 Docker 패키지를 인식할 수 있도록 만들어 준다.

```shell
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

그리고 아래의 명령어를 통해서 위에서 설정한 apt 세팅을 읽어서 이제 docker를 설치할 수 있다.

```shell
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Docker는 설치후 기본적으로 실행이 먼저 된다. 첫 번째 명령어를 통해서 docker 상태를 확인해서 실행 상태인지 알 수 있다. 하지만, 실행이 되지 않는다면, 두 번째 명령어를 사용해 실행할 수 있다.

```shell
sudo systemctl status docker
sudo systemctl start docker
```

### B. MacOS (Apple Silicon)

macOS에서는 공식적으로 [**Docker Desktop**](https://www.docker.com)을 통해 Docker를 설치한다. macOS는 리눅스 커널을 직접 실행할 수 없기 때문에, Docker Desktop은 내부적으로 경량 가상 머신을 사용해 컨테이너를 구동한다. 사용자는 별도의 가상화 설정을 신경 쓰지 않고도 Docker 명령어를 바로 사용할 수 있다는 점에서 설치와 사용이 비교적 간단하다.

### C. Windows

> 윈도우 환경은 OS 특성상 docker 사용시 추천되지 않는다.
{: .prompt-warning }

Windows 환경에서는 (**Docker Desktop**)[https://www.docker.com]을 설치하거나, **WSL(Windows Subsystem for Linux)**을 통해 Docker를 사용할 수 있다. Docker Desktop은 GUI 기반 관리 도구와 함께 Docker Engine을 손쉽게 제공하며, 내부적으로는 WSL 2를 백엔드로 사용해 리눅스 컨테이너를 실행한다. 반면, WSL 환경에 직접 Docker Engine을 설치하면 보다 가볍고 서버 환경과 유사한 방식으로 Docker를 운용할 수 있다.

WSL을 사용할 경우, 위의 Linux 설치 방법을 따르면 된다.

## 3. 기본 실행 방법

Docker의 기본 사용 흐름은 **이미지(image)를 준비하고 → 컨테이너(container)를 실행하는 것**으로 요약된다. 먼저 이미지는 `docker pull <이미지명>` 명령어로 레지스트리에서 다운로드하거나, `docker build -t <이름> .`을 통해 Dockerfile로 직접 생성할 수 있다. 이렇게 준비된 이미지를 확인하려면 `docker images`를 사용한다. 이미지는 실행 가능한 애플리케이션 템플릿이고, 실제로 동작하는 프로세스 단위가 컨테이너라는 점이 핵심이다.

컨테이너 실행은 `docker run`이 가장 중요한 명령어다. 예를 들어 `docker run -it <이미지명> /bin/bash`는 대화형 터미널로 컨테이너를 실행하며, `-d` 옵션을 주면 백그라운드에서 실행된다. 실행 중인 컨테이너 목록은 `docker ps`, 종료된 컨테이너까지 포함하려면 `docker ps -a`로 확인할 수 있다. 컨테이너를 중지할 때는 `docker stop <컨테이너ID>`, 삭제할 때는 `docker rm <컨테이너ID>`를 사용하며, 이미지 자체를 정리하려면 `docker rmi <이미지ID>`를 사용한다. 이 몇 가지 명령어만으로도 대부분의 기본적인 Docker 실행과 관리가 가능하다. 


## 4. 실무 예제

그 다음으로는 실무 예제를 통해서 흐름을 파악하고, 실무에서 쓰이는 추가 실행 옵션에 대해서 알아보도록 한다. 실행 예제를 통해 Docker 컨테이너를 어떻게 활용하는지, 그리고 어떤 옵션들이 왜 필요한지를 살펴보자. 아래 예제들은 로컬 개발 환경을 컨테이너 안으로 옮겨 툴체인·빌드 환경을 재현하는 전형적인 사용 패턴을 보여준다.

### A. 특정 플랫폼용 toolchain container

이 예제의 핵심 목적은 호스트 환경과 무관하게 동일한 빌드/개발 환경을 확보하는 것이다.

```shell
docker pull --platform linux/amd64 ghcr.io/snapdragon-toolchain/arm64-android:v0.3

docker run -it \
  -u $(id -u):$(id -g) \
  --volume $(pwd):/workspace \
  --platform linux/amd64 \
  ghcr.io/snapdragon-toolchain/arm64-android:v0.3
```

> `docker run`이 permission denied 에러를 보인다면, `sudo`릍 통해 실행해주면 된다. \
> 일반적으로 `sudo`를 붙이지 않으면 실행되지 않는 환경이 구축되기도 하는데 이는 보안을 위한 사항이므로 이상한 현상은 아니다.
{: .prompt-info }

`--platform linux/amd64` 옵션은 현재 머신이 ARM(M1/M2 Mac 등)이더라도 amd64 이미지를 강제로 실행하도록 지정한다. 이는 Android NDK나 특정 벤더 툴체인이 x86_64 환경을 전제로 빌드된 경우에 특히 중요하다.

`-u $(id -u):$(id -g)`는 컨테이너 내부 프로세스를 호스트 사용자 UID/GID로 실행하도록 설정한다. 이 옵션이 없으면 컨테이너에서 생성된 파일이 root 소유가 되어, 호스트에서 수정이나 삭제가 번거로워질 수 있다.

`--volume $(pwd):/workspace`는 현재 디렉토리를 컨테이너 내부 /workspace에 마운트하여, 컨테이너 안에서의 작업 결과가 그대로 로컬 파일 시스템에 반영되도록 한다. -it는 인터랙티브 터미널 사용을 위한 필수 옵션이다.

### B. 일회성 작업 패턴

이 패턴은 빌드, 테스트, 스크립트 실행처럼 한 번 쓰고 버리는 컨테이너에 적합하다.

```shell
docker pull ghcr.io/snapdragon-toolchain/arm64-android:v0.3

docker run --rm -it \
  -v ${PWD}:/work \
  -w /work \
  ghcr.io/snapdragon-toolchain/arm64-android:v0.3 bash
```

`--rm` 옵션은 컨테이너가 종료되면 자동으로 삭제되도록 하여, `docker ps -a`에 불필요한 컨테이너가 쌓이는 것을 방지한다. CI 환경이나 반복적인 실험에서 매우 자주 사용되는 옵션이다.

`-v ${PWD}:/work`는 앞선 예제와 동일하게 현재 디렉토리를 컨테이너에 마운트하고, `-w /work`는 컨테이너 시작 시 **작업 디렉토리(working directory)**를 /work로 설정한다. 이 덕분에 컨테이너에 진입하자마자 별도의 cd 없이 바로 작업을 시작할 수 있다. 마지막의 bash는 컨테이너 실행 시 기본 엔트리포인트 대신 Bash 셸을 실행하도록 지정한 것이다.

> docker shell을 나가려면, `exit`을 입력하면 된다.
{: .prompt-info }

--- 

출처: [도커](https://docs.docker.com/engine/install/ubuntu), [Snapdragon-toolchain 깃허브](https://github.com/snapdragon-toolchain/docker/releases/tag/v0.3), [시작하세요! 도커/쿠버네티스](https://www.yes24.com/product/goods/148124010)
---
title: CUDA toolkit 특정 버전으로 업데이트
date: 2024-09-11 00:30:00 +0900
categories: [GPU, CUDA]
tags: [cuda, version, update, pytorch, gpu, ai, Deep Learning, LLM, Linux, Ubuntu]
---

# CUDA toolkit 원하는 버전으로 업데이트

## 들어가며

최근 복학하며 기존에 있던 랩실로 돌아가게 되었다. 원래 하던 연구는 AI와는 크게 연관이 있진 않았다. 하지만, 최근 복귀하면서 교수님께서 한 번 제1저자로 도전해보자고 하는 연구 주제가 기존의 연구 주제 + AI였다. AI를 연구 주제에 섞는다라... 물론, 크게 문제 될 건 없지만, 연구실에서는 해당 소주제에 대해서 다뤄본 사람이 전무후무하다. 그래서, 혼자 랩실에서 고독한 싸움이 예상되던 와중, CUDA toolkit 버전을 업데이트 해야 했다.

## System Environment and Platform

내가 사용하는 환경은 아래와 같다.

- OS: Linux Ubuntu 20.04
- GPU: NVIDIA GeForce RTX 3080
- 가상환경: pyenv (conda 가능)

솔직히 가상환경은 크게 상관없다. 내가 pytorch를 pyenv를 통해서 설치했을 뿐이다.


## WHY?

### Problem

pytorch를 사용해서 프로그램을 실행하던 도중 이러한 에러가 발생했다.
```
The detected CUDA version (11.6) mismatches the version that was used to compile PyTorch (12.1). 
Please make sure to use the same CUDA versions.
```
일단, 해석해보면 PyTorch가 요구하는 CUDA version이랑 연동되지 않는다는 거다. PyTorch가 요구하는 CUDA는 12.1버전, 내 데스크탑에 설치되어 있는 CUDA는 11.6이다. 이걸 해결하기 위해서 pytorch를 내리던, cuda를 올리던 아니면, 둘다 업데이트를 하던, 어떤 선택을 내려야한다.

물론, pip로 할 수도 있다는 글도 보았다.
```
pip3 install torch==2.0.1+cu118 torchvision==0.15.2+cu118 --index-url https://download.pytorch.org/whl/cu118
```
> ref: [https://github.com/vllm-project/vllm/issues/1453](https://github.com/vllm-project/vllm/issues/1453)

물론 이것도 가능하겠지만, 일단 내가 돌리려고 하는 프로그램이 요구하는 pytorch 버전은 2.0.0 이상이다. 그리고 pytorch document 레퍼런스를 찾아보면, 문제가 ***pytorch 버전 2.0.0 이상은 CUDA 11.7 이상부터 호환***이 된다는 점을 발견했다. 그래서 눈물을 머금고 이 방법은 폐기했다.

### docker solution

대부분의 한글 포스팅은 docker를 사용하라고 한다. 근데, 나는 docker에 안 좋은 기억이 있어서, 딱히 선호하지 않는다. 그래서 나는 docker를 쓰지 않는 방법을 찾았다. "docker를 쓰지 않아도 가능한데 굳이?"라는 생각이 강한 탓이라고 생각한다. ***폐기***


## Environment 확인

### NVCC 확인

아래의 명령어로 각각 디렉토리에 bin파일이 있는지 확인.
```bash
cd ~/usr/local
ls cuda cuda-11.6
```

있다면, 있는 bin 파일에 대해서 nvcc 확인하자.

```bash
cuda/bin/nvcc --version
cuda-11.6/bin/nvcc --version
```

그러면, 아래와 같은 결과가 나올텐데, 본인이 가진 cuda 버전이 있는지 확인하자. release 부분을 확인하면 된다.

```shell
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2023 NVIDIA Corporation
Built on Mon_Apr__3_17:16:06_PDT_2023
Cuda compilation tools, release 11.6, V11.6.55
Build cuda_11.6.r11.6/compiler.30794723_0
```

본인이 원하는 버전이 있는지 없는지 잘 확인하자. 있는데, `nvcc --version`을 했을 떄 본인이 원하지 않는 다른 버전이 뜬다면, `.bashrc`가 잘못된 경로를 읽고 있는 것이니 path resolution만 잘 해주면 된다.

### NVIDIA Graphic Driver 확인

현재 새로 설치하고자 하는 CUDA와 호환되는 그래픽 드라이버가 설치되어 있어야 한다. 아래의 명령어로 확인하자.

```bash
nvidia-smi
```

```
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 545.23.08              Driver Version: 545.23.08    CUDA Version: 11.6     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA GeForce RTX 3080        On  | 00000000:01:00.0  On |                  N/A |
|  0%   37C    P8              20W / 340W |    298MiB / 10240MiB |     10%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+
                                                                                         
+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|    0   N/A  N/A      1229      G   /usr/lib/xorg/Xorg                          143MiB |
|    0   N/A  N/A      1415      G   /usr/bin/gnome-shell                         35MiB |
|    0   N/A  N/A     30858      G   ********                                     86MiB |
|    0   N/A  N/A    145721      G   gnome-control-center                          3MiB |
+---------------------------------------------------------------------------------------+
```
`Driver Version`부분을 보면 된다. `545.23.08` 이런 식으로 보면 되는데, 가장 맨 앞 세 자리만 보면 된다. 따라서 나의 버전은 545이다. 본인이 사용하려는 CUDA 버전과 Driver가 호환되는지 살펴보려면 아래 홈페이지를 참고하자.
> ref: [https://docs.nvidia.com/deploy/cuda-compatibility/](https://docs.nvidia.com/deploy/cuda-compatibility/) \
> 예를 들어, Linux x86_64 시스템에서 CUDA 12.x를 쓰고 싶다면 driver 버전이 525.60.13이상이어야 한다고 한다. 또는 r525라고 표기하는 경우도 있는데, 그 경우에는 525라고 생각하면 된다.
{: .prompt-info}

호환성은 필수로 확인하자. 제발.

### NVIDIA Driver 재설치

안 되는 경우 아래를 시도해보자. 물론, 나는 딱히 NVIDIA Driver 호환성 문제는 없었기에 설치하지는 않았지만, 이는 그냥 검색을 토대로 적었을 뿐이다.

```bash
sudo apt-get --purge remove '*nvidia*' # driver 제거
sudo apt-get install nvidia-driver-545 # 545버전 Driver 설치
sudo reboot #재부팅
```

## Let's Go! CUDA 업데이트!

### 1. 기존 CUDA 제거

기본적인 업데이트 방식은 `기존 CUDA toolkit 삭제 > 새로운 CUDA 설치`이다. 단순히 새 버전의 CUDA만 설치해도 괜찮지만, 충돌을 방지하려면 삭제를 추천한다.

```bash
sudo apt-get --purge remove "*cublas*" "cuda*" "nsight*" "nvidia-cuda"
sudo apt-get autoremove
```
기존 CUDA와 그와 관련한 package들을 자동 삭제한다.

### 2. CUDA 12.1 Repository 설정

CUDA 12.1버전을 설치하기 위해서는 `apt` repository에 `CUDA 12.1` repository를 추가해주어야 한다.

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
```
> 위 명령어는 Linux ubuntu 20.04에 대한 명령어이다. [https://developer.download.nvidia.com/compute/cuda/repos](https://developer.download.nvidia.com/compute/cuda/repos)에 가면 가서 본인이 사용하는 ubuntu 버전에 대한 경로를 찾아서 확인해보길 권한다. 꼭. `cuda-ubuntu2004` 등을 명령어에서 바꿔서 적자.
{: .prompt-warning}


위의 명령어를 실행하는 working directory는 어디서해도 상관없다. 그냥 관리하기 편하도록 `HOME/Downloads`가 편하지 않을까싶다. 

그리고 CUDA Repository를 추가한다.

```bash
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/3bf863cc.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
```

### 3. CUDA Toolkit 설치

repository 업데이트 후 CUDA toolkit 설치해줍시다. Let's get it, HO!

```bash
sudo apt-get update
sudo apt-get install cuda-12-1
```

버전과 호환성은 항상 잘 확인하시길.

### 4. 환경 변수 설정

vi, vim, nano 등 어떤 에디터는 사용해서 `.bashrc` 파일로 들어가준다. 난 vim 쓸 거다.

```bash
vim ~/.bashrc
```

그리고 맨 밑 줄에 아래의 환경 변수들을 추가해준다. 

```
export PATH="/usr/local/cuda-12.1/bin:$PATH" #또는 export PATH="/usr/local/cuda-12.1/bin:${PATH+:${PATH}}"
export LD_LIBRARY_PATH="/usr/local/cuda-12.1/lib64:$LD_LIBRARY_PATH"
```

그리고 변경된 `.bashrc`를 적용해주자.

```bash
source ~/.bashrc
```
### 5. Test

```bash
nvcc --version
```

본인이 원하는 버전 적용이 잘 되었을 것이다.

## 아.아..아...아....!

### 이런..

사실 나는 Environment 환경 확인 단계를 건너 뛰었다. 혹시나 하는 마음에 root directory로 들어가서 확인해봤다. CUDA 12 이미 깔려있었다. 이런 ~~시발!!!~~ 아마 11.6이 있었던 건 랩실 전 사용자가 특정 목적으로 바꿔놓았던 것 같다.. 하.. 괜히 이러고 있었네.. ㅋㅋ..
---
title: How to Allocate Resource on Processors
date: 2025-04-22 09:00:00 +0900
categories: [Linux, Android]
tags: [Android, Mobile, Rooting, kernel, CPU, Processor]
---

# How to Allocate Resource on Processors

## 1. 들어가며

일반적인 데스크탑에서 사용하는 CPU의 코어들은 모두 동일한 성능의 코어로 구성된 경우가 많다. 하지만, Android와 같은 모바일이 갖는 CPU는 big.LITTLE(빅리틀) 구조를 갖고 있다. 성능이 다른 코어를 배치함으로써 프로세스의 로드에 따라서 좋은 성능 또는 낮은 성능의 코어에 각 프로세스를 할당시켜 최적화한다. 

하지만, Termux와 같은 Application을 통해 Android에 Linux 터미널을 실행시키게 되면, 이러한 Shell은 기본적으로 CPU 하드웨어가 동일한 성능의 코어로 이루어져있다고 간주한다. 따라서, 적절한 성능의 코어에 Resource Allocation이 불가능하다는 점이다. 따라서, 때론 Android를 이용한 Mobile Computing 연구를 하다보면 특정 CPU 코어에 Process 등과 같은 Resource를 할당시키는 방법이 요구되곤 했다.


## 2. 명령어
이 방식은 Pixel9 기준으로 루팅을 요한다. 루팅이 되어있지 않다면, 루팅을 추천한다. 사용 방법은 크게 2가지를 둘 수 있다. 

### 2-1. 할당과 실행 동시에
다음은 특정 CPU에 할당할 프로세스를 설정하는 동시에 실행 시키는 방법이다. 
```shell

# su -c "taskset [mask] [cmd]"
su -c "taskset f0 runfile.sh" #ex
```
이때 `f0`가 무엇을 의미 하는지 알아야 한다. 이 16진수가 바로 어떤 CPU core에 할당할지에 대한 값이다. 
현재 사용하고 있는 실험용 휴대폰은 Pixel9이므로 Pixel9 기준으로 따지면, Google Tensor G4는 1개의 Prime, 3개의 Mid, 4개의 Little core로 이루어져있다. 따라서, 전체 8개의 코어가 있다. 그리고 `f0`는 이진수로 `11110000`이고 `taskset` 명령어는 1인 영역의 core에만 Process를 할당시키도록 만든다. 이 경우 Prime, Mid core 총 4개의 core에만 할당한다. 

S24와 같이 총 코어 수가 10개이거나 8개가 아닌 Chipset에 대해서는 해당 Chipset이 지닌 CPU의 코어수에 맞게 bit 수를 지정해주면 된다(ex. `3f0`).

> Termux에서의 taskset은 Toybox 버전이므로 16진수를 `f0`와 같은 형식으로 쓴다.
> 하지만, Toybox 버전이 아닌 다른 버전이 사용된다면, `0xf0`와 같이 prefix를 붙여줘야 한다.
{ :.prompt-info }

### 2-2. 실행중인 프로세스를 지정
이 방법은 이미 실행중인 프로세스에 대해 pid를 사용해 지정하는 방식이다.

```bash
# su -c "taskset -ap [mask] [pid]"
su -c "taskset -ap f0 13877" #ex
```
PID를 찾기 위해서는 소스 코드 내에서 찾을 수도 있고, 또는 `top` 명령어로 찾을 수도 있다.

### 2-3. C++ Application
C++에서 활용한다면, 다음과 같이 활용할 수 있다.

```cpp
#include <unistd.h>
#include <stdlib.h>

#include <iostream>
#include <string>

int main(){

    // get pid
    pid_t pid = getpid();
    std::cout << "PID: " << pid << std::endl;
    // perform taskset
    system("su -c \"taskset -ap f0 " + std::to_string(pid) + "\"");

    return 0;
}
```
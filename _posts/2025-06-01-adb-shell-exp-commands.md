---
title: 실험용 ADB SHELL 명령어 모음 
date: 2025-06-01 09:00:00 +0900
categories: [Linux, Android]
tags: [Android, Mobile, Rooting, kernel]
---

# 실험용 ADB SHELL 명령어 모음

## 1. Auto Screen Lock Time Control

화면 자동 잠금 시간 제어하는 코드이다. 
설정 시간 단위는 ms이다.
```shell
adb shell settings put system screen_off_timeout 180000
```

180000ms는 30분을 의미한다. 현재 설정을 보려면 `put` 대신 `get`을 쓰면 된다.


> 앞으로 이 글의 명령어는 계속 추가될 예정이다.
{: .prompt-info }

## 2. Core Affinity Setting

Core Affinity는 어떤 프로세스를 특정 코어에서만 연산될 수 있도록 우선순위를 두는 것이다.
이는 말 그대로 '우선순위'이기 때문에 기기의 상황에 따라서 이 친화도를 파기할 수 있으나 대부분 지킨다.

C/C++ 코드로도 가능한데 여기서는 안 다루고, 다른 포스트에서 다룰 예정이다.
Shell로 통제할 경우, 루팅이 필요하다.

```shell
# {core-affinity}
#   11110000 = f0
#   <-big   little->
su -c "taskset {core-affinity} {command}"
```

여기서 core affinity 섹션으로 정해줘야 하는데, 코어 수만큼의 2진수를 16진수로 변환한 값을 넣어야 한다.
1은 활성, 0은 비활성이다.
가령, 일반적인 모바일 big.LITTLE 구조의 CPU는 8코를 갖고 `1*BIG + 3*MID + 4*LITTLE` 구조로 구성된다.
그래서 BIG, MID 코어를 활성화시키고 싶으면, `11110000`을 16진수로 변환한 값인 `f0`으로 넣으면 된다.

코어가 더 많거나 적은 경우도 있다. 예를 들어, S24는 `1*BIG + 2*MIDH + 3*MIDL + 4*LITTLE`이다.
BIG과 MIDL만 활성화 시키고 싶다면, `1001110000`을 16진수로 변환한 `270`을 넣으면 된다.
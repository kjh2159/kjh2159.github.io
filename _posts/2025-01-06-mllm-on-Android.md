---
title: 안드로이드에서 mllm 구동 (앱버전)
date: 2025-01-06 06:00:00 +0900
categories: [mllm, llm]
tags: [mllm, llm, mobile, android, ai, gguf, github, application]
---


# 1. 들어가며

mllm은 CLI 버전과 앱 버전 2가지가 존재한다. 나는 앱버전을 Android에 deploy할 예정이다.
Chipset은 Snapdragon 8 Gen 1 이상을 추천한다.
mllm 앱 버전은 mllm Team에서 앱을 위한 repository를 따로 관리하고 있다: [mllm](https://github.com/UbiquitousLearning/mllm), [mllm App](https://github.com/lx200916/ChatBotApp/tree/master).

### 1-1. 환경 세팅

Deploy를 위해서는 준비물이 필요하다.
> Build: Linux OS PC
> Deploy: Android Studio (ver. >= ladybug)
{: .prompt-info}

C++과 Java를 이어주는 JNI Bridge API를 활용하기 위한 build는 Linux OS를 요구한다.
다른 OS는 이 Build를 지원하지 않는다.
다른 OS도 충분히 가능하지 않을까 싶긴 한데 아무튼 제작자가 그렇게 말했으므로 나는 그냥 안전하게  Ubuntu에서 Build했다.

# 2. Build

### 2-1. 환경 업데이트

```bash
sudo apt update && apt upgrade
sudo apt-get install git cmake
git clone https://github.com/UbiquitousLearning/mllm.git
cd mllm

```

### 2-2. Android NDK 다운로드

이 [링크](https://developer.android.com/ndk/downloads)에서 Android NDK를 다운받고 압축을 풀어준다.
나는 조금 더 직관적으로 전달하고자 하는 입장이므로 `$HOME` 경로에 저장하였다.
그리고 작성일자 기준의 Android NDK 버전이므로 경로에 버전이 틀리지 않도록 주의한다.

### 2-2. JNI Lib Build

```bash
cmake . \
-DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
-DCMAKE_BUILD_TYPE=Release \
-DANDROID_ABI="arm64-v8a" \
-DANDROID_NATIVE_API_LEVEL=android-28  \
-DNATIVE_LIBRARY_OUTPUT=. -DNATIVE_INCLUDE_OUTPUT=. $1 $2 $3 \
-DARM=ON \
-DAPK=ON \
-DQNN=ON \
-DDEBUG=OFF \
-DTEST=OFF \
-DQUANT=OFF \
-DQNN_VALIDATE_NODE=ON \
-DMLLM_BUILD_XNNPACK_BACKEND=OFF

make mllm_lib -j 4

```
이렇게 하면 `mllm` directory에 `libmllm_lib.a`이란 파일이 생긴다. 그러면 이 파일을 `ChatBotApp` directory에 옮겨줘야 한다.

# 3. Deploy

이제 Android Studio를 설치한 PC가 필요하다.
Linux, Mac, Window 상관 없다.
해당 PC에 git과 adb가 설치되어 있으면 좋다.
adb는 이 [링크](https://developer.android.com/tools/adb)에서 다운로드 받을 수 있다.

### 3-1. Project

```bash
git clone https://github.com/lx200916/ChatBotApp/tree/master
```

Android Studio를 통해 Open Project로 해당 repository를 열어준다.

### 3-2. Move JNI lib

그리고 아까 만든 `libmllm_lib.a` 파일을 `ChatBotApp/app/src/main/cpp/libs/` 경로에 복사해준다.


### 3-3. Download Models

먼저, mllm 엔진의 파일 포맷에 대해서 알아야 한다. 
llama.cpp는 gguf라는 파일 포맷을 사용하듯 mllm 엔진 또한 mllm이라는 특정 파일 포맷을 갖는다.
하지만, 이에 더불어 `vocab.mllm`과 `merges.txt` 파일을 요구한다. 

이제 모델을 다운로드 받아야 한다. 먼저 Qwen 1.5 1.8B 모델로 시험해본다.
이 [링크](https://huggingface.co/mllmTeam/qwen-1.5-1.8b-chat-mllm/blob/main/qwen-1.5-1.8b-chat-q4_0_4_4.mllm)에서 Qwen 1.5를 다운로드 받을 수 있다.
다른 모델을 다운로드 받고 싶다면, 이 [링크](https://huggingface.co/mllmTeam)의 **Models** 부분에서 다운로드 받을 수 있다.

그리고 앞서 설명한 바와 같이 `vocab.mllm`과 `merges.txt` 파일이 필요하다. 이는 `mllm` repository에서 가져올 수 있다.
아래의 명령어 실행하기 이전에 폰과 연결하고 휴대폰의 usb debugging을 허용해준다.

```bash
git clone https://github.com/UbiquitousLearning/mllm # 이미 clone해놨다면 생략

adb shell mkdir sdcard/Download/model
adb push mllm/vocab/qwen_vocab.mllm sdcard/Download/model/
adb push mllm/vocab/qwen_merges.mllm sdcard/Download/model/

adb push {DOWNLOAD-PATH}/qwen-1.5-1.8b-chat-q4_0_4_4.mllm sdcard/Download/model/
```

`{DOWNLOAD-PATH}`에는 본인이 qwen-1.5-1.8b를 다운로드 받은 경로를 넣어주면 된다.

### 3-4. Run

이제 준비가 다 끝났다. Android Studio에서 Run버튼을 눌러주면, 잘 동작한다.
다만, Model Settings에서 Qwen 1.5와 CPU를 선택해줘야 하는 점을 잊지 말자.

완.

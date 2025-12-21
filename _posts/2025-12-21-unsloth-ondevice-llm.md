---
title: Unsloth의 on-device LLM 어플을 윈도우를 통해 안드로이드 폰에 deploy하기
date: 2025-12-21 10:00:00 +0900
categories: [Android, LLM]
tags: [Android, LLM, AI, Unsloth, ExecuTorch, PyTorch]
---

# Unsloth의 on-device LLM 어플을 윈도우를 통해 안드로이드 폰에 deploy하기

## 1. 소개

최근 On-device LLM에 대한 관심이 빠르게 증가하고 있습니다. 
그러던 와중 Unsloth가 정말 좋은 On-device LLM 예시 코드를 공개했습니다.
그에 따라 코드를 deploy하는 방식에 대해 다루어 보고자 합니다.

이 어플은 Unsloth, TorchAO, 그리고 PyTorch·Meta가 공동 개발한 ExecuTorch를 활용해 LLM을 학습한 뒤 안드로이드 스마트폰에 직접 배포하는 전체 워크플로우를 갖는다. 
TorchAO 기반의 quantization-aware training(QAT) 을 적용해 양자화 이후에도 정확도 손실을 최소화하고, 이를 ExecuTorch 포맷으로 변환해 로컬 실행하는 과정을 소개한다. 

이 접근은 Instagram과 WhatsApp 등 대규모 서비스에 실제 사용되는 ExecuTorch 기술 스택을 기반으로 하며, 서버 없이도 즉각적인 응답, 오프라인 동작, privacy-first 추론 환경을 제공한다. 또한, 무료 Colab 노트북을 통해 모델 파인튜닝부터 모바일 배포까지의 전 과정을 실습 형태로 설명한다. 실제 실험 결과로 Qwen3-0.6B 모델 기준 Pixel 8 및 iPhone 15 Pro에서 약 40 tokens/s까지의 생성 속도를 보인다.

이번에는 안드로이드 스튜디오 없이 안드로이드에 배포하는 방법을 다룬다.


## 2. 배포 준비하기

들어가기 앞서, 나는 cmd나 powershell을 쓰지 않고 git bash를 쓸 예정이다. 
다른 shell emulator에서는 안 될 가능성이 있으니 git bash를 사용해주도록 하자.

### *A. Java 설치*

먼저, [여기](https://learn.microsoft.com/en-us/java/openjdk/download)에서 `openjdk-17` 설치를 통해 java를 사용할 수 있도록 해준다. *잔말말고, openjdk-17버전을 깔자. 다른 버전은 안 된다.*

보통 Auto로 설치하게 되면 알아서 환경 변수까지 지정은 하지만, git-bash에서는 경로 해석 방식이 조금 달라서 큰 의미가 없다.
따로 git-bash 자체에서 Linux처럼 export 또는 지정해주는 것이 좋다.
따라서, 아래의 명령어를 입력하도록하자.

```bash
export JAVA_HOME="/c/Program Files/Microsoft/jdk-17.0.17.10-hotspot"
export PATH=$JAVA_HOME/bin:$PATH
```
> jdk 세부 버전 잘 보고 경로를 추가해주자.
{: .prompt-warning}

잘 되었으면 아래를 통해 잘 나오는지 확인해보자.

```bash
java -version
echo $JAVA_HOME
```

### *B. Android commandline tools 설치*

commandline tools를 위치시켜놓을 디렉토리를 만들어야 한다. 아래의 명령어를 통해 만들어 주자.

```bash
mkdir -p ~/android-sdk/cmdline-tools
```

그리고, [여기](https://developer.android.com/studio?hl=en#command-line-tools-only)에서 `command-line-tools-only` 섹션에서 command-line tools를 설치해준다. 그리고, 설치된 zip 파일을 위의 경로 cmdline-tools 안에서 압축해제한다. 그러면, 기본적으로 경로 구성이 `android-sdk/cmdline-tools/cmdline-tools/*` 이렇게 될 것이다. 여기서 가장 마지막의 `cmdline-tools`를 `latest`로 바꿔준다. 최종적으로 `android-sdk/cmdline-tools/latest/*` 경로 구성이 되어야 한다.

그런 다음, 아래 명령어를 통해 경로를 등록해주자.

```bash
export ANDROID_HOME=$HOME/android-sdk
export PATH=$ANDROID_HOME/cmdline-tools/latest/bin:$PATH
```

### *C. Android SDK components 설치*

이제 우리는 sdkmanager를 통해 여러 SDK components를 설치할 수 있다. 
아래의 명령어를 통해서 라이선스 동의와 platform-tools, ndk 등을 설치해주자.

```bash
yes | sdkmanager.bat --licenses
sdkmanager.bat "platforms;android-34" "platform-tools" "build-tools;34.0.0" "ndk;25.0.8775105"
```
> 버전에 주의하자. 완전히 동일한 버전을 추천한다(다른 버전 안 해봄).
{: .prompt-warning }

그럼 이제 다시 경로에 설치한 것들을 등록해주자.

```bash
export PATH=$ANDROID_HOME/platform-tools:$PATH
export ANDROID_NDK=$ANDROID_HOME/ndk/25.0.8775105
```

## 3. Get the Code


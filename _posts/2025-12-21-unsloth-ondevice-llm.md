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
> 혹시나 `java -version` 결과가 not found error라면, git bash를 재시작하거나 컴퓨터를 재시작하면  된다.
{: .prompt-info }

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

드디어 사전 준비가 끝났다. 이제 코드를 받아서 빌드만 해주면 된다.

```
cd /path/that/you/want
git clone https://github.com/meta-pytorch/executorch-examples.git
cd executorch-examples
```

향하는 경로를 잘보고 clone을 해주자.

## 4. Build the APK

```bash
./gradlew :app:assembleDebug
```
> 에러가 발생하면, 아래 trouble shooting을 보고 오자.

이제 아래의 주소에서 빌드된 `.apk` 파일을 발견할 수 있을 것이다.

```bash
app/build/outputs/apk/debug/app-debug.apk
```

## 5. 앱 설치

먼저, `adb`를 통해 설치해줄텐데, adb를 휴대폰에 연결해줄 것이다.
아래의 명령어를 컴퓨터와 휴대폰을 연결하자.

```bash
adb devices
```
"attached device"라고 뜨면 연결이 잘 된 것이다. 
그리고, 위의 `.apk` 파일을 휴대폰에 아래 명령어를 통해 설치해주자.

```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

## 6. 모델 다운로드 및 임베딩

*이 스텝도 마찬가지로 휴대폰을 PC와 연결한 상태에서 진행하도록 하자.*

[이곳](https://huggingface.co/kjh2159/Qwen3-0.6B-unsloth-pte/tree/main)에 중간 파일들을 모두 업로드해놓았다.
Unsloth가 기본 모델의 `.pte` 파일을 안 올려놓아서, 따로 학습을 진행해 업로드해두었다.
학습은 A6000 4장 기준으로 모델 파일 변환까지 합쳐 30분 정도 걸린 것같다.

일단 필요한 파일들만 다운로드 받아 엔진에 삽입하자.
`curl` 명령어가 안 된다면, 먼저 다운 받자. 또는 그냥 huggingface가서 직접 다운로드 받아도 괜찮다.

```bash
# 모델 다운로드
curl -L https://huggingface.co/kjh2159/Qwen3-0.6B-unsloth-pte/resolve/main/qwen3_0.6B_model.pte
curl -L https://huggingface.co/kjh2159/Qwen3-0.6B-unsloth-pte/resolve/main/tokenizer.json

# 모델 삽입 공간 생성
adb shell mkdir -p /data/local/tmp/llama
adb shell chmod 777 /data/local/tmp/llama

# 모델 삽입
adb push qwen3_0.6B_model.pte /data/local/tmp/llama
adb push tokenizer.json /data/local/tmp/llama
```

## 7. 모델 적용

![Desktop View](assets/img/posts/ExecuTorchLlamaDemo.jpg){: width="300" height="533" .w-50 .right}
설치된 앱을 키고, 우측 상단의 설정(톱니바퀴)을 열어 모델 파일, tokenizer를 선택해주면 되고, 모델 타입도 Qwen3로 바꿔주면 끝이다.
우측은 실제 결과이다.

## Trouble Shooting

### *A. Gradle class 오류*

```shell
오류: 기본 클래스 org.gradle.wrapper.GradleWrapperMain을(를) 찾거나 로드할 수 없습니다.
원인: java.lang.ClassNotFoundException: org.gradle.wrapper.GradleWrapperMain
```

이 에러는 기존에 git bash에서 conda를 사용하게 될 때, conda 경로와 충돌하여 발생할 수 있다.
conda를 비활성화해주면 해결된다.

```
conda deactivate
```

### *B. SDK Location not found 오류*

sdk 디렉토리를 명확하게 잡을 수 있도록 설정해주면 된다.
```bash
echo "sdk.dir=$HOME/android-sdk" > llm/android/LlamaDemo/local.properties
```

### *C. Cannot find symbol 오류*

Deprecated code 때문에 발생할 수 있다. 아래 코드를 실행해주자.
```bash
sed -i 's/e.getDetailedError()/e.getMessage()/g' llm/android/LlamaDemo/app/src/main/java/com/example/executorchllamademo/MainActivity.java
```




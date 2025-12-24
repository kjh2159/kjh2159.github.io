---
title: On-device LLM API 분석 방법 (Unsloth 예제) [1편]
date: 2025-12-23 06:00:00 +0900
categories: [Android, LLM]
tags: [Android, LLM, AI, Unsloth, ExecuTorch, PyTorch]
---

# On-device LLM API 분석 방법 (Unsloth 예제)

## 들어가며
일단, on-device LLM을 연구하는 입장에서 우리는 단순히 on-device LLM이 모바일에서 동작하기만 해서는 안된다.
결국 LLM의 핵심 동작 흐름인 MHA(Multi-head attention), FFN(Feedforward network) 등이 어떻게 구현되어있는지 파악하고 커스터마이징할 수 있어야 한다.

llama.cpp, mllm 같은 유명한 mobile LLM 엔진들은 기반 자체가 pure C++로 구성되어 있고 CLI(Command Line Interface) deployment가 기본이기 때문에 그 자체로 내부에 구현체들이 모두 존재한다. 하지만, 이번에 다룰 Unsloth의 엔진이나 MediaPipe는 모바일 어플리케이션 GUI 상에서 동작할 때 API 기반으로 돌아간다.

따라서, 우리는 어떤 API를 사용하여 해당 on-device LLM이 동작하는지 분석하고 이해할 필요가 있다. 이번 포스트에서는 그 과정을 담고자 했다.

## 1. On-device LLM workflow 파악

> 이전 deployment 포스트에서는 Android Studio가 없는 것을 기준으로 했었는데, 이번 포스트에서는 사용하고 있다는 것을 가정으로 한다.
{: .prompt-info }

### *A. 구현 구조 파악*

가장 먼저 Java/Kotlin으로 구성된 코드 파일명과 구현체를 살펴봐야 한다. `app/kotlin+java/com.executorchllamademo/`에서 파일명을 확인해보면, 'callback'이라는 이름의 함수 또는 inferface만 제공하는 함수가 있을 가능성이 높다. 이것이 바로 software 구현에서 매우 중요한 encapsulation을 의미한다. 이는 ***실제 연산 구현체가 Java/Kotlin에 존재하지 않음을 의미***하는 동시에 API를 통해 구현되어 있음을 뜻한다. 

실제 Unsloth 예시에서는 해당 파일 이름은 `ModelRunnerCallback` 함수이다. 그리고 이는 interface class이므로 Android studio에서는 usage를 확인할 수 있다. 그 usage를 확인해보면, `ModelRunner.java`로 향한다(사실 이름만 보고도 `ModelRunner`에서 사용된다는 점을 알 수 있긴 하다.. ㅎ). 그럼 아래와 같은 2가지의 import를 볼 수 있다.

```java
import org.pytorch.executorch.extension.llm.LlmCallback;
import org.pytorch.executorch.extension.llm.LlmModule;
```

### *B. API 정체 파악*

이를 통해 `LlmCallback`, `LlmModule`이라는 파일이 있는지 확인해보면 존재하지 않으며, `org.pytorch.executorch.extension`에서 함수들을 가져온다는 것을 알 수있다. 그리고, `ctrl + 클릭`을 통해 LlmModule을 들어가보면, 아래와 같은 import가 있다.

```java
import com.facebook.jni.HybridData;
import com.facebook.jni.annotations.DoNotStrip;
```

이를 통해서 Native method + HybridData 조합을 사용하기 때문에 우리는 ***JNI를 통해 소스 코드를 빌드해서 가져온다는 점***을 알 수 있다. 따라서, 이 프로젝트는 JNI로 구성된 API를 통해 구현되어 있다는 점을 알 수 있다.

또한, 다음과 같은 특징이 있다.

- 실제 파일 경로가 없다.
- Java로 구현되어 있으나 수정이 되지 않는다.

따라서, 핵심 Interface class를 구성하기 위한 핵심은 *external libraries에 있고 JAR(AAR)에서 온 디컴파일된 API라고 해석*할 수 있다.

다시 정리하자면, ***C/C++로 구현(JNI라면 대부분)되어 있는 코드를 빌드하고, 그것을 JAR(AAR)로 Java bridge를 만들어 Android에서 동작할 수 있도록 API를 완성시켰다***고 볼 수 있다.

## 2. AAR 구현 파악

그렇다면, 이 JAR(AAR)이 어디서부터 오는가를 분석해야 한다. `build.gradle.kts (Module: app)`를 확인해보면, 아래와 같은 줄이 보인다. 

```gradle
if (useLocalAar == true) {
    implementation(files("libs/executorch.aar"))
} else {
    implementation("org.pytorch:executorch-android:1.0.1")
    // skip
}
```

따라서, 이 파일은 결국 `executorch`의 aar로 온다는 점을 명확히 할 수 있다. 정확히는 따로 파일이 없다면, `org.pytorch:executorch-android:1.0.1`로부터 다운로드 받는다. 

그리고, 이 내부에서 어떤 `.so` 실제 구현체를 불러오는지 알기 위해서는 실제 build를 돌려보면 알수있다.

```bash
./gradlew :app:assembleDebug
```

이전 포스트에서 빌드했던 바와 같이 위의 명령어를 사용해 빌드를 해보면, 아래와 같은 문구가 뜬다.

```
> Task :app:stripDebugDebugSymbols
Unable to strip the following libraries, packaging them as they are: libc++_shared.so, libexecutorch.so, libfbjni.so, libimage_processing_util_jni.so.
```

따라서, 위의 `.so` 파일들로부터 API가 구성되어 있다는 것이다. 


## 3. C++ 구현체 확인

`.so`와 `jni`를 통해서 일단, 구현체는 C/C++로 구현되어 있을 가능성이 높음을 확인했다. 그럼, 이들은 어디에 구현되어 있는 것인가?
결론은 우리가 확인한 `org.pytorch:executorch-android:1.0.1`를 빌드하기 위한 소스코드와 그리고 그 내부를 구성한 .so의 구현체를 분석하고 싶다는 것이다. 어차피 이를 gradle로 implementation되어 있다는 점은 단순히 이미 빌드되어 있는 API를 다운로드 받아서 사용할 뿐이라는 것이다.

### *A. 탐색*

unsloth의 예제가 담긴 github [repo](https://github.com/meta-pytorch/executorch-examples/tree/main/)를 확인해보면, `third-party`라는 디렉토리가 있는데 보통 서드파티(3rdparty, third_party) 경로에서 API를 불러올 수 있도록 세팅하는 경우가 많다.
그러면, 실제 구현체가 있는 `executorch` repo로 향하게 된다.

### *B. 구현체 탐색*

이 [repo](https://github.com/pytorch/executorch/tree/8c84780911e7f4e9eb19395181b04761392a4b56) 주소가 그 곳이다.

```
```
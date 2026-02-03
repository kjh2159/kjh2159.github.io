---
title: C/C++로 Perfetto 사용해 Android 분석하기
date: 2026-02-02 12:00:00 +0900
categories: [LLM, Perfetto]
tags: [LLM, Android, Perfetto, Linux]
---

# C/C++로 Perfetto 사용해 Android 분석하기

## 1. 들어가며

Perfetto 분석이 필요하여, perfetto docs를 뒤지던 중 Perfetto SDK가 있음을 확인했다.
그 전에는 이런 게 있는 줄 몰랐는데 C/C++로 구현할 수 있으면 훨씬 확장성이나 적은 overhead로 구현할 수 있어 잘됐다 싶었다.

## 2. 환경 설정

일단, 나 같은 경우는 perfetto를 github repository의 submodule로 만들어서 사용했는데, 이 게시글에서는 매우 간단한 버전으로 구성하도록 하겠다.

```shell
mkdir -p perfetto_ex && cd perfetto_ex
git clone https://github.com/google/perfetto -b v50.1
```

## 3. 라이브러리 빌드

```shell
cd perfetto/examples/sdk
cmake -S . -B build
cmake --build build
```

이렇게 하면 우리는 이제 리눅스와 안드로이드에서 사용가능한 라이브러리를 빌드한 것이다.
따라서, C/C++ 파일에서 *perfetto*를 include할 수 있다.

## 4. C++ 코드 작성

먼저, `perfetto_ex` 경로로 향해준다.

```shell
cd ../../../ # perfetto_ex
```

그 다음, C++ 코드를 작성해준다.
파일이름은 일단 `main.cpp`로 해두겠다.

```cpp
// main.cpp

#include <perfetto.h>
#include <fstream>
#include <thread>
#include <chrono>

int main() {
  // 1) system trace backend
  perfetto::TracingInitArgs args;
  args.backends = perfetto::kSystemBackend;
  perfetto::Tracing::Initialize(args);

  // 2) construct TraceConfig
  perfetto::TraceConfig cfg;
  cfg.add_buffers()->set_size_kb(16384);  // 16MB

  // --- android.power ---
  {
    auto* ds = cfg.add_data_sources();
    auto* dsc = ds->mutable_config();
    dsc->set_name("android.power");

    perfetto::protos::gen::AndroidPowerConfig p;
    p.set_battery_poll_ms(250);
    // defined in AndroidPowerConfig
    // (CHARGE=1, CAPACITY_PERCENT=2, CURRENT=3, CURRENT_AVG=4, VOLTAGE=5 등) :contentReference[oaicite:9]{index=9}
    p.add_battery_counters(perfetto::protos::gen::AndroidPowerConfig::BATTERY_COUNTER_CAPACITY_PERCENT);
    p.add_battery_counters(perfetto::protos::gen::AndroidPowerConfig::BATTERY_COUNTER_CHARGE);
    p.add_battery_counters(perfetto::protos::gen::AndroidPowerConfig::BATTERY_COUNTER_CURRENT);
    p.add_battery_counters(perfetto::protos::gen::AndroidPowerConfig::BATTERY_COUNTER_VOLTAGE);

    p.set_collect_power_rails(true);
    p.set_collect_energy_estimation_breakdown(true);

    dsc->set_android_power_config_raw(p.SerializeAsString());
  }

  // --- linux.ftrace ---
  {
    auto* ds = cfg.add_data_sources();
    auto* dsc = ds->mutable_config();
    dsc->set_name("linux.ftrace");

    perfetto::protos::gen::FtraceConfig f;
    f.set_buffer_size_kb(4096);  // per-cpu ftrace ring buffer size :contentReference[oaicite:10]{index=10}
    f.add_ftrace_events("devfreq/devfreq_frequency");
    f.add_ftrace_events("sched/sched_switch");
    f.add_ftrace_events("sched/sched_waking");
    f.add_ftrace_events("power/cpu_frequency");
    f.add_ftrace_events("power/cpu_idle");

    dsc->set_ftrace_config_raw(f.SerializeAsString());
  }

  // 3) session start / termination
  auto session = perfetto::Tracing::NewTrace(perfetto::kSystemBackend);
  session->Setup(cfg);
  session->StartBlocking();

  // workload
  std::this_thread::sleep_for(std::chrono::seconds(5));
  // add workload if you needed

  session->StopBlocking();

  // 4) save trace output :contentReference[oaicite:12]{index=12}
  std::vector<char> trace_data = session->ReadTraceBlocking();
  std::ofstream out("./traces.perfetto-trace", std::ios::binary);
  out.write(trace_data.data(), static_cast<std::streamsize>(trace_data.size()));
  out.close();

  return 0;
}
```

> Perfetto 버전마다 함수명이 다를 수 있다. \
> 해당 버전은 v50.1임을 명심한다.
{: .prompt-warning }


## 5. CMakeLists 작성

기존에 빌드해두었던 perfetto 라이브러리를 가져와 함께 빌드시켜주기 위해서 이를 위한 CMakeList.txt를 작성하도록 하겠다.

```cmake
cmake_minimum_required(VERSION 3.13)
project(perfetto_test)
set(CMAKE_CXX_STANDARD 17)
find_package(Threads REQUIRED)

message(STATUS ${CMAKE_SOURCE_DIR})

include_directories(${CMAKE_SOURCE_DIR}/perfetto/include)
include_directories(${CMAKE_SOURCE_DIR}/perfetto/sdk)
add_library(perfetto STATIC ${CMAKE_SOURCE_DIR}/perfetto/sdk/perfetto.cc)

add_executable(perfetto_test main.cpp)
target_link_libraries(perfetto_test perfetto Threads::Threads)

if(ANDROID)
  target_link_libraries(perfetto_test log)
endif()
```

## 6. 빌드 및 실행

```shell
cmake -S . -B build
cmake --build build --config Release
./build/perfetto_test
```

이렇게 하면 실행이 가능하다. 빌드 에러가 뜬다면 댓글 부탁드립니다.

그리고 실행 이후에는 `traces.perfetto-trace`라는 파일이 생기는데, 이 파일을 [Perfetto UI](https://ui.perfetto.dev)에 해당 파일을 삽입하여 값을 확인해볼 수 있다.


## 관련 게시글

기존에 adb shell에서 perfetto [측정](https://kjh2159.github.io/posts/perfetto/)과 [분석](https://kjh2159.github.io/posts/perfetto-analysis/)에 대해서 다룬 적이 있으므로 참고하면 좋다.

또한, 내 github [repository](https://github.com/kjh2159/mllm)에 어떻게 submodule로 구성하여 사용하였는지 예시로 두었기에 확인해보면 좋을 것 같다. 

---
출처

[Perfetto Docs](https://perfetto.dev/docs/instrumentation/tracing-sdk)
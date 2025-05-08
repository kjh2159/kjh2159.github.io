---
title: How to Measure Memory Bandwidth in Real Time on Android
date: 2025-05-08 01:00:00 +0900
categories: [Linux, Android]
tags: [Android, Mobile, Rooting, kernel, Memory, Bandwidth, Measurement]
---

# How to Measure Memory Bandwidth in Real Time on Android

> 본 글은 Google Pixel9 루팅 기기를 기준으로 작성하였다.
{ :.prompt-warning }

## 들어가며

일단, 이거 찾으려고 엄청나게 삽질했다. 며칠을 헤맸다.
나는 이 방법을 찾게 된 이유는 실시간으로 활성화되는 Memory Bandwidth를 측정해야 했기 때문이다.
대부분은 Memory가 얼마나 찼는지만을 수집하는데, 연구하면서 나에게 중요했던 데이터는 **Memory Bus의 활성화 정도**이다.
이 값을 측정하는 문서나 논문을 거의 못 찾았다.
논문에서는 GPU-RAM만 측정한다.

## 1. DSU

ARM DSU (DynamIQ Shared Unit)에 대해서 먼저 이해해야 한다. DSU는 ARM이 2016년에 공개한 Mobile에서 big.LITTLE 구조의 Multicore CPU를 효율적으로 최적화해 활용하기 위한 Control Unit이다. 그리고 이 DSU는 ISA 기준으로 따지면 ARMv8.2-A부터 적용되었고, SoC 기준으로는 Snapdragon 845부터 적용되었다. 그래서 이 Unit을 통해 공유 L3 Cache가 도입되기 시작했다. 그래서 실질적인 구조로는 CPU - DSU - RAM로 구성된다. 여기서 DSU가 L3 Cache 등을 포함하는데, 구성을 더 자세히 알고 싶다면, 이 [사이트](https://developer.arm.com/documentation/100453/0401/The-DynamIQ-Shared-Unit/About-the-DSU)를 참고하면 된다.

## 2. 명령어

DSU 섹션에서 설명한 바와 같이 결국 우리가 실제 RAM Bandwidth를 측정하기 위해서는 DSU-RAM의 Bandwidth를 측정해야 한다. 
이를 위해서 명령어 `simpleperf`를 사용하였다. Memory Bandwidth 측정의 기본 형태만을 이곳에서 소개하도록 하겠다.

### 2-1. 측정 이벤트 확인

`simpleperf`를 사용하기 위해서는 측정 가능한 이벤트를 확인해야 한다. 굉장히 많은데, 우리는 DSU에 관련된 이벤트를 확인해야 한다.
아래의 명령어로 dsu 관련한 이벤트를 확인할 수 있다.
```bash
simpleperf list | grep -i "dsu"
# or
simpleperf --help
```
기기마다 이 이벤트 이름은 다를 수 있으니 꼭 확인해야 한다.
그리고, help를 확인해 보는 것도 좋다.

### 2-2. 측정 명령어

추가 명령어는 `--help`로 확인해보면 더 다채롭게 명령어를 구성할 수 있다.
`l3d_cache_refill`, `l3d_cache_wb`는 각각 Memory Read와 Memory Write를 의미한다.
```bash
simpleperf stat -a --duration 10 --interval 300 --group \
arm_dsu_0/bus_access/,\
arm_dsu_0/l3d_cache_refill/,\
arm_dsu_0/l3d_cache_wb/ \
-e cpu-cycles
```

이 명령어를 사용하면 다음과 같은 결과를 얻을 수 있다.
```txt
Performance counter statistics:

#         count  event_name                   # count / runtime
  1,552,396,462  arm_dsu_0/bus_access/        # 184.787 M/sec
    706,772,970  arm_dsu_0/l3d_cache_refill/  # 88.452 M/sec
    193,709,487  arm_dsu_0/l3d_cache_wb/      # 22.261 M/sec
 64,708,683,191  cpu-cycles                   # 1.487286 GHz

Total test time: 8.701382 seconds
```

이 결과에는 단위가 count인데, 이 단위는 말그대로 unit이고 더 자세하게 어떤 unit인지는 기기에 따라 다르다.

## 3. 단위 분석

각 단위는 count인데 이 count가 어떤 수인지를 파악해야 한다. 보통 transactions 또는 beats가 unit으로 된다.
Bus는 quad channel에 각 channel의 bus width는 16 bit이다. 이때 1 beat = 4 x 16 bits = 8B가 된다.
그리고 ARM 기기의 Cache Line Size는 64B가 된다.
Transaction의 경우, Cache Line 하나가 전송되려면 총 4번이 일어나야 한다.
Pixel9의 경우 `bus_access`가 beat이었고, `l3d_cache_refill`과 `l3d_cache_wb`가 transaction 단위임을 알 수 있다.

$$
 \textrm{bus\_access} \times \textrm{tractions} \approx \left (  \textrm{l3d\_cache\_refill} + \textrm{l3d\_cache\_wb} \right ) \times \textrm{beats}
$$

$$
184.787MB \times 4 = 739.148MB \approx \left (  84.130M + 22.364M \right ) \times 8 = 851.952MB
$$

이 정도 값이 나온다.
이 정보들을 수집해서 평균값을 내면 된다.
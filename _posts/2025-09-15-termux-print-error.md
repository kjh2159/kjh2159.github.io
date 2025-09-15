---
title: Termux 출력 에러
date: 2025-09-15 20:00:00 +0900
categories: [Android, terminal]
tags: [Android, Mobile, terminal, termux, error]
---

# Termux 출력 에러

## 1. 에러 설명

```shell
[print-1]
         [print-2]
                  ~/$
```

Android termux에서 C/C++ 코드를 짜서 출력을 하다보면, 위와 같이 출력이 밀리는 경우가 확률적으로 발생한다.
또는, 더 심할 경우에 키보드 입력이 되지 않는 경우가 발생하기도 한다.
사실 몇 번 만 사용하는 코드라면, `ctrl + c`로 shell을 초기화하거나 session을 다시 시작해 사용하면 된다.

하지만, 실험을 수십번 이상 반복해야하는 경우, 매우 화가 날 수 있다.

## 2. 원인

원래 CLI(command line interface)나 텍스트에서 줄바꿈은 LF(Line Feed, 개행)과 CR(Carriage Return, 복귀)로 이루어진다.
LF는 커서를 좌우는 고정하고 한 줄 아래로 내리는 개행을 의미하고, CR은 동일 라인에서 커서를 가장 첫 부분으로 옮기는 복귀를 의미한다.

이 문제가 발생하는 이유는 줄바꿈 출력시 CR이 제대로 되지 않아서 발생했기 때문이다.

## 3. 해결 방법

해결 방법은 매우 쉽다. 단순히 CR을 넣어주기만 하면 된다.

```cpp
std::cout << "[print-1]\r" << std::endl;
// or
std::cout << "[print-2]\r\n";
```
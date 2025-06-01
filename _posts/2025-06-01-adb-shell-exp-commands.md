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
```
adb shell settings put system screen_off_timeout 180000
```

180000ms는 30분을 의미한다. 현재 설정을 보려면 `put` 대신 `get`을 쓰면 된다.


> 앞으로 이 글의 명령어는 계속 추가될 예정이다.
{: .prompt-info }
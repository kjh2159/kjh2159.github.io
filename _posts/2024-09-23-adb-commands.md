---
title: adb 명령어 정리 (내가 보려고)
date: 2024-09-23 10:30:00 +0900
categories: [Android, adb]
tags: [adb, android, shell, commands]
---

# adb 명령어 정리 (내가 보려고)

## 들어가며

이제 Android 모바일도 많이 만져야 하기 때문에, 3년 전에 미뤄뒀던 adb 명령어 정리를 하고자 한다.
이 [사이트](https://developer.android.com/tools/releases/platform-tools?hl=ko)에서 다운로드 받을 수 있으며, adb는 **SDK Platform Tools**에 포함되어 같이 관리되고 있다. 
Android 연구/개발할 때 개꿀 따라시인 프로그램 하나 더: [scrcpy](https://github.com/Genymobile/scrcpy)
Android 모바일 개발할 예정이니까, android studio도 깔아주고..


## 환경 세팅...?

사실 할 것도 없다. 일단 당연하게도 adb 쓰려면 ***폰 개발자 모드*** 전환하고, ***USB 디버깅*** 활성화해주자.

## 1. 기본 명령어들 

연결된 기기들 확인
```bash
adb devices
```
결과에 따른 상태들

상태 | 설명
----|----
device| 정상
unauthorized| 연결된 상태지만, 신뢰되지 않아 ADB 명령 수행 불가. 이 컴퓨터를 신뢰하지 않음 버튼을 눌렀을 때 발생 가능
offline| 연결된 상태지만, adb 명령을 받을 수 없는 상태. USB 케이블 또는 Wi-Fi 재연결로 해결 가능.
no permissions| adb가 기기에 접근 권한이 없는 상태. Linux에서 사용자 계정이 접근 권한이 없을 때 발생 가능. `sudo adb devices` 명령어로 관리자 권한 부여 가능.
unknown| 기기 인식 불가. adb 서버 재시작, 또는 USB 연결모드 `파일 전송` 설정시 해결 가능.
recovery| 복구모드로 부팅된 상태. 제한적으로 수신 가능. `adb sideload`로 작업 수행 가능.
bootloader| bootloader 모드. fastboot로 플래싱이나 루팅시 발생 가능. adb는 사용 불가.
sideload| ADB Sideload 모드. sideload 명령 대기 상태임.

무선 네트워크 연결/해제
```bash
adb connect <device_ip_address>
adb disconnect <device_ip_address>
```

Shell 접근
```bash
adb shell
sudo adb shell # 권한 미보유시
```


## 2. 앱 관련 명령어들

APK 파일 설치 및 제거
```bash
adb install <path_to_apk>
adb uninstall <package_name>
```

실행중인 앱의 프로세스확인
```bash
adb shell ps | grep <package_name>
```

앱 강제 종료
```bash
adb shell am force-stop <package_name>
```

앱 데이터 삭제
```bash
adb shell pm clear <package_name>
```

## 3. 로그 및 디버깅 명령어들

로그캣 출력
```bash
adb logcat
```

실행중인 특정 프로세스 로그캣 필터링
```bash
adb logcat | grep <package_name>
```

스택 트레이스 확인, 앱 충돌 정보 확인 가능
```bash
adb logcat *:E
```

## 4. 파일 전송

파일 복사 (PC -> Android)
```bash
adb push <local_file_path> <remote_path_on_device>
```

파일 복사 (Android -> PC)
```bash
adb pull <remote_path_on_device> <local_file_path>
```

## 5. 기타 유용한 명령어들

재부팅
```bash
adb reboot
```

특정 모드로 재부팅
```bash
adb reboot <mode_name>
```

화면 캡쳐
```bash
adb shell screencap /sdcard/screenshot.png
adb pull /sdcard/screenshot.png
```

화면 녹화
```bash
adb shell screenrecord /sdcard/screenrecord.mp4
adb pull /sdcard/screenrecord.mp4
```

기기 정보: 배터리 상태 확인
```bash
adb shell dumpsys battery
```

기기 정보: 시스템 정보 확인
```
adb shell getprop
```

기기 정보: CPU 사용률 확인
```
adb shell top
```
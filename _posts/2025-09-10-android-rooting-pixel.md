---
title: Android Rooting (Google Pixel 9)
date: 2025-09-09 23:00:00 +0900
categories: [Linux, storage]
tags: [Android, Mobile, Rooting, magisk]
---

# Android 휴대폰 루팅하기 (Google Pixel 9)


> 개인용 휴대폰에서는 파일이 손상 또는 휘발될 수 있으니 필히 백업해주시기 바랍니다. \
> 또한, 루팅을 진행하면 공식 수리나 A/S등의 서비스를 받지 못 하므로 평소 사용하는 휴대전화에 루팅을 한다면, 큰 주의를 기울여주시기 바랍니다. \
> 이거 보고도 못 따라하면, 정말 ~~병신~~입니다.
{: .prompt-warning}


## **0. 들어가며**

갑자기 자기 혼자 루팅이 풀려서 갑작스레 다시해야 해서 올리게 되었다... 모르겠다.. 왜 자기 혼자 루팅이 풀리는지.. 아무튼, android sdk platform tools를 [공식 홈페이지](https://developer.android.com/tools/releases/platform-tools)에서 다운로드 해주자. 그리고, 이 폴더에 있는 명령어를 쓰므로, 해당 경로에서 powershell이나 cmd (linux, mac은 terminal)을 열어주자. 또한 휴대폰이 몇 번 초기화가 될 수 있는데 그 때마다 항상 `개발자 옵션 > USB 디버깅 허용`을 해주길 바란다.


## **1. Bootloader Unlocking**

먼저, Bootloader(이하 부트로더)를 먼저 잠금해제해줘야 한다. 이유는 새로운 이미지 파일을 프래쉬해줘야 하기 때문이다. 이를 위해서 OEM을 허용해줘야 한다. `개발자 옵션 > OEM 잠금 해제`와 `USB 디버깅 허용` 옵션들을 허용한다. 데스크탑과 폰을 케이블로 연결하고 **이 데스크탑 항상 신뢰**를 눌러준다. 그리고 아래 명령어를 실행해준다.

```shell
adb devices
adb reboot bootloader
fastboot flashing unlock
```

`adb devices` 명령어 실행시 `unauthorized` 등의 결과가 나오면 안 되니 꼭 잘 확인하자. 그리고, 우측 상단에 ***Do not unlock Bootloader***라는 문구가 뜬다. 그러면, 볼륨 업/다운 버튼을 눌러 ***Unlock Bootloader?***으로 바꿔준다. 그리고 전원 버튼을 누르면 완료다. 그러면, 우측 상단의 문구가 ***Start***로 바뀌는데, 다시 한 번 누르면 폰이 재시작된다.

## **2. Prerequisite**

[공식 Factory Image 다운로드 사이트](https://developers.google.com/android/images)에 해당 휴대폰의 모델 종류와 빌드 번호에 맞는 factory image를 다운로드 받는다(`설정 > 휴대전화 정보 > 빌드 번호` 확인). 그리고 이중으로 .zip 압축이 되어 있는데 모두 풀어주자. 추가로 magisk라는 앱을 다운로드 받아야한다. [magisk github 페이지](https://github.com/topjohnwu/Magisk/releases)에서 .apk 파일을 받아준다. 그리고 그리고 그 파일을 휴대폰 Download 폴더에 옮겨준다. 그리고 휴대폰의 Download에 가서 Magisk를 휴대폰에 설치해준다.

## **3. File Patching**

이제 SamFW에서 다운로드 받아놓은 파일 중에 `init_boot.img` 파일을 찾아야 한다. 다음 예시 경로 폴더를 따라가면 있을 것이다: `tokay-[BUILD.VER]/image-tokay-[BUILD.VER]/init_boot.img`. 그리고 해당 파일을 adb를 사용해 Pixel 기기 내부 저장소로 옮겨준다(파일을 터미널로 드래그 하면 경로 자동으로 뜬다.). 

```shell
adb push /your/path/tokay-[BUILD.VER]/image-tokay-[BUILD.VER]/init_boot.img /sdcard/Download
```

설치된 magisk를 실행하고 `Magisk > Install > Select and Patch a File > init_boot.img 파일 선택`해준다. 그리고 `Let's Go` 버튼을 누르면 뭔가 진행되고 완료되면 patch 파일이 생성된다(휴대폰에서 파일 앱에서 확인 가능). 파일 이름은 `magisk_patched`로 시작한다. 이렇게 생성된 `magisk_patched` 파일을 나의 PC로 빼내자. `magisk_patched` 파일 이름을 확인하기 위해서는 아래 명령어 첫 줄을 통해 확인 가능하다.

```shell
adb shell ls /sdcard/Download/ 
adb pull /sdcard/Download/magisk_patched-[SERIAL.NUM].img /your/path/
```

## **4. 이미지 파일 설치**

휴대폰을 bootloader 모드로 재시작하자.

```shell
adb reboot bootloader
```

그리고, bootloader 모드가 켜지고 **Device state: unlocked**라고 켜져있으면 flash할 준비가 끝났다. 아래 명령어로 flash해주자.

```shell
fastboot flash init_boot /your/path/magisk_patched-[SERIAL.NUM].img
```

위 명령어 실행 결과에 두 가지의 `OKAY`가 뜨면 된다.


## **6. Root Check**

```shell
fastboot reboot
```

이제 `Root Checker Basic` 다운로드 받은 이후에 루팅이 됐는지 확인하면 끝이다. 완.
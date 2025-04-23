---
title: How to Control Brightness in Android Shell
date: 2025-04-21 17:00:00 +0900
categories: [Linux, Android]
tags: [Android, Mobile, Rooting, kernel, brightness]
---

# How to Control Brightness in Android Shell

## 들어가며

안드로이드 셸에서 화면 밝기를 조절은 여러 이유에 의해서 필요하겠지만, 나의 경우 연구를 하면서 디스플레이가 차지하게 되는 Power를 제외하기 위해서 필요했다. 밝기 조절을 하는 방식은 여러가지가 존재하겠지만, 크게 2가지를 소개하고자 한다.


## 1. Hold 버튼 효과
먼저 Shell script를 통해 홀드 버튼 효과를 주어 화면 밝기를 낮출 수 있다. 즉, 홀드 버튼 효과이기 때문에 화면이 꺼진 상태에서는 켜짐을 유의하자. 아래의 명령어를 통해서 화면의 홀드를 누르고, 화면 잠금 해제 효과를 낼 수 있다. 이러한 기능을 활용하기 위해서는 adb tool이 거의 필수적으로 필요하다. 

```bash
# In adb shell
input touchscreen keyevent 26 # click hold button
input touchscreen keyevent 82 # drag to unlock screen
```

하지만, 이 방식을 사용할 때는 몇 가지 주의점을 고려해야 한다.
> 1. 홀드 버튼을 누른 상태에서도 잘 동작하는 application인지 확인해야 한다. \
> 2. `sysfs` 등을 다룰 때는 홀드 버튼에 의해 `sysfs`에 re-event가 발생해 커널 설정이 무의미해질 수 있다.
{: .prompt-warning }

이렇게 크게 2가지의 주의점을 고려하며 실험을 설계하는 것이 좋다. 이것때문에 며칠 날렸다.



## 2. 밝기 값의 자체적 조절
또한, 디스플레이의 밝기 자체만을 조절하는 방법도 있다. 이 방식은 화면의 밝기만을 없애며, 디스플레이에 작성되어야 하는 디스플레이와 GPU 사이의 데이터 흐름은 존재한다. 중요하게도 이 방식은 사용하는 Android 기기가 ***루팅되어 있어야 한다***. 따라서, 루팅이 되어 있지 않다면, 루팅이 필요할 수 있다.

```bash
# In adb shell
su -c "echo 0 > /sys/class/backlight/panel0-backlight/brightness" # turn off the screen brightness
su -c "echo 1023 > /sys/class/backlight/panel0-backlight/brightness" # maximize the screen brightness
```

보통 화면의 밝기 값은 0 ~ 1023의 값을 가질 수 있도록 되어있다. 휴대폰의 GUI 상에서 조절하는 화면 밝기는 내릴 수 있는 밝기의 최저값이 정해져있다. 예를 들어, Pixel9 기준, 최저 값은 184이었다. 하지만, 우리는 이렇게 cli를 통해 그 이하까지도 조절이 가능하다.



## 3. Overall Application in C++
C++에서는 이렇게 쓸 수 있다.

```cpp
/* @kjh2159
 * Execute the bin file on Android shell 
 */ 

#include <stdlib.h>
#include <thread>

int main(){
    // turn off the screen
    system("su -c \"echo 0 > /sys/class/backlight/panel0-backlight/brightness\""); // rooting is required

    std::this_thread::sleep_for(1000); // sleep for 1 sec

    // unlock the screen
    system("input touchscreen keyevent 26"); // turn off screen
    std::this_thread::sleep_for(1000); // sleep for 1 sec
    system("input touchscreen keyevent 26"); // turn on screen
    system("input touchscreen keyevent 82"); // unlock the screen

    return 0;
}

```
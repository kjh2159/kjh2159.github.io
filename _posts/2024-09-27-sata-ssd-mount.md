---
title: Ubuntu에서 SATA SSD Mount하기
date: 2024-09-27 08:00:00 +0900
categories: [Linux, storage]
tags: [Linux, Ubuntu, commands, SSD, SATA, storage]
---

# Ubuntu SATA SSD Mount

## 0. 들어가며

연구를 하면서, 데이터를 저장할 공간이 부족해 급하게 SSD를 추가 연결해야 했다. 원래 PC가 그 이전 연구의 실험 PC를 연구가 끝나고 물려받아 실험을 했던 것이기 때문에 이미 NVMe SSD가 800GiB/1TB 정도 차있었다. 그래서 데이터를 추가 저장해야 할 공간이 필요했고, 급하게 연구실에 돌아다니던 새 SATA SSD를 마운트했다.

내가 사용하는 시스템은 이렇다.
> OS: Ubuntu 20.04 LTS \
> Motherboard: ROG STRIX Z490-F Gaming \
> NVMe 0: Samsung SSD 970 EVO Plus 1TB \
> SATA 0: WDC WDS500G2B0A (3D NAND SATA SSD)
{: .prompt-info}


## 1. 디스크 확인

먼저, 물리적으로 연결되었는지 확인해야 한다. 연결했다는 가정에서 제대로 연결됐는지 확인하는 방법은 2가지가 있다.

#### A. 명령어로 확인하기

아래 명령어를 치고 `Disk /dev/sdx` 형태로 출력되는 섹션이 있는지 확인한다. x는 알파벳 소문자들이다. 예를들어 `Disk /dev/sda`, `Disk /dev/sdb` 등이다. 나는 `sda`이므로 앞으로 `sda`라고 적겠다.

```bash
sudo fdisk -l
```

아래 예시처럼 출력된다면, 잘 연결됐다는 의미다.

```
Disk /dev/sda: 465.78 GiB, 500107862016 bytes, 976773168 sectors
Disk model: WDC  WDS500G2B0A
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

#### B. GUI로 확인하기

명령어로 확인하지 않는 방법은 `설정 > 정보 > 디스크 용량`에서 크기 총합이 맞는지만 확인하면 된다. 나는 1TB가 기존에 있었고, 500GB를 추가했으니 1.5TB라고 적혀져있다. 이렇게 나온다면 잘 인식됐다.


## 2. Partition 생성 (Optional)

파티션을 나눠줘야 하는데, 지금 사용하는 SSD가 완전히 새 제품이므로 파티션이 안 나눠져있을 것이므로 파티션을 나눠준다. 근데 저장 장치로만 사용할 예정이므로 전체를 하나의 파티션으로만 사용하겠다 (sector를 default값들로 양쪽 끝을 선택). 

```bash
sudo fdisk /dev/sda
```

어떤 글이 나올텐데, 그럴 때 아래의 값들을 순서대로 넣어주면 된다.

```shell
n # create a new partition
p # select partition type
1 # for partition number
enter # for first sector
enter # for last sector
w # save edits
```

## 3. 파일 시스템 포맷

이제 파티션을 나눴으면(나눠져있다면), 해당 파티션이 어떤 파일 시스템 형식을 사용하게 할 것인지 정하면 된다. 그냥 대부분 `ext4`을 선택하면 된다. 나는 리눅스이기 때문에 당연한 것이기도 하고..

```bash
sudo mkfs.ext4 /dev/sda1
```

여기서 1은 우리가 partition을 나눌 떄 설정한 partition number이다. 잠시 기다리면 해당 파티션이 `ext4`로 포맷될 것이다. 중요한 파일은 없는지 꼭 확인해보자.


## 4. 마운트

이렇게 됐으면 마운트를 할 준비가 다 되었다. 먼저 마운트를 하고자하는 지점을 생성해주어야 한다. 다시 말해 어디로 접근할 것인지를 그 directory를 지정하고 그 지점에 마운트를 한다. 기본적으로 Ubuntu를 사용하면 `/mnt/`의 working directory가 있을 것이다. 그 곳에 마운트한다.

```bash
sudo mkdir /mnt/ssd
sudo mount /dev/sda1 /mnt/ssd
```

에러가 없다면 성공이다.

## 5. 자동 마운트 설정

Linux에서는 자동으로 부팅되면서 이 ssd 저장장치를 자동으로 마운트 시키지는 않는다. 그래서 부팅시 자동으로 마운트 시킬 수 있도록 설정해줘야 한다. 그렇게 하기 위해 먼저, UUID를 확인해야 한다. 아래의 명령어로 확인해본다.

```bash
sudo blkid /dev/sda1
```

아래와 같은 결과가 출력되는데 여기서 UUID 부분을 복사해둔다.

```
/dev/sda1: UUID="a1b2c3d4e5-f6789-f6789-f6789-a1b2c3d4e5" TYPE="ext4" PARTUUID="0123456-78"
```

`/etc/fstab`에서 부팅시 마운트할 수 있도록 파일을 수정한다. `/etc/fstab`은 파일이므로 이 파일을 수정하기 위해 text editor가 필요한데, 나는 vim이 훨씬 편하므로 vim을 통해 파일을 수정하도록 하겠다. 만약 vim이 설치되어 있지 않다면, vim을 nano로 바꿔서 명령어를 실행시키면 된다.

```bash
sudo vim /etc/fstab/
```

이렇게 실행시켰으면, 아래의 코드 중에 `UUID=` 부분에 본인의 UUID를 적어 넣은 후 `/etc/fstab`의 맨 마지막줄에 추가해 넣으면 된다.

```
UUID=your-uuid /mnt/ssd ext4 defaults 0 2
```


## 6. 파일 권한 설정 (Optional)

일반적으로 파일 탐색기에서 `다른 위치 > 500GB 볼륨`을 접근할 수 있다. 그런데 들어가서 마우스 우클릭을 해보면 `새 폴더`나 이런 기능들이 전부 비활성화되어 있는 것을 볼 수 있다(안 되어 있으면 그냥 쓰면 된다). 나는 이 기능들이 전부 비활성화 되어 있어서 활성화를 시켜줘야 했다. 활성화는 별다른 장치는 없고 해당 볼륨에 대한 권한을 풀어주면 된다. `ctrl + h`를 누르면 `lost+found`라는 파일이 나올텐데, 이 폴더는 잠겨있어야 한다. 이 폴더는 백업용 폴더로 linux가 활용하기 때문에 건들지 않는 편이 좋다. 하지만, 나는 권한만 살짝 만졌다. 이 SSD에 대해서는 전부 권한들 풀어줬고, `lost+found` 폴더에 대해서만 권한을 다시 잠궜다.

```bash
sudo chmod -R 777 /mnt/ssd
sudo chmod -R 700 /mnt/ssd/lost+found
```

이렇게 하면, 해당 SSD에서 새 폴더를 만들거나 기타 동작들을 할 수 있다.

## 7. GUI 설정 (Optional)

추가적으로 나의 경우, 잘 되는지 확인하고자 재부팅했는데 `파일 > 다른 위치 > 500GB 볼륨`이 갑자기 보이지 않는 것을 확인했다. 원래는 잘 보여야 하는데... 하... 아무튼 디스크 유틸 장치를 켜서 해결해줬다.

```
gnome-disks
```

위의 명령어를 입력하면, 새로운 창이 하나 뜬다. 이거 모르겠으면 그냥 앱 목록에서 `디스크` 찾아 들어가면 된다. 그러면, 여기에 500GB짜리 하나가 추가적으로 떠있는데, 여기서 선택한다. 그리고 중간에 톱니바퀴 모양의 `설정`이 있다. `설정 > 마운트 옵션 편집 > 사용자 인터페이스에서 보이기`로 가서 체크해준다. 그리고 다시 재부팅해보면, 문제없이 잘 뜬다. 해결 완.
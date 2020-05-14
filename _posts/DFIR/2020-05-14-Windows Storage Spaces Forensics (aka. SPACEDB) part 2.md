---
title : "Blog #8: Windows Storage Spaces Forensics (aka. SPACEDB) part 2"
category :
  - Windows Storage Spaces
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - Storage Spaces
  - Storage Direct
  - Software RAID
  - SPACEDB
  - 동작 방식
  - 복원 유형
  - 저장 방식
sidebar_main : true
author_profile : true
use_math : true
toc: true
toc_sticky: true
toc_label: "Table of Contents"
header:
  overlay_image : /assets/images/post.jpg
  overlay_filter: 0.5
#published : true
---
윈도우 저장소 공간 포렌식(SPACEDB) part 2

## Windows Storage Spaces Series
- [Blog #7: Windows Storage Spaces Forensics (aka. SPACEDB) part 1](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-1/)

지난 포스트에서 윈도우 저장소 공간에 대한 개괄적 내용을 살펴보았다. 이번에는 저장소 공간을 생성하기 위한 여러 가지 방법, 복원 유형, 데이터 저장 방식을 살펴보도록 한다. 윈도우 서버군에서의 저장소 공간의 일부 내용은 언급을 하나 전체적인 내용은 윈도우 10에 초점을 맞춘다.

## 저장소 공간 생성 (Storage Spaces Creation)
윈도우 저장소 공간은 제어판의 저장소 공간 관리에서 생성 및 확인이 가능하다. Powershell 명령어로도 생성이 가능하지만 가장 쉽게 할 수 있는 방법은 GUI를 이용하여 생성하는 것이다.

<p align="center">
  <img src="https://i.imgur.com/oRO0PqT.png" alt="image"/>
</p>

### 풀을 생성할 디스크 형태 (Disk Type to be member of Pool)
윈도우 저장소 공간을 생성하고 서비스하기 위해서는 물리 디스크를 이용하여 구성을 해야한다. 하지만 간단한 테스트나 실험을 하기 위해서는 가상 디스크 형태인 VHD(Virtual Hard Disk) 또는 VHDX 형태로도 구성이 가능하다.

하지만 VHD로 저장소 풀을 구성했을 때 반드시 단일 디스크 파일 크기가 4GB를 넘어야 정상적으로 풀을 구성할 수 있다. 즉, VHD로 테스트를 할 때 최소 4.1GB로 가상 하드 디스크를 생성하면 저장소 공간을 사용할 수 있는 디스크가 된다.

<p align="center">
  <img src="https://i.imgur.com/2RHC3kn.png" alt="image"/>
</p>

VHD로 풀을 구성할 때 파일시스템을 NTFS로만 구성할 수 있지만 VHDX로 구성하면 ReFS(Resilient FileSystem) 파일시스템도 선택할 수 있게 된다. 참고로 윈도우 서버군의 경우 기본적으로 파일시스템은 ReFS로 설정이 된다.


### 디스크를 풀로 구성할 수 없을 때 (If pool cannot be created)
물리 디스크로 저장소 공간을 구성하고 싶은데 풀이 막상 잘 생성되지 않는 경우가 있다. 오류는 위에서 확인한 "풀을 만들 수 없습니다."라는 동일한 오류 메시지를 보여준다.

Powershell 명령어로 **Get-PhysicalDisk**의 결과를 확인했을 때, 저장소 풀을 구성할 수 있는지 없는지 상태를 확인할 수 있는데 False로 되어있으면 해당 디스크로 풀을 구성할 수 없다.

<p align="center">
  <img src="https://i.imgur.com/d5qL19Z.png" alt="image"/>
</p>

저장소 풀에 참여하려면 기본적으로 할당된 볼륨이 삭제된 상태에서만 가능하다. 따라서 볼륨이 할당된 상태라면 볼륨을 삭제한 다음 아래 Powershell 명령어를 통해 디스크의 상태를 초기화 해야 한다.
<p align="center">
  <img src="https://i.imgur.com/JBPUlhs.png" alt="image"/>
</p>

``` shell
Reset-PhysicalDisk -FriendlyName [Friendly name of Disk]
```

그리고 다시 **Get-PhysicalDisk** 명령어로 상태를 확인 했을 때 <span style="color:red">**True**</span>로 변경된 것을 확인할 수 있다.
<p align="center">
  <img src="https://i.imgur.com/g2sHslz.png" alt="image"/>
</p>

두 개의 디스크가 시리얼 넘버가 동일한 이유는 내가 가지고 있는 하드 도킹 스테이션을 사용하여 2개의 물리 디스크를 연결했을 때 시리얼 넘버가 동일하게 보였다. 만일 시리얼 넘버가 동일하다면 필자와 같은 상황일 수 있으니 참고하면 될 것 같다.


## 복원 유형 (Resiliency Options)
윈도우 저장소 공간에 저장된 데이터를 보호하고 저장소 공간의 논리 볼륨을 보호하기 위해서 4가지 복원유형을 사용할 수 있다.

### 단순 (Simple)
Raid-0(Striping) 또는 JBOD(Just Bunch Of Disks)와 비슷해 보이지만 차이점이 있다.

**1) 디스크의 모든 공간을 사용**  
예를 들어 1TB 디스크와 2TB 디스크를 Raid-0로 묶으면 실제 가용 용량은 2TB(1TB*2)가 되지만 마이크로소프트의 저장소 공간은 3TB를 사용할 수 있다.  
참고로 Synology NAS의 SHR(Synology Hybrid RAID) 기능의 경우 Raid-0로 구성해도 거의 3TB 용량을 사용할 수 있다.

**2) 순차적이지 않은 기록 방식**  
 JBOD처럼 디스크에 순차적으로 저장되는 것과 다르게 256MB 크기의 <span style="color:red">***slab***</span> 단위로 데이터를 저장하며 풀을 구성하는 디스크에 데이터를 각각 나눠서 저장된다.

단순(Simple) 복원 유형은 데이터의 빠른 입출력 속도를 위해서 사용하는 옵션이며 자주 또는 주기적으로 생성되고 삭제되는 임시 파일(동영상 렌더링 파일 등)을 저장하는 공간으로 사용될 수 있다.  
해당 복원 유형의 단점으로는 드라이브의 오류로부터 보호받지 못한다는 단점이 있다.

### 양방향 미러 (Two-way mirror)
원본 데이터를 한 개 더 복사하여 디스크에 2개에 각각 저장하는 방식이다. 즉 동일한 데이터가 2개가 있다는 뜻이다. 만일 디스크 3개로 양방향 미러를 사용한다면 디스크 3개 중 내부 로직에 따라 서로 다른 디스크 2개를 선정하고 동일한 데이터 2개를 slab 단위로 나눠 저장하는 방식이다.

### 3방향 미러 (Three-way mirror)
3방향 미러는 양방향 미러와 비슷한 방식이나 동일한 데이터를 3개를 만들어 디스크 3개로 나눠서 저장하는 방식이며 한 번에 2개의 디스크의 장애가 발생해도 데이터를 보호할 수 있다.

### 패리티 (Single/Dual Parity)
Window Server 2016부터 Single Parity, Dual Parity 선택 가능하며 기본적으로 Single parity 방식은 RAID-5와 비슷하며 Dual parity는 RAID-6와 비슷하다.
Single Parity는 단 1개의 디스크의 장애가 발생할 때만 복원이 가능하기 때문에 MS에서는 굳이 싱글 패리티를 사용하기보단 3방향 미러를 사용하기를 권장한다.


## 데이터 저장방식 (How data is stored)
저장소 공간 논리 볼륨에 데이터가 저장될 때 256MB 크기의 **slab**이라는 블록 단위로 데이터를 저장한다.  
예를 들어 저장소 공간의 크기가 1TB일 경우 총 4,000개의 slab을 구성할 수 있다. 그리고 복원 유형에 따라 데이터를 보호한다.

양방향 미러 방식을 예를 들면, 저장소 공간을 slab 단위로 나누고 각 slab마다 2개의 데이터를 만든 뒤(복제) 서로 다른 2개의 물리 디스크로 저장이 된다. 즉, 동일한 데이터가 2개이니 2개의 다른 디스크로 저장되는 것이다.

그래야 1개의 디스크에 고장이 발생해도 나머지 디스크에서 데이터 복원이 가능하다는 의미이다.
<p align="center">
  <img src="https://i.imgur.com/5HZRBB2.png" alt="image"/>
</p>

또한 Pool metadata를 이용하여 각 디스크 드라이브의 slab 관리하며 특정 디스크 1개에 데이터가 집중되지 않고 골고루 저장되도록 설계되어 있다.

<p align="center">
  <img src="https://i.imgur.com/QJ6tKNW.png" alt="image"/>
</p>

조금 더 상세한 내용인 디스크 장애가 발생되거나, 용량 부족 시 동작 방식 등 추가적인 내용에 대해서는 논문을 참고하길 바란다.

다음 포스트에서는 단일 디스크 및 다중 디스크를 이용한 저장소 공간의 획득했을 때 발생할 수 있는 이슈를 알아보고 이를 해결할 수 있는 내용으로 글을 작성할 예정이다.


## Reference
- <https://docs.microsoft.com/ko-kr/windows-server/storage/storage-spaces/storage-spaces-states>
- <https://support.microsoft.com/en-us/help/12438/windows-10-storage-spaces>
- <https://docs.microsoft.com/en-us/windows-server/storage/storage-spaces/storage-spaces-fault-tolerance>
- <https://techcommunity.microsoft.com/t5/storage-at-microsoft/deep-dive-the-storage-pool-in-storage-spaces-direct/ba-p/425959>
- <https://kamilake.com/246>
- <http://www.storage-spaces-recovery.com/library/storage-spaces-fundamentals.aspx>

## Copyright (CC BY-NC 2.0 KR)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
---
title : "Blog #12: Leveraging WinFE(Forensic Environment) part 1"
category :
  - Portable OS
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - WinPE
  - WinFE
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
WinFE 활용 방법 part 1

현재 윈도우 운영체제는 종류가 매우 다양하다. 종류가 다양한 만큼 사용자 또는 조직의 관점에서 필요한 운영체제를 선택하여 사용할 수 있는데 운영체제의 GUI가 거의 비슷하여 사용함에 편리함을 제공한다.
 
WinPE 운영체제를 커스텀하게 구성하면 내가 원하는 도구를 넣을 수 있는 등 유용하게 사용할 수 있다. 
특히 압수 현장에서 압수 대상 시스템을 탐색할 때 사용도 가능한데, 이 모든 것들은 디스크의 쓰기방지를 한 상태에서 이뤄져야 할 것이다.

WinPE에서는 기본적으로 쓰기방지를 보장하지 않고 사용자가 직접 쓰기방지를 위한 설정을 하는 것도 쉽지 않다.

이번 포스트 시리즈에서는 제한된 서비스를 제공하는 경량 운영체제인 WinPE 및 WinFE를 소개하고 포렌식 관점에서 WinFE가 어떻게 사용될 수 있는지 확인할 예정이다.

## Portable OS
부팅 가능한 포터블 OS를 사용하는 일은 매우 다양하다. 대표적으로 윈도우 암호를 알 수 없는데 PC의 내부 파일을 탐색하고 복사할 때 사용할 수 있고 공용 PC에서도 사용할 수 있다.

USB로 OS 부팅을 하려면 가장 먼저 USB를 부팅 가능한 형태로 제작한 다음 USB로 부팅을 할 수 있는데, 부팅이 완료됐다면 탐색 대상인 디스크를 마운트 해서(자동 마운트 가능) 파일 탐색과 복사를 수행할 수 있다. 

지금까지 내가 사용해 본 부팅 가능한 OS는 KaliLinux, Ubuntu, WinPE 정도였다. 포렌식적으로 이 방법을 통해 대상 디스크의 이미지를 획득할 때도 종종 사용한다.

## WinPE (Windows Preinstallation Environment)
윈도우 PE는 일반적인 운영체제(주 운영체제)에 비해 제한적인 서비스 기능을 제공하여 최소한으로 기능으로 동작할 수 있게 하는 경량판 운영체제이다.

**Windows PE (Preinstallation Environment)**
<p align="center">
  <img src="https://i.imgur.com/WiQowFa.png" alt="image"/>
</p>

주 운영체제 설치 이전에 설치 동작을 위한 간이형 운영체제로 사용할 수 있어 데이터 드라이브를 손쉽게 탐색할 때도 사용할 수 있고 윈도우 배포 서비스(Windows DS), 문제 해결 및 복원을 위한 운영체제(Windows RE) 등 특정 목적을 위해 사용된다.

**Windows RE (Reocvery Environment)**
<p align="center">
  <img src="https://i.imgur.com/AiZJWk5.png" alt="image"/>
</p>

### WinPE의 한계점
WinPE의 가장 큰 단점은 일부 사용자들이 WinPE로 데스크탑 환경을 구성하는 경우가 있다. 그렇게 되면 정식 배포판 운영체제의 라이선스 없이 사용할 수 있기 때문에 MS社는 본래 목적(윈도우 설치, 복구 등)으로 사용하든 불법으로 사용하든 WinPE 사용시간을 72시간으로 제한을 두고 있다. 즉 72시간마다 재부팅이 되어 WinPE의 장기간 사용을 일부 제한하고 있다.

<p align="center">
  <img src="https://i.imgur.com/96V8sft.png" alt="image"/>
</p>

### WinPE 사용 이유
이와 같이 제한사항에도 불구하고 WinPE를 사용하는 가장 큰 장점은 가벼운 운영체제에 사용자가 원하는 도구를 넣을 수도 있으며, USB에 운영체제를 담아 업무환경이 아닌 곳에서도 자유롭게 나만의 OS를 사용할 수 있다는 점이다. 

특히 PC방 등과 같은 곳의 공용으로 사용하는 PC는 보안에 취약할 가능성이 높기 때문에 USB 부팅을 정책적으로 막아놓지 않았다면 WinPE로 직접 부팅하여 사용할 수 있다.

하지만 글 서두에서 언급했듯이 쓰기방지가 담보된 상태에서 이미지를 획득해야 된다는 것이다.

### Portable OS의 포렌식 관점
WinPE와 같이 포터블 OS를 이용하면 컴퓨터 등 저장 기기의 탐색 관점에서 직접적인 압수 대상물 접근을 피할 수 있고, 손쉬운 GUI 환경을 이용하여 탐색을 할 수 있다.

또한 저장 매체의 획득 관점에서도 좋은 도구로 사용될 수 있는데, 예로 압수 현장에서 노트북 등 저장기기에 포함된 HDD, SSD 매체를 이미징 한다고 가정했을 때 기기의 뒷면 또는 앞면을 열어 저장매체를 굳이 탈거하는 수고를 줄일 수 있다.

특히 기기의 접착제 실링, 육각 나사로 고정된 기기와 같이 현장에서 기기의 분해가 불가능한 상태에서는 Portable OS로 부팅한 뒤 포렌식 소프트웨어 도구를 통해 부득이하게 획득하는 방법밖에 없다.

이는 현장뿐 아니라 분석실 내에서도 파손이 우려되는 일체형(All-in-one) PC, 저용량 eMMC 메모리가 장착된 넷북 등 기기에서도 증거물의 운영체제를 부팅하는 대신 Portable OS를 이용하여 획득할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/Gq0MKac.png" alt="image"/>
</p>
<p align="center"><b>[ Asus Chromebook CB3-532-C42P (e-MMC 16GB) ]</b></p>

그런데 이렇게 포터블 OS를 이용할 때 주의해야 할 점이 하나 있다. 확실한 저장매체의 개수 검증이 필요하다는 것인데 만일 현장에서 사무실로 돌아왔는데 획득해온 이미징 파일에서 윈도우 운영체제가 보이지 않고 데이터만 잔득 보인다면 기기 내 숨어있는 또 다른 저장매체를 발견하지 못했을 가능성이 크다.

실제로 의뢰물 중에 PC를 이미징 해왔다고 하는데 데이터 저장용으로 사용하는 HDD만 이미징을 획득해온 경우가 종종 발생하였다. 요즘은 칩으로 된 형태의 M.2, NVMe SSD나 슬롯 형태의 SSD도 있으니 자세히 눈여겨봐야 한다.

아마 부분 때문에 Portable OS만 사용하기보다 직접 기기를 개봉하고 매체를 탈거하면서 직접 눈으로 살피는 과정이 필요한 것 같다.

## Portable OS 쓰기방지 이슈
WinPE의 단점으로는 기본적으로 디스크의 쓰기방지를 보장하지 않는다는 점이다. 윈도우 운영체제는 부팅이 되면 기기에 연결된 모든 디스크를 자동으로 마운트 한다는 특징이 있다. 그리고 운영체제에서 인식하지 못하거나 파일 시스템을 로드하기 위한 드라이버가 설치되어 있지 않은 경우에는 마운트 하지 못한다.

즉 압수 대상인 윈도우 PC에서 WinPE로 부팅을 하게 되면 모든 디스크가 자동으로 마운트가 되어 사용 준비 상태가 되는 것이다. 이 과정에서 쓰기 작업이 일어나는데 아래의 테스트 결과를 살펴보자.

## Test Scenario & Results
현재 일반 사용자단에서 가장 활발히 배포되고 있는 커스텀하게 제작된 Windows 10 PE Lite 버전을 이용하여 노트북 획득 테스트를 진행하였다.

테스트에 사용한 WinPE는 WinPE에는 포렌식 도구를 몇 개 넣고 개인적으로 재 빌드를 한 상태이다. USB로 부팅한 뒤 모습을 살펴보면 기본적으로 탑재되어 있는 도구가 이미 많이 있다.

<p align="center">
  <img src="https://i.imgur.com/VIZIgBz.png" alt="image"/>
</p>

그리고 탐색기를 실행하여 디렉터리 구조를 살펴보면 **C:\\** 드라이브가 압수 대상 저장매체이고 **E:\\** 및 **X:\\** 드라이브는 WinPE 영역이다.

<p align="center">
  <img src="https://i.imgur.com/XGtfUSa.png" alt="image"/>
</p>

여기서 중요한 건 노트북의 저장매체인 디스크는 이미 마운트 되어 있다는 점이다.
현재 이 상태에서 도구를 통해 어느 부분에 쓰기 작업이 수행됐는지를 살펴보도록 하자.

<p align="center">
  <img src="https://i.imgur.com/UPUBIAK.png" alt="image"/>
</p>

Winhex 도구를 사용해서 바로 C:\\를 로드시키고 Modified 시간을 잘 살펴보면 $RECYCLE.BIN 폴더에 무언가 수정이 일어난 것을 확인할 수 있다.
> 참고로 본 테스트를 수행한 일자는 2020. 3. 18. ~ 3. 19.이다.

<p align="center">
  <img src="https://i.imgur.com/c2GRKm2.png" alt="image"/>
</p>

바로 내부를 살펴보자. <span style="color:red">**S-1-5-18**</span> 폴더가 생성되었다. 윈도우는 기본적으로 저장매체가 마운트 될 때 해당 저장 매체의 휴지통 폴더에 운영체제의 SID(Security Identifier, 보안 식별자) 값을 남긴다.

<p align="center">
  <img src="https://i.imgur.com/zq7iJEH.png" alt="image"/>
</p>

정리하면 획득 대상 노트북에서 WinPE 운영체제로 부팅을 했고, 노트북의 HDD(C:\\)가 OS에 자동으로 마운트 되면서 C:\\에 WinPE의 SID값을 폴더 형태로 쓰기 작업이 수행된 것이다.

SID 값은 포렌식적으로 의미 있는 아티팩트인데 레지스트리와의 연계분석을 통해 사용자의 계정명을 알 수 있기도 하고, 마운트가 언제 됐는지도 시간 값을 통해 확인할 수 있다.

또한 도구를 다루다 보면 생각지도 못한 곳에서 쓰기 작업이 발생할 소지가 있는데, 아래와 같이 도구(winhex)의 설정 옵션에 임시 폴더 설정이 증거물의 드라이브로 구성이 되어있는 상태에서 도구를 사용하게 되면 추가적으로 쓰기 작업이 발생할 수 있다.

아래 WinHex의 옵션을 살펴보자.  
만일 도구를 사용했는데 기본 Temp 파일 저장 경로, 케이스 저장 경로 등 경로가 획득 대상 디스크로 저장되어 있는 경우 도구를 이용하는 순간 의도치 않게 원본 매체에 쓰기를 할 수 있다.
<p align="center">
  <img src="https://i.imgur.com/zecXiau.png" alt="image"/>
</p>

<p align="center">
  <img src="https://i.imgur.com/zE42X2C.png" alt="image"/>
</p>

## Wrap-up
이번 포스트에는 기본적으로 WinPE를 사용했을 때 발생할 수 있는 문제점에 대해 실험을 통해 살펴봤다. WinPE를 활용할 때는 쓰기 방지 작업을 한 뒤에 작업을 진행하고 항상 주의를 기울여야 한다.

그런데 사실 이렇게 주의를 기울이고 쓰기 방지 작업을 해야 하는 등 할 일도 많고 조치를 취하더라도 뭔가 마음속에서 좀 걸리는 게 있는 것 같다. 조금 더 쉽고 안전한 방법으로 현장에서 사용할 수 있는 방법이 없을까?

다음 포스트에서는 WinFE에 대한 소개, 설명, 사용법에 대해서 알아볼 예정이다.


## Reference
- <https://jsb000.tistory.com/778>
- <https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-recovery-environment--windows-re--technical-reference>
- <https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-7/dd799308(v=ws.10)?redirectedfrom=MSDN>
- <https://forums.tomshardware.com/threads/can-i-upgrade-the-emmc-on-this-laptop-asus-chromebook-cb3-532-c42p.3465592/>

## Copyright (CC BY-NC 2.0 KR)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
---
title : "Blog #9: Windows Storage Spaces Forensics (aka. SPACEDB) part 3"
category :
  - Windows Storage Spaces
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - Storage Spaces
  - Storage Direct
  - Software RAID
  - SPACEDB
  - X-Ways Forensics 19.9
  - EnCase 8.11
  - Magnet AXIOM 4.01
  - FTK Imager 4.3.0.18
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
윈도우 저장소 공간 포렌식(SPACEDB) part 3

## Windows Storage Spaces Series
- [Blog #7: Windows Storage Spaces Forensics (aka. SPACEDB) part 1](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-1/)
- [Blog #8: Windows Storage Spaces Forensics (aka. SPACEDB) part 2](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-2/)
- [Blog #9: Windows Storage Spaces Forensics (aka. SPACEDB) part 3](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-3/)


이번 포스트에서는 단일 디스크 및 다중 디스크를 이용한 저장소 공간을 획득했을 때 발생할 수 있는 이슈를 확인하고 이를 해결할 수 있는 방법을 살펴보도록 한다.


## 단일 디스크 사용 시 이슈
저장소 공간에서의 단일 디스크 사용이라고 함은 단순(복원 없음, Simple) 복원 옵션을 설정하고 빠른 입출력 속도를 낼 수 있는 저장소 공간을 의미한다.

상황에 따라 다르지만 기본적으로 포렌식 실무에서 디지털 증거를 획득할 때, 저장매체를 장치에서 탈거한 뒤 이미징 장비 또는 소프트웨어를 이용하여 이미지 파일로 획득한다.

마찬가지로 저장소 공간에 사용한 단일 디스크를 획득하여 이미지를 도구에서 로드 하였을 때 파일 시스템이 정상적으로 해석되지 않는다는 문제점이 있다.

1TB의 단일 물리 디스크를 이용하여 저장소 공간을 구성하면 '저장소 풀'이라는 이름의 볼륨과 'SPACEDB' 시그니처가 확인되나, 파일 및 폴더 구조는 아무것도 확인되지 않는다. 마치 암호화된 파일 시스템을 해석하지 못할 때와 같은 증상이다

특히 윈도우 7 PC에서는 쓰기방지장치가 연결된 원본 디스크 자체를 PC에 직접 연결하여도 동일한 증상이 나타난다.

아래 사진은 각 도구마다 어떻게 보여주는지 스크린 캡쳐 화면이다.

**FTK Imager**
<p align="center">
  <img src="https://i.imgur.com/BrASqGy.png" alt="image"/>
</p>

**EnCase**
<p align="center">
  <img src="https://i.imgur.com/9qWybW3.png" alt="image"/>
</p>

**X-Ways Forensics**
<p align="center">
  <img src="https://i.imgur.com/ACU4Y07.png" alt="image"/>
</p>

**Magnet AXIOM**
<p align="center">
  <img src="https://i.imgur.com/5wdaLij.png" alt="image"/>
</p>


허무하게도 윈도우 10 운영체제에서는 원본 물리 디스크를 연결하면 PC에 볼륨으로 자동 마운트되며 파일 및 폴더 구조를 볼 수 있다.

저장소 공간 기술은 윈도우 8, 윈도우 서버 2012부터 나왔던 기술이기 때문에 윈도우 7에서 저장소 공간을 해석해주는 소프트웨어가 있으면 가능할지 모르나 기본적으로 윈도우 7 에서는 물리 디스크를 직접 로드해도 저장소 공간을 해석하지 못하는 이슈가 있다. 따라서 논리 볼륨을 구성하는 NTFS 파일 시스템 역시 파싱하지 못하는 것이다.

즉 이슈를 정리하면 다음과 같다.
1. 윈도우 7, 10 모두 이미징 장비에서 획득한 저장소 공간 이미지 파일 해석불가
2. 윈도우 7에서는 물리디스크를 직접 연결해도 파일시스템 해석불가


### 획득 방안
이슈의 해결 방안은 생각보다 단순하다. 내가 확인한 2가지 획득 방안은 다음과 같다.

**1. 윈도우 10에서 직접 이미지 획득하는 방법**

윈도우 10에서는 저장소 공간을 해석할 수 있기 때문에 쓰기방지장치에 연결된 <span style="color:red">**물리디스크를 직접 PC에 연결하여 이미지를 획득**</span>하는 방법이다.
마운트 된 논리 볼륨을 대상으로 증거 이미지를 생성하면 분석할 수 있는 형태가 된다.

<p align="center">
  <img src="https://i.imgur.com/59YGLQ6.png" alt="image"/>
</p>

위 그림에서는 PHYCALDRIVE3가 저장소 공간의 물리 영역이고, **PHYSICALDRIVE4**가 해석된 가상 볼륨이다. 따라서 후자를 이미징하면 된다.

윈도우 10에서 저장소 공간을 획득했다면, 이미 해석된 저장소 공간 볼륨을 획득했기 때문에 이미지를 가지고 윈도우 7에서도 정상적으로 분석을 할 수 있게 된다.

**2. 이미 생성한 이미지(e01) 파일을 사용하는 방법**

만일 획득 절차가 종료됐고 피압수자에게 원본 증거물을 이미 반환한 경우, 원본이 그대로 보존되고 있는지에 대한 보장도 없으며 법적 절차에 따라 다시 원본을 획득해야 할 것이다.

이와 같이 다시 획득할 수 없는 상황이라면 또 다른 방법이 있다.  
대안으로는 기존에 이미징 장비로부터 <span style="color:red">**획득된 이미지 파일(.e01) 자체를 마운트**</span>하면 저장소 공간 볼륨이 운영체제에 자동 마운트된다. 그러나 중요한 것은 e01 파일 마운트 시 저장소 공간을 해석할 수 있는 윈도우 8 이상 운영체제에서 진행해야 한다.

필자가 최근에 최신 버전의 EnCase, X-Ways Forensics 등 도구에서 마운트 기능을 이용하여 다시 테스트한 결과 [Arsenal Image Mounter](https://arsenalrecon.com/downloads/) 도구를 통해 마운트를 했을 때 가장 적절한 처리 결과를 보여줬다.
Arsenal Image Mounter 도구로 마운트를 했을 때, 실제로 물리 디스크를 연결한 것처럼 내 PC에 드라이브가 정상적으로 생겼고 파일 탐색도 가능하였다.

<p align="center">
  <img src="https://i.imgur.com/cFQC3Lo.png" alt="image"/>
</p>

<p align="center">
  <img src="https://i.imgur.com/UjfAtEN.png" alt="image"/>
</p>

<p align="center">
  <img src="https://i.imgur.com/G4g1jv5.png" alt="image"/>
</p>

다른 도구들의 경우 내 PC에 드라이브가 정상적으로 마운트 되지는 않았으나 도구 내에서 파일시스템은 정상적으로 파싱되어 저장소 공간에 저장된 파일을 볼 수 있었다.

만일 피의자가 다수의 물리 디스크로 저장소 공간 풀을 구성해서 사용했다면 포렌식적으로 고려해야 할 사항이 어떤 것들이 있을까?


## 다중 디스크 사용 시 고려사항
만일 피의자가 다수의 물리 디스크로 풀을 구성해서 사용했다면, 여러 디스크 중 1개를 윈도우 10에 연결해도 자동 마운트가 되지 않았을 뿐 아니라 파일 시스템 해석조차 못할 것이다.  
그 이유는 다수의 디스크가 한 개의 논리 볼륨을 이루어 구성이 되기 때문이다.

### 획득 방안
다중 디스크를 사용했다면 획득하는 방법은 단일 디스크와 크게 다르지 않다.

예를 들어 양방향 미러로 옵션의 3개의 하드디스크로 구성된 경우, 3개의 디스크를 모두 획득할 PC(Win10)에 연결하면 저장소 공간이 자동 마운트 된다. 

마운트 된 상태에서 도구 FTK Imager를 통해 드라이브를 확인하면 10GB VHD 3개(PHYSICALDRIVE3 ~ 5)가 확인되며 자동 마운트 된 저장소 공간(Microsoft Storage Space Device, 12GB)인 가상 볼륨도 확인된다.

<p align="center">
  <img src="https://i.imgur.com/SQOAyfI.png" alt="image"/>
</p>

<p align="center">
  <img src="https://i.imgur.com/liCwUAv.png" alt="image"/>
</p>

여기서 획득할 대상은 저장소 공간 볼륨(PHYSICALDRIVE6, 12GB)이며 해당 볼륨을 대상으로 복제 이미지를 생성하면 된다.

### 논리 볼륨 구성 확인 팁
만일 논리 볼륨을 이루는 물리 디스크의 개수를 모르는 상태에서 디스크가 몇 개로 볼륨을 구성하는지, 제대로 획득을 한 것인지 확인이 되지 않을 경우에 작은 팁이 하나 있다.
윈도우 저장소 공간은 디스크가 하나라도 정상적이 아닐 경우 메시지를 띄워 사용자에게 알려주는 기능이 있다.

만일 디스크 3개 중 1개만 PC에 연결한 경우 <span style="color:red">**문제 파악을 위해 저장소 공간 확인**</span> 이라는 윈도우 토스트 메시지가 현출된다.

<p align="center">
  <img src="https://i.imgur.com/dE2afjn.png" alt="image"/>
</p>

위와 같은 메시지를 확인했다면 저장소 공간을 구성하기 위해 필요한 디스크가 더 있다는 의미이니 나머지 디스크들을 더 찾아서 볼륨 구성을 완성해야 한다. 이는 PC에 디스크를 직접 연결하는 방식이 아닌 이미 획득된 이미지 파일을 마운트 할 때도 동일하게 적용된다.


## 포렌식적 고려사항 요약 (Wrap-up)
지금까지 다뤘던 내용을 요약해보자.

### 첫 번째,
소프트웨어 레이드인 저장소 공간은 윈도우 8, 윈도우 서버 2012부터 나왔던 기술이다.   
따라서 현재 많이 사용되는 윈도우 10 PC에 직접 연결하면 바로 저장소 공간 가상 볼륨이 마운트 되나 윈도우 7에서는 인식하지 못하여 파일 시스템을 해석하지 못한다.

### 두 번째,
하드웨어 장비 및 소프트웨어로 원본 디스크 전체를 이미징한 복제이미지(e01 등) 파일은 운영체제에 관계없이 포렌식 도구에서 해석이 불가하다. (EnCase 8.11, X-Ways Forensics 19.9, Magnet Axiom 4.01 모두 해석 불가)   
단, 윈도우 10에서 저장소 공간 가상 볼륨을 이미징한 경우에는 이미 볼륨이 해석된 상태로 획득된 상태이기 때문에 포렌식 도구에서 해석이 가능하다.

### 세 번째,
다수의 디스크로 저장소 공간을 구성하였다면 다수의 디스크를 운영체제에 모두 연결하면 저장소 공간 가상 볼륨이 마운트된다.   
만일 여러 개의 디스크 중 모든 디스크가 연결되지 않은 경우 "문제 파악을 위해 저장소 공간 확인"이라는 메시지가 현출되며 이를 토대로 저장소 공간을 이루기 위한 추가적인 디스크 여부를 파악할 수 있다.

조금 더 쉽게 접근하자면
PC에서 Bitlocker로 암호화된 디스크를 비밀번호를 입력하여 암호화를 해지하더라도 그 상태에서 Physical 획득하면 암호화된 상태 그 자체로는 해석이 안되고 비밀번호가 필요하나, Logical Volume을 획득하면 해석이 되는 것과 동일하다고 보면 된다.

### 기타 사항
지난 2020. 1. 14.부로 윈도우 7에 대한 지원은 공식적으로 종료되었다. 현재 많은 유저가 윈도우 10을 사용하고 있으나 귀찮거나 어떤 사유로 인해 넘어가지 못한 유저들이 아직 있을 것이다.

앞으로 윈도우 7 유저가 점점 줄어들면서 저장소 공간 자체 해석을 못하는 이슈는 줄어들 수 있으나, 전통적인 방법처럼 이미징 장비에서 획득했을 때 현재 최신 버전의 도구에서 해석을 못하는 이슈는 유효하다.

## 참고 사항 (Note)
최근에 도구들을 최신 버전으로 업데이트 하였고, 이 글은 Falcon Neo 3.1으로 획득한 저장소 공간(Simple 복원 옵션) E01 이미지 파일을 대상으로 다시 한번 테스트를 수행하여 결과를 확인한 뒤 작성한 글이다.

1. 저장소 공간 구성: 단순(Simple) 복원 옵션으로 구성된 250GB HDD 1개
2. 확인한 도구: EnCase 8.11, X-Ways Forensics 19.9, Magnet AXIOM 4.01, FTK Imager 4.3.0.18


## Reference
- <https://www.microsoft.com/ko-kr/windows/windows-7-end-of-life-support-information>

## Copyright (CC BY-NC 2.0 KR)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
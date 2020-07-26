---
title : "Blog #10: Windows Storage Spaces Forensics (aka. SPACEDB) part 4"
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
  - EnCase 20.2
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
윈도우 저장소 공간 포렌식(SPACEDB) part 4

## Windows Storage Spaces Series
- [Blog #7: Windows Storage Spaces Forensics (aka. SPACEDB) part 1](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-1/)
- [Blog #8: Windows Storage Spaces Forensics (aka. SPACEDB) part 2](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-2/)
- [Blog #9: Windows Storage Spaces Forensics (aka. SPACEDB) part 3](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-3/)
- [Blog #10: Windows Storage Spaces Forensics (aka. SPACEDB) part 4](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-4/)
- [Blog #11: Windows Storage Spaces Forensics (aka. SPACEDB) part 5](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-5/)

이번 포스트에서는 다중 디스크를 이용하여 저장소 공간을 구성했을 때, 단 1개의 디스크로 복원율을 높여보고 어느 정도 복원이 가능한지 살펴보도록 한다.

## 실험 구성
실험은 3개의 10GiB의 가상 디스크(VHD)를 양방향 미러 옵션으로 저장소 공간을 구성하였으며, 파일 시스템은 NTFS로 설정하여 테스트를 진행하였다. 

## 카빙에 의존한 복원

### 파티션 검색 기능 (Search Partitions)
카빙 방법으로 복원을 하기 전에 파티션이 검색되는지 확인할 필요가 있다. 파티션 검색은 EnCase에서는 **Partition Finder**로, X-Ways Forensics에서는 **Scan For Lost Partitions**로 수행할 수 있다.

파티션 검색 기능은 이전에 존재했던 파티션들을 찾아주는 역할을 하며 기본적으로는 MBR, 파티션 테이블을 검색하고 FAT, NTFS 등 파일시스템의 부트 영역(VBR)의 시그니처 기반으로 탐색하는 기능이다.

**EnCase > Partition Finder**
<p align="center">
  <img src="https://i.imgur.com/CZaNNZJ.png" alt="image"/>
</p>

**X-Ways Forensics > Scan For Lost Partitions**
<p align="center">
  <img src="https://i.imgur.com/2LmCmuJ.png" alt="image"/>
</p>

테스트용 디스크를 대상으로 검색했더니 다행스럽게(?) 나오는 건 없다. 뒤에서 다시 언급할 것이지만, 아마 저장소 공간의 파일시스템에서 사용하는 섹터 크기, 클러스터 내 섹터 개수가 일반적인 NTFS 파일시스템과 달라 검색을 못한 것이라고 생각된다.

<p align="center">
  <img src="https://i.imgur.com/k3u04v4.png" alt="image"/>
</p>

### 파일 카빙 (File Carving)
이번에는 카빙을 시도해보자. 카빙 옵션에 따라 복원되는 양이 다를 수 있어 최대한 모든 옵션을 선택하고 바이트 레벨로 카빙을 수행하였다.

<p align="center">
  <img src="https://i.imgur.com/9O9mPKy.png" alt="image"/>
</p>

카빙으로 파일을 복원해 냈지만 뭔가 좀 아쉬운 부분이 있다.
카빙에 의존하여 복원을 수행하면 카빙은 정상적으로 수행되어서 복원된 파일을 확인할 수 있으나, 파일시스템 정보는 가져오지 못해 파일명, 데이터에 대한 타임스탬프 등 가치 있는 정보를 확인할 수 없다.

그리고 카빙 된 파일은 파일시스템에 기반하여 복원된 파일보다 정확성이 떨어지는데 이는 원본 파일과 대비해서 해시값이 다른 것을 토대로 정확성을 판단하였다.

만일 카빙으로 복원된 파일이 아동 음란물일 경우, 소지 자체가 위법하므로 파일 존재 그 자체가 혐의 구증에 있어 의미 있는 데이터가 될 수 있으나 일반적인 경우에 메타데이터 없이 카빙으로만 분석하는 것은 한계가 존재한다.

도구에 따라 카빙 된 파일에 대해 자동으로 이름으로 이름을 부여하는 기능을 가지고 있는 도구도 있는데, 그림에 보이는 Carved_0000XX (Adobe embedded 88.70~).jpg와 같은 이름은 X-Ways Forensics 도구의 "Intelligent Naming" 기능이라고 해서 카빙 시 파일 내부의 EXIF 등 메타데이터를 이용해서 카빙 된 파일에 대해서도 이름을 부여해 카빙의 단점을 일부 보완해 주는 역할을 하기도 한다.

## 파일 시스템을 이용한 복원

### 파일 시스템 기본 분석
테스트 환경의 저장소 공간 내 파일시스템은 NTFS를 사용하기에 NTFS의 부트섹터(VBR, Volume Boot Record)가 있을 가능성이 크다.

기본적으로 VBR이 있다면 파일시스템 구조가 그대로 유지될 가능성이 있다는 뜻이고, 결론적으로 VBR을 바탕으로 NTFS 파일시스템을 자체 해석해도 된다는 의미이다.

3개의 디스크 중 2개의 디스크에서는 VBR이 확인이 되었고 나머지 한 개의 디스크에서는 확인되지 않았다. 확인된 VBR의 일부 구조는 다음과 같다.

<p align="center">
  <img src="https://i.imgur.com/N8F7yKs.png" alt="image"/>
</p>

NTFS 파일시스템 특성상 $MFT를 해석하면 폴더의 트리구조 및 메타데이터를 정상적으로 파싱할 수 있는데 VBR 기준으로 특정 오프셋의 값 몇 개가 맞으면 이를 해석할 수 있다. $MFT 해석을 위한 중요한 정보는 3가지이다.
1. 섹터 크기 (현재 4,096 Bytes)
2. 클러스터 내 섹터 수 (현재 1개)
3. $MFT 엔트리까지의 거리 (클러스터 넘버 기준, 현재 786,432 CL)

그리고 도구에 따라 추가적으로 Hidden Sector도 맞춰줘야 하는 경우도 있으니 참고하자.

> NTFS에서 MFT 엔트리는 각 파일 및 디렉터리마다 하나씩 생성되며 실제 파일과 디렉터리의 메타 정보를 저장하고 관리하는데 사용된다. 파일시스템 특성상 MFT 영역 자체도 하나의 파일인 $MFT로 인식하기 때문에 $MFT는 전체 MFT 영역의 메타 정보를 유지하고 있는 엔트리이다.

일반적인 경우에 섹터 크기가 512 Bytes, 클러스터 내 섹터 수가 8개로 클러스터 크기는 4,096 Bytes가 된다. 저장소 공간의 경우에는 한 섹터의 크기가 4,096 Bytes, 클러스터 내 섹터 수는 1개로 섹터가 일반적인 클러스터 크기만큼 크다.

결론적으로 말하면 일반적인 NTFS와 클러스터 크기는 동일하다.

- 일반적인 NTFS 클러스터 크기: 512Bytes * 8개 = 4,096 Bytes
- 저장소 공간의 클러스터 크기: 4,096 Bytes * 1개 = 4,096 Bytes

### $MFT 실제 위치 확인
위에서 VBR 시작점 기준으로 $MFT의 위치가 0xC0000(786,432 CL) 클러스터로 확인이 되었다. 즉, VBR부터 $MFT까지 0xC0000000 Bytes 만큼 떨어져 있다는 것을 의미한다.

- 0xC0000(786,432) Clusters * 0x1000(4,096) Bytes = **0xC0000000(3,221,225,472) Bytes**

이는 일반적인 NTFS VBR에서 확인되는 $MFT의 위치랑 동일한 값이다.   
실제로 위치를 확인하여 저장소 공간 디스크 상 $MFT의 위치가 위 값과 동일한지 검증을 해보자. 그 전에 NTFS VBR 기준 위쪽 데이터는 파일시스템을 해석하기에 상관없는 영역이므로 모두 날려버리고 VBR 시작점을 offset 0로 맞췄다.

<p align="center">
  <img src="https://i.imgur.com/D06sJP3.png" alt="image"/>
</p>

$MFT의 위치를 확인했더니 0xC0000000 위치가 아닌 <span style="color:red">**0x2000(8,192) Bytes**</span> 만큼 떨어져 있다.  
VBR에 적혀있던 거리와는 다르게 실제로는 $MFT 위치가 다른 곳으로 되어 있어 정상적으로 메타데이터를 파싱하지 못했을 것이라는 가설을 생각해볼 수 있다.

<p align="center">
  <img src="https://i.imgur.com/14KNAlH.png" alt="image"/>
</p>

그럼 복원을 시도하기 위해서는 어떻게 해야 할까?

### 복원 프로세스
$MFT 거리를 조정하여 VBR 내 존재하는 값을 간단히 수정하면 된다. 참고할 사항은 E01 이미지 파일은 수정이 되지 않기 때문에 DD(RAW) 형태로 이미징을 변경해 준 뒤 winhex 등 도구를 통하여 수정을 할 수 있다.
테스트 환경 상 VHD로 저장소 공간을 구성했기 때문에 바로 수정이 가능하다.

그럼 값을 얼마나 수정해야 될까? 쉽게 생각하자. 저장소 공간 NTFS의 클러스터의 크기는 4,096 Bytes이다. 그리고 실제 $MFT는 VBR 시작점으로부터 8,192 Bytes 만큼 떨어져 있다.
그럼 결론은 **두개의 클러스터**만큼 떨어져 있다는 계산이 된다.

- $MFT엔트리 시작 주소 = $MFT엔트리까지의 클러스터 거리 * 섹터 크기 * 클러스터 내 섹터 수  
  → 8,192(0x2000) = X * 4096 * 1  
  → X = 2

이를 도식화하면 아래 그림과 같다.
<p align="center">
  <img src="https://i.imgur.com/FV3Koyo.png" alt="image"/>
</p>

복원을 하기 위해 위 수식으로부터 도출된 거리(Cluster 2)를 이용하여 클러스터 개수에 해당하는 VBR 값을 수정하는 작업을 하면 복원될 수 있을 것 같다.


### 파일시스템 복원 결과
VBR 내 적혀있는 $MFT 위치의 값을 수정한 뒤 다시 도구로 로드해보자.

**VBR 변경 전**
<p align="center">
  <img src="https://i.imgur.com/4ilT0sn.png" alt="image"/>
</p>

**VBR 변경 후**
<p align="center">
  <img src="https://i.imgur.com/O4FolSg.png" alt="image"/>
</p>

변경된 이미지로 도구에 다시 로드 시 이전과 똑같이 아무것도 나오지 않는 이유는 지금까지의 작업은 VBR부터 $MFT 엔트리까지의 거리만 맞춘 것이다. 

<p align="center">
  <img src="https://i.imgur.com/UjeUAlS.png" alt="image"/>
</p>

실제로 파일, 폴더 등 데이터를 정상적으로 해석하기 위해서는 도구의 파일시스템 복원 기능인 X-Ways Forensics의 'Particularly thorough file system data structure search', EnCase의 'Recover Folders' 기능을 이용하여 파일시스템 데이터 구조 검색 및 복원 작업이 필요하다.

복원율을 더 높이기 위해서 데이터 카빙을 통한 파일 복원을 같이 수행할 수 있다. 카빙으로 추가적인 파편화된 파일도 복원할 수 있다. 아래는 모든 복원이 완료된 모습이다.

<p align="center">
  <img src="https://i.imgur.com/wQtxM2d.png" alt="image"/>
</p>

실제로 테스트를 위해 구성했던 트리구조가 정상적으로 복원이 되었고, 파일도 일부는 원본 파일과 해시값까지 동일한 상태로 복원이 되었다.

물론 저장소 공간을 구성할 때 3개의 디스크를 이용하여 가상 볼륨을 구성하였다. 따라서 slab 단위로 데이터가 나눠져서 저장되는 환경 때문에 일부 파일은 원본 해시값과 다른 상태로 복원이 되는 경우도 있었다.

하지만 수동 복원을 통해 디스크의 파일/폴더 전체의 레이아웃을 알 수 있고 특히 파일의 타임스탬프가 분석이 가능해졌다. 그리고 $LogFile, $UsnJrnl, $MFT 등 NTFS 메타 파일을 복원이 가능하여 파일시스템의 일부 아티팩트 분석이 가능해져 또 다른 분석 결과를 낼 수 있는 기회가 생긴 셈이다.


## 참고 사항
### EnCase에서의 특이점
특이하게도 EnCase 도구(EnCase 8.11, EnCase 20.2 동일)에서는 섹터 크기(512 Bytes), 클러스터 내 섹터 수(8개)까지 일반적인 NTFS 값과 동일하게 변경해 줘야만 정상적으로 복원이 가능하다.

<p align="center">
  <img src="https://i.imgur.com/vodnbSS.png" alt="image"/>
</p>

**EnCase 20.2에서 복원 완료된 모습**
<p align="center">
  <img src="https://i.imgur.com/0mJjoaB.png" alt="image"/>
</p>


심지어 섹터 크기를 1024 Bytes로 변경해 주고 Recover Folder 기능을 이용하면 Hit 되는 건수는 나오나 트리구조 형태로 보이지 않을 뿐더러 아무것도 보여주지 않음을 확인하였다.

<p align="center">
  <img src="https://i.imgur.com/k7c4eUN.png" alt="image"/>
</p>

### Magnet AXIOM 결과
AXIOM 4.01에서는 수정된 이미지를 로드하였으나 정상적으로 파일시스템을 해석하지 못하였다.

<p align="center">
  <img src="https://i.imgur.com/BeYXMRX.png" alt="image"/>
</p>

## Wrap-up
지금까지 저장소 공간의 디스크를 대상으로 파일 시스템을 수동 복원 하는 방법을 알아보았다. 다중 디스크 구성 환경에서 한 개의 디스크로도 실제 파일과 메타데이터를 복원 가능성을 확인하였다.

다음 포스트에서는 카빙 된 파일과 파일시스템 수동 복원된 결과를 비교하고, 복원된 메타 파일의 이용하여 아티팩트 분석을 하여 분석된 정보를 확인할 예정이다.


## Reference
- <http://www.x-ways.net/winhex/manual.pdf>


## Copyright (CC BY-NC 2.0 KR)
본 게시글은 CC BY-NC 2.0 KR Licence를 따릅니다.  
비영리 목록으로만 사용할 수 있고, 저작자와 출처를 표시하면 언제든지 게시글을 자유롭게 사용할 수 있습니다.

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
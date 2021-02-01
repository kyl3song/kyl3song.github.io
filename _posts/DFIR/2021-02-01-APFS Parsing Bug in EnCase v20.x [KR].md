---
title : "Blog #24: APFS Parsing Bug in EnCase v20.x [KR]"
category :
  - Miscellaneous
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - File System
  - APFS
  - Apple File System
  - macOS
  - EnCase v20.4
  - EnCase v8.11
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
APFS 파싱 버그와 EnCase 버전별 비교


## This Post Covers

BitLocker로 암호화된 디스를 획득 후 EnCase 등 도구에 로드하면 복호화에 필요한 비밀번호를 묻는 팝업창이 나타나게 된다.

마찬가지로 T2칩이 탑재되지 않은 맥북에서 FileVault2가 적용된 APFS(Apple File System)를 획득한 이후 이미지 파일을 EnCase에 로드하면 비밀번호를 묻는 팝업창이 발생된다.

EnCase v20.x에서 유효한 비밀번호를 넣어도 정상적으로 파일시스템을 파싱 해주지 못하는 이슈를 확인하고 정리를 하고자 한다.


## EnCase Version
먼저 버전에 대해 설명하자면, 기존 8.x의 가장 최신 버전인 EnCase 8.11 이후로 갑자기 v20.x로 변경되었다. 처음에 버전이 갑작스럽게 너무 많이 높아지고 UI도 파란 색으로 바뀌면서 뭐가 많이 변경됐나 보다 했지만 크게 달라진 건 아니었다.

버전이 20.X로 변경된 것은 2020년부터 OpenText에서 EnCase 신규 버전을 릴리즈 할 때 분기마다 제품 릴리즈 하는 것에 맞춰 버전이 20.1 ➔ 20.2 ➔ 20.3 ➔ 20.4 형태로 바뀐 것이다.

앞으로는 **(년도).(분기)** 형태로 버전을 관리하고 업데이트할 예정으로 보이니 참고하면 좋을 것 같다.

<p align="center">
  <img src="https://i.imgur.com/Ob2yMYx.png" alt="image"/>
<br>[ Fig 1. Product Release Timeline ]
<br>(Source: OpenText Forensics DataExpert Webinar) </p>


## EnCase v20.4 vs. v8.11
v8.x의 가장 마지막 버전인 v8.11과 현재 시점에서 최신 버전인 v20.4를 대상으로 APFS 이미지를 로드 및 비교하였다.

파일볼트가 걸린 APFS 파일시스템을 파싱하려면 복호화를 해야 하기 때문에 비밀번호 입력창이 나온다. EnCase v20.4에서는 비밀번호가 맞음에도 불구하고 입력창이 **총 3회 반복**하여 현출되었다.

<p align="center">
  <img src="https://i.imgur.com/5omLpTu.png" alt="image"/>
<br>[ Fig 2. Password Required ]</p>

동일한 비밀번호를 3번 입력하고 나면 그제서야 파일시스템을 해석하는 것처럼 보여준다.
마치 정상적인 것 마냥 Catalog도 파싱하고 FS Tree도 구성하는 과정을 보여줘서 분석하는 입장에서는 헷갈릴 수 있는 부분으로 주의가 필요하다.

<p align="center">
  <img src="https://i.imgur.com/ihpp8JY.png" alt="image"/>
<br>[ Fig 3. Processing FS Tree in EnCase v20.4 ]</p>

먼저 Macquisition 도구로 획득 시 **disk1 - APFS Continer (synthesized)** 를 보면 총 5개의 볼륨이 확인되고 이 중에서 실제 유저 데이터는 **<span style="color:red">Macintosh HD - 데이터</span>** 볼륨에 저장된다.

<p align="center">
  <img src="https://i.imgur.com/kQdPBMW.png" alt="image"/>
<br>[ Fig 4. Disk Drives listed by Macquisition ]</p>

버전별로 뭐가 좀 다른지 살펴보았다.

### 1. Volume List in APFS Container
v8.11는 정확히 볼륨 5개를 표시해 주는 반면 v20.4에서는 이것저것 추가로 더 많이 표시되는 것으로 보아 Container SuperBlock을 정상적으로 해석하지 못한 것임을 알 수 있다.

<p align="center">
  <img src="https://i.imgur.com/fdZcHBC.png" alt="image"/>
<br>[ Fig 5. Volume List - EnCase v8.11 ]</p>

<p align="center">
  <img src="https://i.imgur.com/zSdIJ9u.png" alt="image"/>
<br>[ Fig 6. Volume List - EnCase v20.4 ]</p>


### 2. Data validity
EnCase v8.11에서는 데이터를 정상적으로 보여주고 있었다. 예를 들어 사진, 문서 파일의 경우 뷰어에서 사진을 볼 수 있었고 동영상의 경우에도 정상적으로 재생이 되었다.

<p align="center">
  <img src="https://i.imgur.com/hBosCjR.png" alt="image"/>
<br>[ Fig 7. EnCase v8.11 - IMG_0459.JPG (valid picture) ]</p>

EnCase v20.4의 경우 데이터를 정상적으로 표시해 주지 못하였다. 파일 및 폴더명은 정상적이나 대부분의 사진은 볼 수 없었고, 몇몇 사진은 보이긴 하는데 원본 사진과 다르게 아래쪽이 약 70% 정도 보이지 않는 형태로 표시해 주기도 하였다.

<p align="center">
  <img src="https://i.imgur.com/5S48jBF.png" alt="image"/>
<br>[ Fig 8. EnCase v20.4 - IMG_0459.JPG (invalid picture) ]</p>

FAT32, NTFS에서도 섹터가 밀렸을 경우 정상적으로 표시해 주지 못하는 현상과 비슷했다. NTFS로 비유하자면 $MFT는 정상적으로 접근하여 파일, 폴더 등 정보들은 잘 가져왔으나 Data Run(run list)에서 저장된 length 또는 offset 값이 실제 파일이 저장된 위치와 달라 발생하는 것처럼 보인다.

<p align="center">
  <img src="https://i.imgur.com/f8LUeYW.png" alt="image"/>
<br>[ Fig 9. EnCase v20.4 (partially invalid) ]</p>


### 3. Export Files & Compare them
사진 파일을 대상으로 오프셋 시작이 잘못됐는지 살펴봤더니 파일 시그니처가 보여 파일 시작 지점은 정상적인 것으로 볼 수 있다.

<p align="center">
  <img src="https://i.imgur.com/oeqSLkf.png" alt="image"/>
<br>[ Fig 10. Header of the sample picture in EnCase v20.4]</p>

양쪽 버전에서 샘플 사진 파일을 추출한 뒤 winhex를 통해 차이점을 확인했다.

오프셋 0x1000부터 차이가 확연했다. 즉 **0x0 ~ 0x1000(4096 bytes)** 까지는 원본과 동일한 데이터고 그 이후부터는 완전히 다른 데이터가 들어가 있는 것이 보였다. 맨 마지막 부분은 사진 파일의 Footer도 아닌 것을 확인하였다.

<p align="center">
  <img src="https://i.imgur.com/ldnpOBU.png" alt="image"/>
<br>[ Fig 11. Data compare between v8.11 & v20.4]</p>

또 어떤 파일은 추출을 하려고 하면 오류메시지를 띄워 주면서 추출이 되지 않아 더 이상 파볼것도 없이 이쯤에서 마무리했다. 도구 자체의 버그인 것 같다.


### 4. Entry Counts

마지막으로 파일 리스트 중 누락된 파일도 꽤나 많이 확인할 수 있었는데 전체 엔트리 카운트 수를 비교하니 약 2배 정도 차이가 났다.

<p align="center">
  <img src="https://i.imgur.com/5ft217w.png" alt="image"/>
<br>[ Fig 12. Entry counts between v8.11 & v20.4]</p>


## Wrap-up
예전에 EnCase v20.2에서 처음으로 APFS 파일 시스템을 정상적으로 해석하지 못하는 현상을 경험했었다. 당시에는 그냥 v8.11로 갈아타니 정상적으로 해석되어 버그인가 보다 하고 넘어갔는데 아직 픽스가 되지 않은 것 같다.

당분간 EnCase 도구를 이용하여 APFS 분석할 때는 v8.11로 분석해야 할 것 같다.


## Reference
- <https://developer.apple.com/support/downloads/Apple-File-System-Reference.pdf>
- <https://www.dataexpert.nl/media/rczo0203/opentext-forensics_dataexpert_webinar.pdf>



## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

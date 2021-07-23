---
title : "Blog #4: The Basics of Car Blackbox Forensics part 3"
category :
  - Blackbox
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - Blackbox
  - Dashcam
  - MP4
  - Video Container
  - 블랙박스
  - MPEG4
  - codec
  - FOURCC
sidebar_main : true
author_profile : true
use_math : true
toc: true
toc_sticky: true
toc_label: "Table of Contents"
header:
  overlay_image : /assets/images/post.jpg
  overlay_filter: 0.5
published : False
---
차량용 블랙박스 분석 기본(Part 3)

## Blackbox Post Series
- [Blog #2: The Basics of Car Blackbox Forensics part 1](https://kyl3song.github.io/blackbox/The-Basics-of-Car-Blackbox-Forensics/)
- [Blog #3: The Basics of Car Blackbox Forensics part 2](https://kyl3song.github.io/blackbox/The-Basics-of-Car-Blackbox-Forensics-part-2/)
- [Blog #4: The Basics of Car Blackbox Forensics part 3](https://kyl3song.github.io/blackbox/The-Basics-of-Car-Blackbox-Forensics-part-3/)
- [Blog #5: The Basics of Car Blackbox Forensics part 4](https://kyl3song.github.io/blackbox/The-Basics-of-Car-Blackbox-Forensics-part-4/)

지난 포스트에서 영상 파일 포맷(컨테이너), 코덱, 블랙박스 폴더/파일 구성, AVI 파일에 대한 전반적인 기초 지식을 확인하였다. 원래 모든 학문이 깊게 들어가면 한도 끝도 없는 부분이라 일단 기본적인 부분만 알고 업무에 좀 더 필요한 내용이거나 개인적으로 관심이 생기면 더 깊게 공부하기를 추천한다.  
이번 포스트에서는 코덱에 대해서 추가적으로 필요한 내용을 한 번 정리하고 MP4 파일에 대해서 알아보도록 한다.

## 영상 코덱 추가 설명 (Details of Video Codecs)
### MPEG(Moving Picture Experts Group) / H.264
코덱을 검색하다 보면 같은 코덱임에도 불구하고 서적이나 기술 문서에서 다양한 이름으로 불리고 있다. 예를 들면, 현재 가장 많이 사용되는 'H.264' 코덱은 'AVC'라고 불리기도 하고, 'MPEG-4 Part 10'라고 불리기도 한다. 코덱 종류는 매우 다양한데 이름도 다양하니 헷갈리기 쉽다.

MPEG(Moving Picture Experts Group)은 국제표준화단체로서의 공식 명칭은 ISO/IEC JTC1/SC29/WG11이다. MPEG은 ITU-T(국제전기통신연합)의 비디오 코딩 전문가 그룹(VCEG, Video Coding Experts Group)과 공동으로 조인트 비디오 팀(Joint Video Team)을 구성하여 비디오 압축 표준화를 진행하였다.

<p align="center">
  <img src="https://i.imgur.com/XB8SgyX.png" alt="image"/>
</p>

위의 [표](https://www.nexpert.net/364)는 단체마다 코덱의 이름을 정리한 것이다. 결국 ITU-T의 'H.264'와 ISO/IEC의 'MPEG-4 Part 10 AVC'(공식적으로는 ISO/IEC 14496-10-MPEG-4 파트 10, 고급 비디오 부호화)는 결국 같은 코덱인데 각 단체에서 계속 서로 다른 이름으로 불리고 있다. 실무상으로는 H.264라고 가장 많이 불리는 것 같다.
블랙박스 영상에서도 H.264 코덱이 가장 많이 사용된다. 이외에도 MPEG-4 Part2(MPEG-4 Visual)등 코덱이 사용되기도 한다.

### H.265 코덱의 낮은 이용률(Why H.265 is not popular)
최근에는 H.265(HEVC) 코덱을 이용한 영상들도 많이 나오고 있다. 경험상 DVR에서는 해당 H.265 코덱 구조를 가진 영상 파일을 본 적이 있는데, 블랙박스에는 아직까지 개인 경험상 확인한 적은 없다.
그럼 블랙박스 시장은 어떨까?   
이미 다수의 [블랙박스 제조사](http://blackvue.co.kr/review/?mod=document&uid=533)에서 예전부터 H.265 코덱을 이용한 블랙박스 제품을 판매하고 있다. 하지만 대부분 H.264와 같이 옵션으로 지원하는 것으로 보인다.

H.265 코덱은 H.264보다 높은 압축률로 인해 영상 렌더링, 디코딩하는데 동일한 하드웨어 스펙내에서 시간이 더 오래 걸린다. 아무래도 더 높은 화질의 영상을 지원하려면 블랙박스 하드웨어 성능 또한 받쳐줘야 한다. 그런데 하드웨어 성능을 높이려면 가격이 비싸지기 마련이다.
또한 H.265 코덱이 H.264 코덱보다 보급이 더딘 [이유](https://namu.wiki/w/H.265#s-5.3)로 고사양의 컴퓨팅 환경이 필요, 레거시 환경에서의 저효율 등의 원인으로 분석하고 있다.


### 오픈소스 코덱(Open Source Codec - AV1)
많은 기술 기업이 H.265의 특허 사용료와 기술 제약 때문에 오픈소스 소프트웨어에 눈을 돌리고 있다는 [기사](https://www.zdnet.co.kr/view/?no=20190404122747)도 종종 볼 수 있다. 해당 기사에서 설명한 AV1(AOMedia Video 1)은 오픈 포맷 비디오 압축 포맷으로 Google, Facebook 등이 참여하는 Alliance for Open Media에서 만들었으며, 파일 포맷이 공개되어 있다.
앞으로는 AV1 코덱으로 압축된 형태의 블랙박스 영상 파일을 보는 날도 곧 다가올 것으로 예상되기도 한다.

### FOURCC & Codec Identifier
코덱을 말할 때 FOURCC라는 단어를 많이 언급한다. [FourCC(Four Character Code)](https://www.fourcc.org/fourcc.php)는 말 그대로 "4글자 코드"라는 뜻이며, FourCC는 주로 AVI 파일의 영상 코덱을 구분할 때 사용된다. DIVX, MJPG(Motion JPEG), H264 등과 같이 표현한다.
코덱을 확인할 수 있는 도구는 [AVicodec](http://avicodec.duby.info/), [GSpot](http://www.headbands.com/gspot/), [MediaInfo](http://mediainfo.sourceforge.net/en) 등 여러 가지가 있는데 원하는 도구를 사용하면 된다.


## MP4 Format
### Container Format
MP4 파일의 구조는 아톰(atom) 또는 박스(box)로 표현된다. [이미지 출처](https://unipro.tistory.com/104)

<p align="center">
  <img src="https://i.imgur.com/I4lBRo1.png" alt="image"/>
</p>

atom은 MP4 파일 구조 내 계층적 구조로 미디어 타입, 데이터를 저장할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/JnO5quv.png" alt="image"/>
</p>

MP4 컨테이너에서 기본적으로는 최상단 루트 레벨의 ftyp, mdat, moov 박스만 개념을 잡으면 된다. 그리고 AVI와 다르게 MP4는 데이터 저장 방식이 Big-endian 방식이다.

- ftyp(file type box):
  - 파일 타입, 호환성 등을 표시하는 부분이다.
- mdat(media data box)
  - 실질적인 비디오/오디오 프레임 데이터(미디어 데이터)가 존재하는 부분이다. 즉, 영상/음성은 이 부분에 존재한다는 뜻이다.
- moov(movie box):
  - 메타 데이터(meta data)가 포함되는 부분이다. MP4 파일의 최상위 수준에 저장된다(파일의 시작 또는 끝부분의 가까이에 존재). 각 데이터의 track이 포함되고, 각 track에는 데이터의 형식(비디오 데이터의 경우에는 codec 종류와 해상도를 나타내는 정보, 오디오 데이터의 경우에는 codec의 종류와 channel, sampling rate, bit per sample 등의 정보)이 저장된다. 또한, 각 비디오/오디오 프레임(sample) 데이터에 관련한 정보가 저장된다.
  - MP4 포맷은 모든 프리젠테이션 레벨 정보(메타 데이터)를 비디오/오디오 데이터를 포함하는 멀티미디어 데이터(미디어 데이터)로부터 분리하고 이것을 파일 내의 하나의 내장된 구조인 moov 박스에 저장한다.
  - 메타 데이터가 분리된 미디어 데이터는 mdat box로 구분되어 저장되며, 여기에 저장되는 미디어 데이터는 moov box를 참조하여 해석된다.


### MP4 Structure with binary file
대부분의 블랙박스 영상 파일은 ftyp, mdat, moov box 순서대로 파일이 구성되어 있다. 그러나 스트리밍 영상이나 일반 영상 중에서도 moov와 mdat box가 바뀌어 있는 경우도 자주 관찰할 수 있다.   
MP4 파일의 구조를 살펴보자.
<p align="center">
  <img src="https://i.imgur.com/xb1UXtY.png" alt="image"/>
</p>

<span style="color:red">**ftyp box**</span>
- OFFSET 0x00~0x03(Size): 0x00000020 -> 32 byte  
  문자열 앞 4바이트는 ftyp 박스의 사이즈를 나타낸다. 특히 사이즈는 사이즈 바이트를 포함한다. 즉 오프셋 0x00부터 32byte만큼의 크기를 가진 것이다.
- OFFSET 0x04~0x07(box): ftyp box signature
- OFFSET 0x08~0x0B(brand): avc1  
  MP4는 ISO/IEC 기반 미디어 파일 형식을 식별하기 위해 "[브랜드(brand)](https://en.wikipedia.org/wiki/ISO/IEC_base_media_file_format)"라는 것이 파일 형식의 식별자로 사용되는데, 다수의 브랜드 값 중 "avc1", "isom"을 가장 많이 볼 수 있다.

<span style="color:red">**mdat box**</span>
- OFFSET 0x20~0x23(Size): 0x019A8873 -> 26,904,691 byte (25MB)  
  mdat 박스의 사이즈를 나타낸다. 블랙박스 영상, 음성이 모두 들어 있는 중요한 영역이며 사이즈를 계산할 때 ftyp 박스와 동일한 방식으로 오프셋 0x20부터 25MB만큼의 크기를 가진 것이다.
- OFFSET 0x24~0x27(box): mdat box signature

<span style="color:red">**moov box (+mvhd box)**</span>
<p align="center">
  <img src="https://i.imgur.com/mxjfr43.png" alt="image"/>
</p>

- OFFSET 0x19A8893~0x19A8896(Size): 0x2F01(12,033 byte)  
  moov 박스는 메타데이터가 저장된 영역이다.
- OFFSET 0x19A8897~0x19A889A(box): moov box signature
- OFFSET 0x19A889B~0x19A889E(mvhd size): 0x6C(108 byte)  
  moov 박스 내부에는 mvhd(moovie header)가 존재하는데 해당 영역도 마찬가지로 mvhd box 시그니처 앞 4바이트가 사이즈를 의미한다.
- OFFSET 0x19A88A7~0x19A88AA(Create Time): 0xDA1F2FE1(2019. 12. 18. 01:22:41 LT)  
  OFFSET 0x19A88AB~0x19A88AE(Modify Time): 0xDA1F2FE1(2019. 12. 18. 01:22:41 LT)  
  mvhd에는 영상의 시간 값이 저장되는데 시간 값은 UnixTime, HFS, HFS+ 등 시간이 저장될 수 있다.

<p align="center">
  <img src="https://i.imgur.com/XKwJUEj.png" alt="image"/>
</p>

현재 샘플 파일의 생성 시간, 수정 시간이 모두 동일하고 HFS로 시간이 저장되어 있다. 만일 UnixTime으로 저장되어 있을 경우, UTC+9 보정 작업을 해줘야 하는 경우가 대부분이다.
정확한 것은 파일명, 블랙박스 영상에 현출되는 시간 값, 파일 시스템의 생성 시간을 참조해서 시간을 보정(bias)해줘야 한다. 

이 경우에는 HFS로 저장이 되었고 파일명과 영상에 현출되는 시간으로 명확한 시간 및 보정 값을 확인할 수 있다.
- 파일명: EVT_2019_12_18_01_22_41_F.MP4
- 영상에 현출되는 시간: 2019. 12. 18. 01:22:32 ~ 2019. 12. 18. 01:22:50

아래는 mvhd box의 구조로 box(atom) 버전에 따라 시간 값을 다르게 사용하기도 한다. mvhd box에서는 영상의 다양한 메타정보를 확인할 수 있는 box이다. ([이미지 출처](https://m.blog.naver.com/PostView.nhn?blogId=yesing1&logNo=70096278829&proxyReferer=https%3A%2F%2Fwww.google.com%2F))
<p align="center">
  <img src="https://i.imgur.com/GZgLfN1.png" alt="image"/>
</p>

더 자세한 mp4 구조는 [여기](https://developer.apple.com/library/archive/documentation/QuickTime/QTFF/QTFFChap2/qtff2.html)를 참조 바란다.


## 복원 로직 (Recover Logic)
AVI와 마찬가지로 MP4 영상 파일도 아래와 같이 세 가지의 단계를 거쳐 복원을 할 수 있다.

1. 파일 시스템 레벨 복원
2. 파일 단위 카빙 복원
3. 프레임 단위 복원

가장 베스트한 방법은 언제나 첫 번째 방법이다. 첫 번째 방법이 여의치 않을 때 두 번째, 세 번째 단계를 거쳐야 한다.
이 포스트에서는 기초 단계인 파일 단위 카빙을 설명한다.

MP4는 카빙을 할 경우 구조가 깨지는 경우가 많이 있다. 특히 MP4 영상의 경우 구조가 틀어지면 아예 재생이 안 되는 경우가 AVI 보다 현저히 많기 때문에 고려해야 할 사항들이 있다.
특히 각 box의 사이즈가 실제 데이터의 크기와 달라서 재생 불가한 문제가 있는데, 사이즈 바이트에 기재된 크기와 실제 데이터 크기가 단 1바이트만 달라도 재생이 되지 않는다.

사이즈를 1바이트만 변경한 뒤(0x73 -> 0x74) 영상을 재생해보면 재생이 되지 않는것을 확인할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/dCxBDpZ.png" alt="image"/>
</p>

[재생 불가]
<p align="center">
  <img src="https://i.imgur.com/fHWSWVl.png" alt="image"/>
</p>

MP4는 구조 때문에 복원하는 과정에서 mp4 box 구조를 맞추기가 어렵고, 정교한 작업이 필요한 수동으로 복원하는 케이스가 많지 않아 도구를 사용하는 것을 추천한다.

파일 복원 도구를 이용하여 복원을 하게 되면 한 뒤, [videorepair](http://grauonline.de/cms2/?page_id=5)와 같은 깨진 영상 복원 도구를 이용하면 복원이 일부 가능할 수 있다. 해당 도구에서는 깨진 파일, 정상 파일 두 개를 입력으로 받아 정상적인 파일을 참조하여 손상된 영상 파일의 moovie box 등 영상 재생에 필요한 메타데이터를 만들어 주는 방식이다.

영상의 구조를 파악한다는 것은 재미없고 지루한 일이다. 그렇지만 복원을 하기 위해서는 영상의 구조뿐 아니라 코덱을 어느정도 이해하는 것이 중요하다.

두 가지의 개인적인 경험에 의한 사례가 있다. 첫 번째는 영상에 대한 이해가 없었을 때였는데, 당시 기억을 되짚어보면 드론(Drone)을 통해 어느 집을 촬영한 사건이었다. 드론에 삽입된 MicroSD 매체에 대해 분석요청을 했었다.  
먼저 기본적인 파일시스템 분석 후 카빙의 방법을 통해 파일을 복원했던 것으로 기억하는데, 당시 파일은 복원이 됐지만 정작 재생할 수 없는 파일이었다. 그리고 더 안타까웠던 것은 정상적으로 촬영된 파일도 없어 정상파일에 대한 구조분석 조차 힘들었다.
결론적으로 영상에 대한 내용은 빠진 채 분석한 내용으로만 최대한 보고서를 작성하여 마무리 지었다.
이 때 만일 영상(+코덱)에 대한 좀 더 있었더라면 프레임이라도 복원했을 것이다. 예전에 갑자기 이 사건이 기억나서 혹시나 샘플이 있나해서 찾아봤으나 역시 내 PC에서 삭제되고 없었다. 증거 분석이 완료되면 사건과 관련된 모든 것은 파기해야 하기 때문이다.

두 번째는 복원이 된 케이스이다.  
어느 여성이 지하철 물품보관함에 물건을 넣고 있을때 몰래 휴대폰을 이용하여 촬영한 사건이었다. 안드로이드 휴대폰에서 기본 사진앱으로 촬영된 영상 파일은 통상 "20190522_195126.mp4" 파일명으로 저장된다. 기타 사진앱을 설치하여 사진, 영상을 촬영할 경우 파일명이 다를 수는 있지만 대부분 [연월일시분초] 형태로 저장이 된다.
당시 영상 파일명에 기재된 시간과 범행 시간이 얼핏 비슷하게 맞았는데 정말 중요한 단서가 되는 영상이 재생이 되지 않는 상황이었고, 프레임 복원 단계로 일부 영상에 대해 복원을 성공한 적이 있어 복원된 영상, MP4 내부의 메타데이터 등을 바탕으로 보고서를 작성하여 마무리를 할 수 있었다.


## Reference
- <https://ko.wikipedia.org/wiki/MPEG>
- <https://www.nexpert.net/364>
- <https://manorgass.tistory.com/61>
- <http://blackvue.co.kr/review/?mod=document&uid=533>
- <https://namu.wiki/w/H.265#s-5.3>
- <https://www.zdnet.co.kr/view/?no=20190404122747>
- <https://ko.wikipedia.org/wiki/AV1>
- <https://unipro.tistory.com/104>
- <https://openmp4file.com/format.html>
- <https://patents.google.com/patent/KR101316579B1/ko>
- <https://en.wikipedia.org/wiki/ISO/IEC_base_media_file_format>
- <https://m.blog.naver.com/PostView.nhn?blogId=yesing1&logNo=70096278829&proxyReferer=https%3A%2F%2Fwww.google.com%2F>
- <https://developer.apple.com/library/archive/documentation/QuickTime/QTFF/QTFFChap2/qtff2.html>
- <https://ilovegaz.tistory.com/250>


## Copyright (CC BY-NC)
본 게시글은 CC BY-NC Licence를 따릅니다.  
비영리 목록으로만 사용할 수 있고, 저작자와 출처를 표시하면 언제든지 게시글을 자유롭게 사용할 수 있습니다.

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

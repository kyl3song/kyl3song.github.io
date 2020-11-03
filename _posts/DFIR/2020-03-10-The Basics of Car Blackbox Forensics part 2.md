---
title : "Blog #3: The Basics of Car Blackbox Forensics part 2"
category :
  - Blackbox
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - Blackbox
  - Dashcam
  - AVI
  - Video Container
  - 블랙박스
  - H.264
  - AVC1
  - codec
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
차량용 블랙박스 분석 기본(Part 2)

## Blackbox Post Series
- [Blog #2: The Basics of Car Blackbox Forensics part 1](https://kyl3song.github.io/blackbox/The-Basics-of-Car-Blackbox-Forensics/)
- [Blog #3: The Basics of Car Blackbox Forensics part 2](https://kyl3song.github.io/blackbox/The-Basics-of-Car-Blackbox-Forensics-part-2/)
- [Blog #4: The Basics of Car Blackbox Forensics part 3](https://kyl3song.github.io/blackbox/The-Basics-of-Car-Blackbox-Forensics-part-3/)
- [Blog #5: The Basics of Car Blackbox Forensics part 4](https://kyl3song.github.io/blackbox/The-Basics-of-Car-Blackbox-Forensics-part-4/)


## 영상 파일 종류 (Video File format)

이번 포스팅에는 영상 파일 및 폴더에 대해 조금 다뤄보려고 한다. 동영상 파일은 누구나 한번씩 재생해본 경험이 있을 것이다. 영상 파일은 드라마, 영화, 인강 등 다양한 영상을 담고 있지만 영상뿐 아니라 소리 및 부가 정보까지 담고 있는 것을 의미한다. 이러한 영상 파일의 껍데기를 <span style="color:red">**'컨테이너(container)'**</span>라고 부른다. 컨테이너는 우리가 잘 알고 있는 .AVI, .MP4, .MKV 등 파일 포맷을 말한다.  
정확하게 표현하면, 여러 데이터 구성요소들이 메타데이터와 파일 안에서 같이 있을 수 있는 포맷을 컨테이너 포맷(container format)이라고 한다.

블랙박스에서 저장되는 영상 파일의 형태는 대부분 AVI, MP4 포맷을 사용하나 가끔 TS 확장자로 저장하는 기기도 드물게 있다. 그리고 제조사의 전용 뷰어를 통해서만 볼 수 있는 영상 포맷도 존재한다.


### 영상 컨테이너의 예시 (Example of video container)
- AVI
- MP4
- TS
- JDR
- RIM
- AV2

영상 파일에 대해서는 포스트 뒤쪽에서 다시 알아보도록 하자.


## 폴더 구조 (Folder Structure)

블랙박스에 저장된 영상을 담고 있는 폴더의 구조는 대부분 블랙박스의 녹화 모드에 따라 구조가 나뉘어 진다. 크게 상시 녹화, 주차 녹화, 이벤트 녹화, 수동 녹화로 나뉘고 이에 따라 파일명도 비슷한 구조를 가지고 있다. 지금은 단종됐지만 [Mando KV200](https://www.mandoplaza.com/) 블랙박스에서 사용되는 폴더명 및 파일명은 다음과 같다.

<p align="center">
  <img src="https://i.imgur.com/BBiIpC5.png" alt="image"/>
</p>

또 다른 블랙박스인 [PONTUS Sense(R620DL)](http://www.pontus.co.kr/product/view/194)의 매뉴얼을 살펴보면 폴더 구조가 비슷하지만 조금씩 차이가 있다.

<p align="center">
  <img src="https://i.imgur.com/YKpINzZ.png" alt="image"/>
</p>

대략적인 블랙박스의 폴더명, 파일명의 구조는 아래와 같이 정리할 수 있다.

### 상시 녹화 
차량에 전이 인가되거나 시동을 걸면 기기 부팅이 된 후의 녹화
   - 폴더명: INF, NORMAL, DRIVING 등
   - 파일명: 연월일시분초_NOR_FILE.avi / REC_연월일시분초_채널수.avi 등
<p align="center">
  <img src="https://i.imgur.com/Gh0pZ8M.png" alt="image"/>
</p>

### 이벤트 녹화
차량의 충격이 감지되면 이벤트 녹화가 시작
   - 폴더명: EVT, EVENT 등
   - 파일명: 연월일시분초_EVT_MENU.avi / EVT_연월일시분초_채널수.avi 등
<p align="center">
  <img src="https://i.imgur.com/yfoyOyg.png" alt="image"/>
</p>

### 주차 녹화
차량 시동을 끄거나 전원이 인가되지 않는 경우에 주차모드 녹화
   - 폴더명: PARK(모션 감지), EVT(충격 감지), PARKING 등
   - 파일명: 연월일시분초_PAK_EXIT.avi / MDI_연월일시분초_채널수.avi 등
<p align="center">
  <img src="https://i.imgur.com/tAJP0H7.png" alt="image"/>
</p>

### 수동 녹화
사용자가 수동으로 녹화시 저장
   - 폴더명: MANUAL, USER, EVENT 등
   - 파일명: 연월일시분초_MAN_RCVR.avi / EVT_연월일시분초_채널수.avi 등

이처럼 폴더명은 녹화 종류에 따라 지정된 폴더에 영상이 저장되고, 파일명에는 촬영 일시, 녹화 종류, 채널수 등 내용으로 구성된다.
조금 더 심플하게 EVENT, INF, PARK 폴더 형태로 저장되는 경우도 있으나 전반적인 구조는 비슷하다고 보면 된다.


## 제품별 파일명 (Filename by Blackbox Models)

이번에는 블랙박스 제품별 전방/후방 영상 파일명을 기준으로 살펴보려고 한다. 다음은 블랙박스 제품명, 모델명을 기준으로 해당 기기에서 사용하는 영상 파일의 이름을 정리한 것이다.

|No.|블랙박스 제품명|모델명|파일명|
|:---:|:---:|:---:|:---:|
|1|다본다(Dabonda)  |DBH-3500F|20000102_192354_I2.avi|
|2|만도(Mando)      |KP100    |2019_0208_085643_NOR_FILE.AVI|
|3|만도(Mando)      |KV200    |2018_0630_202523_PAK_FILE.avi|
|4|미르테크(MirTech)|BH-2     |00.rim|
|5|BenzStartView    |IRIVER-X350<br>(OEM)|REC_2019_09_16_18_19_36_F.MP4 (전방)<br>REC_2019_09_16_18_19_36_R.MP4 (후방)|
|6|뷰게라(VUGERA)   |RG7      |20190217_225445_E_A.avi (전방)<br>20190217_225445_E_B.avi (후방)|
|7|뷰게라(VUGERA)|VG-701V|REC_2019_02_26_13_46_22_F.MP4 (전방)<br>REC_2019_02_26_13_46_22_R.MP4 (후방)|
|8|뷰게라(VUGERA)|VG-701V3|EVT2_20190904_081611.avi|
|9|뷰게라(VUGERA)|VG-900V3|EVT2_20191011_102503.avi|
|10|아이나비(INavi)|QXD1000|REC1_20190228_184830.avi|
|11|아이나비(INavi)|V100|Rec_20190423_094642_D.avi|
|12|아이나비(INavi)|V900|REC_2018_05_10_17_30_56_F.MP4 (전방)<br>REC_2018_05_10_17_30_56_R.MP4 (후방)|
|13|아이나비(INavi)|Z300|EVT_2019_12_18_01_22_41_F.MP4 (전방)<br>EVT_2019_12_18_01_22_41_R.MP4 (후방)|
|14|아이머큐리(IMERCURY)|SAPPHIRE|20190527_083710_I2.avi|
|15|아이머큐리(IMERCURY)|TOPAZ|20220203_115603_E2.avi|
|16|아이트로닉스<br>(ITRONICS_IPASS)|ITB650HD|20190410_115057_EVT_1.avi|
|17|아이트로닉스<br>(ITRONICS_IPASS)|ITB-5000Plus|20190805_040025_INF_2.avi|
|18|아이트로닉스<br>(ITRONICS_IPASS)|N9|20190808_184953_EVT_2.avi|
|19|위니캠(Winycam)|Winner|evt1_8749_20190408_061921_P.avi|
|20|유라이브(URIVE)|UC3000P|EDR_190420_095126.AVI|
|21|태건(TAEKEON))|SHD-214|cbx_9633_20190210_164247_014_61939_NA.av2|
|22|파인뷰(FineVU)|CR2000S|2019-01-14-08h-54m-40s_F_normal.mp4 (전방)<br>2019-01-14-08h-54m-40s_R_normal.mp4 (후방)|
|23|파인뷰(FineVU)|T20|2015-11-24-23h-50m-41s_F_normal.mp4 (전방)<br>2015-11-24-23h-50m-41s_R_normal.mp4 (후방)|
|24|파인뷰(FineVU)|X300|20190719-12h19m06s_FR_N.avi|
|25|아톰골드<br>(Atom Gold)|EQ3000D|inf1_4100_20180927_204735_I.avi|
|26|폰터스(PONTUS)|R620DL|REC_20190517_210653_2.avi|

파일명이 어떻게 이뤄졌는가도 중요하지만, 여기서 눈여겨봐야 하는 것은 전방/후방 채널의 영상이 따로 존재하는지 단일 파일로 존재하는지에 대한 부분이다.
파일명이 하나만 있는 것은 영상 파일 하나에 2개의 채널이 섞여있는 상태를 의미한다. 만일 전방/후방 채널별 영상 파일이 각자 나눠진 경우 구분할 수 있게 F(Front) 또는 R(Rear)이 대부분 파일명에 포함되어 있다.

``` shell
> 전방(Front): 2019-01-14-08h-54m-40s_F_normal.mp4
> 후방(Rear): 2019-01-14-08h-54m-40s_R_normal.mp4
```

이 부분이 중요한 이유는 영상이 나눠져 있을 때와 하나로 있을 때의 복원 접근 방법 달라지기 때문이다.


## AVI Format

### Container Format
블랙박스에서 가장 많이 사용되는 영상 파일의 형태는 AVI, MP4 컨테이너 포맷이다.
RIFF(Resource Interchange File Format)는 파일 컨테이너 포맷으로 동영상 재생을 위해 Microsoft가 만든 데이터 포맷인데, 여기에서 파생되어 데이터를 "chunk"로 저장하는 방식이 AVI 포맷이다.  
AVI 파일의 시그니처는 이미 많이 알려진 것처럼 ‘RIFF’이다. 과연 이게 맞는 말일까? 맞을 수도 있고 틀릴 수도 있다. 아래 블랙박스 영상 파일의 헥사코드를 보자.

<p align="center">
  <img src="https://i.imgur.com/aBDN6Fu.png" alt="image"/>
</p>

전형적인 AVI 영상 파일이고, RIFF 시그니처도 있다. 그럼 일반적인 음성 파일인 WAV 파일의 헥사코드는 어떨까?

<p align="center">
  <img src="https://i.imgur.com/TTuF4hB.png" alt="image"/>
</p>

파일의 제일 앞 4바이트는 같은데 뒤는 WAVE로 다르다. 즉, 정확히 말하면 AVI 컨테이너는 RIFF 기반에 파생된 형식이라 RIFF 시그니처가 기본적으로 있는 것이다. RIFF 뒤쪽에 있는 AVI(LIST도 포함 가능) 또는 WAVE로 파일에 대한 시그니처를 보완해 주는게 맞다.

물론 위 샘플 AVI 파일에서 제일 첫 4바이트가 RIFF로 시작해야 AVI 파일 타입으로 인식한다. 맨 앞의 한 바이트만 다른 문자열로 바뀌어도 대부분의 동영상 프로그램에서 재생이 되지 않는 것은 맞다. 하지만 AVI의 파일 시그니처는 'RIFF'라는 말은 엄연히 틀린 표현이다.

기본적으로 영상에 있어 중요한 부분은 영상 크기와 코덱의 종류이다. 샘플 파일에서 영상 사이즈와 코덱에 해당하는 값을 살펴보자.
- OFFSET 0x04(Size): 0xB8A9E8 -> 12,102,120 byte
- OFFSET 0xBC(Codec): AVC1


### 영상 크기(Data Size)
영상의 크기의 시작점은 사이즈 오프셋 뒤로부터 계산된다. 샘플 영상에서는 OFFSET 0x08부터 0xB8A9E8 만큼 크기를 가지고 있다는 의미이다. 즉, 앞 8byte를 제외한 영역이기 때문에 영상 전체 크기에서 8byte 만큼 빠진 크기이다.

<p align="center">
  <img src="https://i.imgur.com/lZdiVdp.png" alt="image"/>
</p>
<p align="center">
  <img src="https://i.imgur.com/RjHkzng.png" alt="image"/>
</p>

영상을 복원할 때는 역으로 빠진 8byte를 더해서 복원을 하면 AVI 복원은 완료된다.
정리하면, 기본적으로 영상을 카빙의 방법으로 복원할 때 아래의 단계로 복원을 하게된다.

1. 영상 파일의 Signature(RIFF, AVI)를 찾음
2. 검색이 되면 Size Offset으로 이동해서 Size 값을 읽음
3. 영상 시작점(RIFF)부터 Size+8byte 만큼 추출

무료 도구나 상용 도구에서는 로직은 다를 수 있지만 이런 부분을 대부분 자동으로 수행해 준다. 특히 
X-Ways Forensics 도구의 경우 카빙 기능 외 'Intelligent naming'이라고 해서 Exif의 정보를 이용하여 
카빙된 파일의 이름을 카메라 모델과 타임스탬프를 조합하여 이름을 정해주는 부가적인 기능도 있다.
<p align="center">
  <img src="https://i.imgur.com/XoMCE7h.png" alt="image"/>
</p>

[X-Ways Forensics/WInHex Manual](http://www.x-ways.net/winhex/manual.pdf)을 참고하면 다음과 같이 기술되어 있다.
> With a specialist license or higher, the "intelligent naming" option will cause Exif JPEG files to be named after the digital camera model that created them and their internal time stamp, if available.

영상 파일(blackbox_sample.avi) 사이즈 끝부분에 자세히 보면 4byte가 남는 것을 볼 수 있는데, 이것은 파일 슬랙 영역을 의미한다. 영상 사이즈 이외의 영역 시작점이 [JUNK](https://www.fileformat.info/format/riff/egff.htm)라는 시그니처로 되어 있는 경우도 있는데 이 부분은 패딩 영역이다. 가끔 이런 슬랙 공간에서 영상프레임이 복원되기도 하니 주의 깊게 살펴볼 필요가 있다. JUNK 시그니처가 보이면 그 바로 뒤 4byte가 사이즈를 의미한다.

<p align="center">
  <img src="https://i.imgur.com/e4fOhwm.png" alt="image"/>
</p>

### 영상 코덱(Codecs)
코덱은 영상을 압축하거나 압축된 영상을 해제할 때 사용된다. 코덱에 따라 복원 방법이 달라질 수 있어 여러 [코덱](https://www.fourcc.org/codecs.php)을 공부해보는 것도 도움이 많이 된다.  
샘플 파일에서는 코덱이 AVC1로 되어있다. AVC 코덱은 현재 가장 많이 사용되는 H.264 코덱과 거의 비슷하다고 보면 된다. 단, 이 두 코덱의 [차이점](https://docs.microsoft.com/en-us/windows/win32/directshow/h-264-video-types?redirectedfrom=MSDN)이 있다면 H.264 비트스트림의 startcode가 있냐 없냐의 차이이다. 이게 정확히 어떤 의미인지는 가능하면 추후 포스팅에서 내용을 언급하도록 하겠다.

사실 현업에서는 이런 차이점이 복원하는데 큰 영향을 주지는 않는다. 두 개의 코덱이 거의 비슷하기도 하고, 확실한 건, AVI 파일에서는 프레임 구조가 코덱의 영향을 받는 경우는 거의 보지 못하였다. 경험상 AVI에서는 H264 구조가 많고, MP4는 AVC 코덱의 구조가 많았다.


## 수동 복원이 필요한 이유(Reasons why programming is needed)
블랙박스, DVR분석 등 영상 관련된 복원은 이왕이면 코딩하여 분석을 해야 한다. 그럼 상용툴, 무료툴은 못 미더운가? 절대 그런 것은 아니다. 도구들은 필요할 때 적절히 사용하는 것이 가장 좋은 방법이다.

프로그래밍을 사용하는 대부분의 이유는 내 마음대로 커스터마이즈 할 수 있다는 점이다. 아래 하나의 상황을 가정하였다.

[상황 가정]   
  \- 블랙박스의 비할당 영역에서 AVI 파일을 카빙 해야하는 상황  
  \- 정상적인 블랙박스 영상은 약 100MB 정도의 크기임

[복원 로직]  
영상의 시그니처를 찾은 뒤 영상의 사이즈 값을 읽을 것이다. 그런데 이때 사이즈 값이 비정상적인 값으로 덮어쓰여있다.(ex. FF FF FF FF) 이럴 때 무분별하게 영상 시작점(RIFF)부터 사이즈+8byte 만큼 추출한다면 복원은 의미 없는 행위일 것이다.
이럴 경우, 사이즈 값의 최대값(ex. 120MB)을 설정해서 최대값을 넘을 경우 특정 크기만큼(ex. 120MB) 픽스시켜서 저장하는 로직으로 복원할 수 있다.

사실 도구에서도 이런 문제점을 일부 수정하는 알고리즘이 있다. 그런데 알고리즘이다 보니 내가 정확히 파악하기도 힘들고 위의 상황에서는 최대값의 경우 블랙박스 모델에 따라 정상 영상 크기가 다르기 때문에 내가 직접 정하여 복원하는 방법을 취해야 한다.

실무에서는 이와 비슷한 상황이 로우 레벨 단계인 프레임 단위에서 많이 발생하니 일단 현재는 참고만 하면 될 것 같다.


## Reference
- <https://www.file-recovery.com/avi-signature-format.htm>
- <https://ko.wikipedia.org/wiki/RIFF>
- <https://johnloomis.org/cpe102/asgn/asgn1/riff.html>
- <https://www.fourcc.org/fourcc.php>
- <http://telnet.or.kr/directx/htm/avirifffilereference.htm>
- <https://docs.microsoft.com/en-us/windows/win32/xaudio2/resource-interchange-file-format--riff->
- <https://www.wooleelife.com/27>
- <https://m.blog.naver.com/PostView.nhn?blogId=dbfan24&logNo=10128721121&proxyReferer=https%3A%2F%2Fwww.google.com%2F>
- <https://docs.microsoft.com/en-us/windows/win32/directshow/h-264-video-types?redirectedfrom=MSDN>
- <https://www.fourcc.org/fourcc.php>
- <http://www.x-ways.net/winhex/manual.pdf>
- <https://www.fileformat.info/format/riff/egff.htm>
- <http://ipassmall.co.kr/ipassblack/download_view.asp?bs=&pg=1&seq=298&Cat2=15&sf=bSubject&ss=ITB%2D650HD>
- <https://www.mandoplaza.com/>
  

## Copyright (CC BY-NC)
본 게시글은 CC BY-NC Licence를 따릅니다.  
비영리 목록으로만 사용할 수 있고, 저작자와 출처를 표시하면 언제든지 게시글을 자유롭게 사용할 수 있습니다.

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

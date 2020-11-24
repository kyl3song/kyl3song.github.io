---
title : "Blog #20: NEW Artifact of Chrome Browser (Media History) part 2"
category :
  - Artifacts
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - Chrome
  - SQLite
  - Media History
  - Media Tracking
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
크롬의 새로운 아티팩트 "Media History" part 2

## NEW Artifact of Chrome Browser Series
- [Blog #18: NEW Artifact of Chrome Browser (Media History) part 1](https://kyl3song.github.io/artifacts/NEW-Artifact-of-Chrome-Browser-(Media-History)-part-1/)
- [Blog #20: NEW Artifact of Chrome Browser (Media History) part 2](https://kyl3song.github.io/artifacts/NEW-Artifact-of-Chrome-Browser-(Media-History)-part-2/)


## Preface
지난 글에서 Media History DB의 테이블과 일부 테스트를 통해 내부 저장된 값들에 대해 확인하였다. 오늘은 몇 가지 상황을 가정하고 그에 따라 히스토리 기록이 어떻게 남는지 살펴보도록 한다.

그전에.. 크롬 버전 87.0.4280.66로 업그레이드 했더니 **kaleidoscopeData**라는 또 다른 테이블이 추가되어 있었다.

<p align="center">
  <img src="https://i.imgur.com/b5wL5X5.png" alt="image"/>
<br>[kaleidoscopeData Table]</p>

이 역시 테이블 내 데이터가 남아있지 않아 무슨 역할을 하는 테이블인지 찾아봤더니 넷플릭스, 아마존 프라임 등과 같은 VOD 스트리밍 서비스를 하나의 인터페이스로 통합시켜주는 역할을 하는 테이블로 사용될 것 같다.

<p align="center">
  <img src="https://i.imgur.com/63hjFgu.png" alt="image"/>
<br>[(출처) https://gotipath.com/chrome-kaleidoscope-upcoming-vod-aggregation-service-by-google/]</p>

Chrome Kaleidoscope는 [이곳에서](https://www.chromestory.com/2020/08/chrome-kaleidoscope/) 조금 더 자세하게 확인할 수 있었는데 유저가 방문하는 사이트의 비디오를 자동으로 추가하여 다음에 보기 서비스인 "Watch it Later" Service의 개념을 담고 있는 것 같다.

아무튼 눈 깜짝 할 사이에 DB 테이블들이 생겨나고 있고 또 업데이트가 될 것으로 예상된다.


## Case 1. Play Videos through Cloud Drives
첫 번째 실험은 구글 드라이브와 같이 클라우드 상에서 실시간으로 재생하는 경우 미디어 히스토리에 기록이 되는지 여부이다.

구글 드라이브에 샘플 영상을 업로드 한 뒤 링크를 통해 접근하여 재생하는 방식으로 테스트를 하였다. N번방, 박사방의 사건에서 텔레그램, 디스코드 등과 더불어 성착취물을 공유한 하나의 수단이었다.

<p align="center">
  <img src="https://i.imgur.com/GZBi3Bx.png" alt="image"/>
</p>


간단하게 영상을 약 7초정도 재생한 뒤 일시정지를 하고 크롬 브라우저를 닫았다. 그리고 DB 각 테이블에 기록된 데이터를 살펴보았다.

<p align="center">
  <img src="https://i.imgur.com/Bi4JyxA.png" alt="image"/>
</p>

### origin Table
먼저 Origin 테이블에는 BaseURL 정보가 남았고, 시간값 그리고 총 재생 시간인 7초도 정확하게 남았다.

<p align="center">
  <img src="https://i.imgur.com/AEDd8dT.png" alt="image"/>
</p>

- origin : https://drive.google.com
- last_updated_time_s : 13250601039 ➔ **2020. 11. 23. 19:30:39 (UTC+9)**
- aggregate_watchtime_audio_video_s : 7 ➔ **7 seconds**


### playback Table
구글 드라이브에서 사용됐던 스트리밍 Full URL이 확인되었고, 시청 시간, 동영상 및 음성 포함여부 등 데이터가 확인된다.

<p align="center">
  <img src="https://i.imgur.com/44PEKO6.png" alt="image"/>
</p>

- url : https://drive.google.com/file/d/1Wts~
- watch_time_s : 7 ➔ **7 seconds**
- has_video : 1 ➔ **Video contained**
- has_audio : 1 ➔ **Audio contained**
- last_updated_time_s : 13250601039 ➔ **2020. 11. 23. 19:30:39 (UTC+9)**

### playbackSession Table
영상의 전체 재생 시간(ms), 현재 사용자가 재생한 위치 시간값(ms), 영상 파일명 정보가 정확히 남은 것을 확인하였다.

<p align="center">
  <img src="https://i.imgur.com/rJTpeQK.png" alt="image"/>
</p>

- url : https://drive.google.com/file/d/1Wts~
- duration_ms : 20038 ➔ **20.038 seconds**
- position_ms : 7410 ➔ **7.41 seconds**
- last_updated_time_s : 13250601039 ➔ **2020. 11. 23. 19:30:39 (UTC+9)**
- source_title : drive.google.com

이와 같이 첫 번째 케이스에서 클라우드를 통해 스트리밍으로 재생된 영상에 대해서도 정상적으로 기록되는 것을 확인할 수 있었다.

하지만 네이버 클라우드(MBOX), 원드라이브(OneDrive)와 같은 클라우드 서비스는 스트리밍으로 재생할 수 있는 방식이 아니고 다운로드 하는 방식이기 때문에 기록이 남지는 않았다.


## Case 2. Autoplay Video when Visiting Websites
사이트에 방문만 해도 영상이 자동으로 재생되는 경우가 있는데 그런 경우에는 기록이 될까?

내가 자주가는 전자기기 리뷰 [유튜브 채널](https://www.youtube.com/channel/UCdUcjkyZtf-1WJyPPiETF1g)을 방문해보자. 채널 메인에 방문하면 굳이 클릭하지 않아도 가장 최신의 영상이 자동으로 재생된다.

<p align="center">
  <img src="https://i.imgur.com/2IosQxu.png" alt="image"/>
<br>[영상 자동 재생의 경우 (유튜브 채널 메인 페이지)]</p>

그리고 남는 기록을 살펴보면 'playback' 및 'playbackSession' 테이블에 미디어 접근 기록이 잘 나타난다.

<p align="center">
  <img src="https://i.imgur.com/eOYR6h3.png" alt="image"/>
<br>[playback 및 playbackSession Table]</p>

또한 음악 스트리밍 사이트(e.g. Amazon Music, Naver VIBE 등) 뿐만 아니라 온라인 영어 사전 사이트에 접속하여 특정 단어를 찾고 발음 듣기 버튼을 클릭할 때 나오는 음성과 같이 영상이 아닌 오디오 재생 시 미디어 히스토리 기록 여부를 확인하였다.

<p align="center">
  <img src="https://i.imgur.com/k05Sp9v.png" alt="image"/>
<br>[Naver VIBE (https://vibe.naver.com/)]]</p>

확인 결과 사이트에 있는 오디오(음성, 음악 등)를 재생하는 경우 playback 테이블에만 기록되고 **playbackSession 테이블에는 기록되지 않았다.**

<p align="center">
  <img src="https://i.imgur.com/AdIwz7T.png" alt="image"/>
<br>[playback Table]</p>

두 번째 케이스에서는 비디오나 오디오의 경우 내가 굳이 재생하지 않더라도 사이트를 접속했을 때 자동 재생을 한다면 본의 아니게(?) 기록이 남을 수 있다.

또한 비디오의 경우 playback, playbackSession 테이블 모두 사용하고 오디오만 있는 경우 playbackSession 테이블에 데이터는 남지 않는 것을 확인하였다.


## Case 3. Skip Ads to watch Videos

세 번째로 유튜브나 네이버 TV 등에서 영상이 처음 재생될 때 광고가 나오고, 또 중간에 광고가 나올경우 그걸 스킵한 경우 기록 여부이다.

<p align="center">
  <img src="https://i.imgur.com/JmLEFfX.png" alt="image"/>
<br>[유튜브 광고]</p>

내가 보고자 하는 영상과 광고 영상이 같이 기록된다면 어떤 방식으로 기록되는지 확인하기 위해 유튜브와 네이버 TV 두 서비스를 가지고 테스트를 진행하였다.

### Youtube Test

영상 테스트는 다음과 같은 순서로 진행하였다.

1. 영상 시작과 동시에 광고 재생 (약 5초간)
2. SKIP 버튼 클릭
3. 영상 계속 재생
4. 02:26에 중간 광고 재생 (약 5초간)
5. SKIP 버튼 클릭
6. 영상 계속 재생
7. 03:00에 영상 종료

DB playback Table 확인 결과 총 4번의 기록이 남았고 DB에 기록된 값과 테스트 순서별로 정리를 하였다.

<p align="center">
  <img src="https://i.imgur.com/TiWNN4K.png" alt="image"/>
<br>[유튜브 광고 건너뛰기 테스트 결과]</p>

1. 영상 시작과 동시에 광고 재생 (약 5초간)  
 \- 첫 번째 광고 5초간 재생된 기록 일치
2. SKIP 버튼 클릭
3. 영상 계속 재생  
 \- 본 영상 처음부터 중간 광고가 나오기 전까지 재생 기록 일치 (146초 또는 2분 26초)
4. 02:26에 중간 광고 재생 (약 5초간)  
 \- 중간 광고 5초간 재생된 기록 일치
5. SKIP 버튼 클릭
6. 영상 계속 재생  
 \- 중간 광고부터 영상 종료까지 재생 기록 일치 (34초)
7. 03:00에 영상 종료  
 \- 유튜브 영상을 총 3분을 재생하였고, DB 기록상 유튜브 순수 영상 재생 시간 일치 (146초 + 34초 = 180초, 3분)

> 참고로 유튜브 및 네이버 TV 영상 재생시간에는 광고시간은 카운트 되지 않는다. 즉 영상 시작하자마자 5초가 광고로 재생되도 광고가 끝난 후 본 영상은 0초부터 시작된다.

### Naver TV Test

네이버 TV역시 유튜브와 비슷한 결과를 보여 따로 설명은 하지 않았도 될 것 같다.

1. 영상 시작과 동시에 광고 재생 (약 5초간)
2. SKIP 버튼 클릭
3. 영상 계속 재생
4. 01:00에 영상 종료

<p align="center">
  <img src="https://i.imgur.com/pfmlpIh.png" alt="image"/>
<br>[네이버 광고 건너뛰기 테스트 결과]</p>


## Wrap-up
지금까지 3가지의 케이스를 직접 실험을 통해 데이터가 어떻게 남는지 확인하였다.

이외에도 위에서 언급은 따로 하지 않았지만 Twitter, Linkedin 등 SNS 내부에 삽입된 영상을 재생할 경우에도 Media History에 기록되었다. 물론 당연한 거겠지만 크롬 브라우저를 이용했을 때에 해당된다.

위에서 실험을 통해 확인한 것들을 정리하면 다음과 같다.

- 클라우드를 통해 스트리밍으로 재생된 미디어 기록이 남음
- 스트리밍으로 재생되는 형태가 아닌 파일 다운로드만 가능한 클라우드 서비스는 미디어 기록에 남지 않음
- 사이트에 방문했을 시 자동으로 재생되는 미디어(음악, 영상 등)의 경우 미디어가 재생됨에 따라 기록이 자동으로 남음
- 음악 스트리밍 사이트 등 오디오 형태를 재생할 경우 'origin', 'playback' 테이블에는 정보가 남으나 'playbackSession' 테이블에는 기록되지 않음
- 유튜브, 네이버 TV 영상 스트리밍 서비스에서 광고가 처음 혹은 중간에 나오게 될 경우 광고 시점, 광고 재생 시간, 본 영상 재생 시간 등 섬세한 정보가 남음

Media History 아티팩트는 생각보다 사용자의 미디어 재생 기록을 상세하게 기록하고 있었다. 이를 통해 앞으로 크롬 브라우저를 통한 유저의 미디어 재생 행위를 Tracking 할 수 있고 범죄 행위를 규명할 수 있는 수단이 될 것으로 판단된다.


## Reference
- <https://gotipath.com/chrome-kaleidoscope-upcoming-vod-aggregation-service-by-google/>
- <https://www.chromestory.com/2020/08/chrome-kaleidoscope/>
- <https://vibe.naver.com/>
- <https://www.youtube.com/watch?v=RyF_oA1KMTA>
- <https://tv.naver.com/v/16814658?query=bts&plClips=false:16705815:16768627:16787083:16842120:16761413:16809598:16700970:16617617:16373226:16299391:16302669:16303951:16283529:16792523:16236488:16229507:16200713:16121531:16052005:16020944:16039132:16814658:16802839:16038863:16812164:16038909:16204544:16038862:16030333:16039023:16038953:16039096:16039067:16211907:16199140:16774178:16797894:15557865:16835599:16802633>


## Copyright (CC BY-NC)
본 게시글은 CC BY-NC Licence를 따릅니다.  
비영리 목록으로만 사용할 수 있고, 저작자와 출처를 표시하면 언제든지 게시글을 자유롭게 사용할 수 있습니다.

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

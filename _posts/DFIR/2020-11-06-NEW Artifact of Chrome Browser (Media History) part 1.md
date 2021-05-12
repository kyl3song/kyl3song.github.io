---
title : "Blog #18: NEW Artifact of Chrome Browser (Media History) part 1"
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
크롬의 새로운 아티팩트 "Media History" part 1

## NEW Artifact of Chrome Browser Series
- [Blog #18: NEW Artifact of Chrome Browser (Media History) part 1](https://kyl3song.github.io/artifacts/NEW-Artifact-of-Chrome-Browser-(Media-History)-part-1/)
- [Blog #20: NEW Artifact of Chrome Browser (Media History) part 2](https://kyl3song.github.io/artifacts/NEW-Artifact-of-Chrome-Browser-(Media-History)-part-2/)


## Preface
약 한 달 전인 2020. 10. 6. Chrome 86버전이 릴리즈 되었다. 릴리즈된 새로운 버전에서는 Media History라는 이름의 아티팩트가 새롭게 추가되었다.

<p align="center">
  <img src="https://i.imgur.com/aiE1ChP.png" alt="image"/>
<br>그림 1. Chrome 86 Release Date</p>

Media History는 영상 또는 음성과 같이 미디어 재생 이력을 트래킹 할 수 있는 SQLite Database이다. 재생 이력에는 내가 방문한 URL부터 동영상의 재생 Position 정보, 제목, 최근 시청 일시 등 다양한 정보를 저장한다.

이번 포스트 시리즈에서는 Media History 테이블에 대한 전반적 이해와 몇 가지 테스트 시나리오를 바탕으로 남는 결과를 확인하고자 한다.


## What is Media History Database?
우리가 웹 브라우저를 통해 시청하는 동영상은 홍보, 교육, 재미 등의 목적으로 많이 사용된다. 이런 동영상을 보기 위해 찾는 대표적인 사이트는 유튜브일 것이다.

크롬 브라우저를 통해 유튜브와 같이 영상뿐 아니라 노래를 재생할 수 있는 음악 스트리밍 사이트에 접속해서 노래를 듣게 되는 경우에도 기록이 남을 수 있다.

최근 1년의 크롬 브라우저의 점유율은 약 64%로 많은 사용자가 크롬을 사용하고 있다. 이런 상황에서 크롬 브라우저의 아티팩트 변화는 증거분석을 하는 입장에서 매우 고마운(?) 변화이다.

<p align="center">
  <img src="https://i.imgur.com/BgPhBVK.png" alt="image"/>
<br>그림 2. Browser Market Share (Oct. 2019 - Oct. 2020)
<br>(https://gs.statcounter.com/browser-market-share)</p>

사실 내가 크롬의 새로운 아티팩트가 생겼다는 것을 접하고 가장 궁금했던 점은 따로 있었다.

예전부터 많이 있었지만 올해 이슈가 됐던 N번방, 박사방 등 디지털성범죄는 아직도 수사가 진행 중에 있다. [(관련 뉴스)](https://imnews.imbc.com/news/2020/society/article/5960303_32633.html)

특히 올해는 디지털성범죄 사건 관련하여 매체를 불문하지 않고 가장 많이 분석을 한 것 같다. 범죄와 관련된 사진, 동영상의 양은 정말 엄청날 정도로 많았다.

그중에 클라우드로 영상을 올리고 해당 링크를 메신저를 통해 공유되는 부분도 있어 영상을 스트리밍으로 볼 수도 있었다.

만일 이런 경우도 관련 기록이 남을지 궁금했다. 앞으로 디지털성범죄가 지속적으로 발생한다면 분명 혐의를 구증할 수 있는 좋은 아티팩트로 활용될 수 있기 때문이다.

### File Location
Media History 파일의 위치는 크롬 브라우저의 History, Cookies와 같은 기존의 아티팩트가 존재하는 기존의 경로와 동일하다.

**[경로]**
``` shell
%USERPROFILE%\AppData\Local\Google\Chrome\User Data\Default
```

<p align="center">
  <img src="https://i.imgur.com/18Ift7G.png" alt="image"/>
<br>그림 3. Media History 저장 경로</p>

## DB Table Overview
Media History DB 내부에 총 8개의 테이블이 존재한다. 그중 현재 의미가 있어 보이는 테이블은 'mediaImage', 'origin', 'playback', 'playbackSession', 'sessionImage'로 5개 정도이다.

<p align="center">
  <img src="https://i.imgur.com/Be2cq5l.png" alt="image"/>
<br>그림 4. Media History DB Tables</p>

8개의 테이블을 전체적인 틀만 파악해보면 나중에 도움이 될 수 있으므로 모든 테이블을 먼저 간단히 살펴보자. 테이블간 서로 연관되어 있는 부분도 있기 때문에 테이블간 연관성이 있다면 기초가 되는 테이블을 먼저 설명한다.


### 1. mediaFeed Table
크롬을 업데이트한 뒤 며칠이 지났으나 다른 테이블과 달리 이 테이블의 데이터는 아무것도 확인할 수 없었다.

<p align="center">
  <img src="https://i.imgur.com/7a2UXTb.png" alt="image"/>
<br>그림 5. mediaFeed Table</p>

일단 테이블명이 mediaFeed로 되어 있고 컬럼 중 'user_status', 'user_identifier'과 같이 유저의 정보가 있는 것으로 보아 크롬으로 미디어 서비스를 제공하기 위해 남기는 데이터들을 모아놓은 테이블로 추측을 해볼 수 있을 것 같다.


### 2. mediaFeedItem Table
이 테이블도 비어있는 상태이다. 그런데 이번 테이블에서는 컬럼명을 잘 살펴보면 무언가 유추는 할 수 있을 것 같다는 생각이 든다.

<p align="center">
  <img src="https://i.imgur.com/mcouT3j.png" alt="image"/>
<br>그림 6. mediaFeedItem Table</p>

컬럼명인 'name', 'author', 'date_published_s', 'is_family_friendly', 'content_rating', 'genre', 'tv_episode', 'play_next_candidate'만 살펴봐도 유튜브 영화, TV 시리즈 컨텐츠 서비스 관련된 테이블이라 생각할 수 있다.

앞선 테이블과 연관되어 향후 이 테이블을 활용해서 유저의 편의성, UX(User Experience)를 생각하는 것 같다. 그게 사업성이든 아니든 다시 한번 느끼지만 역시 구글은 구글이다.


### 3. mediaImage Table
동영상의 썸네일 URL이 기록되는 테이블이다. 처음에 URL을 보고 크롬에서 재생되는 모든 영상에 대한 썸네일을 웹으로 올려서 관리하는 것인가라고 생각했으나 데이터를 확인해보니 URL 모두 "https://i.ytimg.com/vi"로 시작을 한다.

<p align="center">
  <img src="https://i.imgur.com/vr7F7Mb.png" alt="image"/>
<br>그림 7. mediaImage Table</p>

ytimg는 youtube image를 줄인 단어로 유튜브는 YouTube Data API를 통해 동영상에 대한 정보와 썸네일을 예전부터 제공하고 있다. 그 썸네일이 저장된 URL이 DB 테이블 내 기록된 것이고 해당 URL로 접근하면 영상에 대한 썸네일을 확인할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/fBg8kKg.png" alt="image"/>
<br>그림 8. 재생한 동영상의 썸네일</p>



현재 확인한 것은 유튜브 영상만 URL이 확인되고 일반 개인 블로그에 유튜브 영상을 삽입한 형태여도 URL 값이 남는다.
'playback_origin_id'값을 통해 playback 또는 playbackSession 테이블에 기록된 영상 URL을 확인할 수 있다.

### 4. meta Table
meta 테이블에 딱 3개의 Row 데이터만 존재하는걸로 봐서 호환성, 버전 정보를 기재하는 테이블로 생각된다. 포렌식적인 큰 의미는 없어보인다.

<p align="center">
  <img src="https://i.imgur.com/Xd1uyAg.png" alt="image"/>
<br>그림 9. meta Table</p>


### 5. origin Table
origin 테이블은 뒤에 설명할 playback 테이블 내 기록된 레코드 중에서 최신 정보(시간 값), 병합된 정보(미디어 시청 시간)이다.

<p align="center">
  <img src="https://i.imgur.com/vhd8Oof.png" alt="image"/>
<br>그림 10. origin Table</p>

빨간색으로 하이라이트 된 유튜브 데이터를 설명하면 다음과 같다.

**origin**
- 유튜브를 통해 시청한 많은 영상의 URL은 각각 다르지만 그 기준이 되는 Base URL(https://www.youtube.com)은 같다. origin 컬럼은 Base URL 개념과 같다고 보면 된다.

**last_updated_time_s (e.g. 13248891473)**
- Base URL이 같은 영상 중 가장 최근 시청했던(음악이라면 들었던) 시간을 의미한다.  
- Safari(webkit), 구글 크롬 타임스탬프 포맷은 1601/1/1 00:00 UTC 기준으로 지나온 값을 microsecond로 사용하기에 해당 값을 적절히 변환하면 Human Readerable 시간으로 변경할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/1ZFuuQz.png" alt="image"/>
<br>그림 11. 크롬 시간 변환 결과
<br>(https://www.epochconverter.com/webkit)</p>

**aggregate_watchtime_audio_video_s (e.g. 21755)**  
- Base URL을 기준으로 총 재생한 미디어 시간을 의미한다. playback 테이블에서 기재된 재생시간을 모두 합친 값이라고 할 수 있다.


### 6. playback Table
playback 테이블부터는 의미 있는 데이터들이 저장되어 있다. 유저가 실제 접근한 URL을 기록하고 시청 시간(초), 비디오/오디오의 유무, 그리고 마지막으로 시청한 시간도 기록한다.

<p align="center">
  <img src="https://i.imgur.com/uqfxD8g.png" alt="image"/>
<br>그림 12. playback Table</p>

계산을 통해 실제로 시간이 맞는지 확인해 보자. 현재 빨간색으로 하이라이트 된 뉴스 영상의 시청 시간(watch_time_s)을 더하면 10초이다. 앞서 설명한 origin Table의 'aggregate_watchtime_audio_video_s' 컬럼의 값을 보면 **10**으로 되어 있음을 확인할 수 있다.


### 7. playbackSession Table
playback 테이블이 URL 정보, 시청 시간 등 전반적인 데이터를 담고 있다면 playbackSession 테이블은 보다 상세한 정보를 기록한다.

<p align="center">
  <img src="https://i.imgur.com/N1E52a7.png" alt="image"/>
<br>그림 13. playbackSession Table</p>

**origin_id (e.g. 888)**

origin_id는 origin 테이블의 기본키(id)와 연결되어 테이블의 관계를 식별할 수 있는 외래키이다. 다른 테이블과의 관계를 위한 컬럼이라고 생각하면 된다.

**URL (e.g. https://dfir.blog/~)**

방문한 URL로 playback 테이블과 동일하게 남기는 값이다.

**duration_ms (e.g. 2476101)**

동영상의 총 재생시간(ms)을 의미한다. 밀리초 단위이므로 변환을 해보면 맞아 떨어진다.

- 2476101 ms ➔ **41**.26835 minutes  
  0.26835 minutes ➔ **16**.101 seconds  
  **41분 16초**


<p align="center">
  <img src="https://i.imgur.com/RUrDqyc.png" alt="image"/>
<br>그림 14. SANS DFIR Summit 2020 - Ryan Benson
<br>(https://dfir.blog/unfurl-video-at-sans-dfir-summit-2020/)</p>

**position_ms (e.g. 1233110)**

동영상의 현재 재생 위치를 의미한다. 재생 위치를 기억하고 있다가 나중에 동일한 영상을 다시 재생할 때 유저가 마지막으로 본 위치에서부터 다시 재생을 할 수 있게 해준다. 현재 동영상은 20:33초에서 정지되어 있고 DB에 기록된 값을 변환하면 일치한다.
- 1233110 ms ➔ **20분 33초**

**last_updated_time_s (e.g. 13248830334)**

가장 최근에 미디어가 재생된 시간을 의미한다. 해당 값은 다른 테이블의 값과 일치하는 경우도 있으나 차이가 있을 수도 있다.

- origin Table(last_updated_time_s):  
  \- 13248828711 (2020-11-03 07:**11**:51)  
- playback Table(last_updated_time_s):  
  \- 13248828711 (2020-11-03 07:**11**:51)  
- playbackSession Table(last_updated_time_s):  
  \- 13248830334 (2020-11-03 07:**38**:54)

실험에서 재생된 영상의 경우 약 27분의 차이가 났다. 차이가 왜 나는 것일까? 이유는 간단하다. 해당 테이블은 동일한 URL에 대해 중복을 남기지 않고 기존의 레코드를 업데이트하는 방식이다. 즉, 영상의 마지막 상태 정보를 갱신한다는 의미이다.

### 8. sessionImage Table
동영상 썸네일의 크기를 기록하는 테이블이다. 'image_id'값으로 테이블에 기록된 데이터 모두 320x180 크기로 동일하였다.

<p align="center">
  <img src="https://i.imgur.com/aEpmz56.png" alt="image"/>
<br>그림 15. sessionImage Table</p>

'image_id'값을 통해 mediaImage Table의 'id'값을 매칭하면 썸네일을 확인할 수 있다.


## Wrap-up
지금까지 Chrome의 Media History에 대한 테이블과 일부 테스트를 통한 내부 저장된 컬럼들에 대해 전반적으로 확인하였다.

SQLite 테이블에 저장되는 값을 간단하게 정리하면 다음과 같다.
- **mediaImage:** 동영상의 썸네일 URL (현재까지 유튜브만 확인됨)
- **origin:** 재생한 영상의 Base URL, 최근 시청 일시, Base URL 기준 통합 시청 시간
- **playback:** 영상의 Full URL, 시청 시간, 영상/음성 포함 여부, 최근 시청 일시
- **playbackSession:** 영상의 Full URL, 영상의 길이(ms), 현재 재생 위치(ms), 레코드 업데이트 시간, 영상 제목, 업로드 닉네임(유튜브), Base URL
- **sessionImage:** 영상 썸네일의 크기

다음 포스트에서는 몇 가지 케이스를 정하고 DB에 어떻게 기록되는지를 확인할 예정이다.

## Reference
- <https://chromereleases.googleblog.com/>
- <https://meterpreter.org/google-is-developing-the-media-history-feature-in-chrome/>
- <https://dfir.blog/media-history-database-added-to-chrome/>
- <https://source.chromium.org/chromium/chromium/src/+/master:chrome/browser/media/history/>
- <https://webdir.tistory.com/472>
- <https://www.epochconverter.com/webkit>
- <https://linuxsleuthing.blogspot.com/2011/06/decoding-google-chrome-timestamps-in.html>
- <https://stackoverrun.com/ko/q/1735671>
- <https://gs.statcounter.com/browser-market-share>


## Copyright (CC BY-NC)
본 게시글은 CC BY-NC Licence를 따릅니다.  
비영리 목록으로만 사용할 수 있고, 저작자와 출처를 표시하면 언제든지 게시글을 자유롭게 사용할 수 있습니다.

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

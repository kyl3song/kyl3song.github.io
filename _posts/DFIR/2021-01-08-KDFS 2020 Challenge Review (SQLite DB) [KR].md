---
title : "Blog #23: KDFS 2020 Challenge Review (SQLite DB Recovery) [KR]"
category :
  - Digital Forensics Challenges
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - KDFS 2020
  - Digital Forensics Chanlleges
  - Media History
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
KDFS 2020 디지털포렌식 챌린지 리뷰 (SQLite DB 복원)


## This Post Covers

디지털포렌식 챌린지는 포렌식 분야에 익숙하지 않은 사람이거나 포렌식 기술들을 좋아하는 사람에게 좋은 기회이다. 대회 참가를 통해 기술의 트렌드를 파악할 수 있고 아직 많이 알려지지 않은 아티팩트를 대상으로 이것을 내가 어떻게 풀어낼 수 있을지를 확인할 수 있는 자리이기도 하다.

문제 중 잘 풀리지 않는 부분은 수많은 삽질(?)과 노력을 통해 해결 방안을 찾고 이를 보고서에 녹여 잘 작성된 리포트는 대회에서 좋은 성적을 낼 수 있도록 해준다.

이번 포스트에서는 KDFS 2020 챌린지의 증거물을 분석하면서 확인한 SQLite DB에 관한 내용을 살펴보고 어떻게 복원할 수 있을지 확인하도록 한다.


## KDFS Digital Forensic Challenge
KDFS 챌린지는 [한국디지털포렌식학회](https://kdfs.jams.or.kr/)에서 주최하는 대회로 약 1달간 주어진 시나리오를 대상으로 분석한 뒤 리포트를 제출하면 이를 평가하여 순위가 정해진다.

<p align="center">
  <img src="https://i.imgur.com/vvBDOtV.png" alt="image"/>
</p>

2020년에는 MicroSD를 분석하는 것이었고 어느 정도 분석이 진행되면 마약 사건임을 바로 알 수 있다.

<p align="center">
  <img src="https://i.imgur.com/7kVQul9.png" alt="image"/>
<br>[ Challenge Scenario ]</p>


## Getting into the Challenge
### com.android.chrome
먼저 com.android.chrome 등 여러 안드로이드 패키지명을 보니 MicroSD를 휴대폰에 삽입했던 흔적이다.

<p align="center">
  <img src="https://i.imgur.com/wRER67C.png" alt="image"/>
</p>

Chrome의 Top Sites, History 등 SQLite DB의 대부분 테이블이 정상적으로 보이는데 Media History DB는 테이블 자체가 보이지 않는다.

여기서 뭔가 임의적인 조작을 의심할 수 있다. 뭐가 되지 않거나 이상하다고 느끼면 의심부터 하는데 이건 어쩔 수 없는 직업병인 것 같다.

<p align="center">
  <img src="https://i.imgur.com/gz6V0lf.png" alt="image"/>
</p>

내부 데이터를 확인해 보니 역시 의미 있는 문자열들이 보이고 이것을 잘 복원하면 될 것 같다. 과연 어떻게 복원해야 할까?

<p align="center">
  <img src="https://i.imgur.com/hsgn2DM.png" alt="image"/>
</p>

### How to Recover Media History DB
크롬 Media History는 [Blog #18](https://kyl3song.github.io/artifacts/NEW-Artifact-of-Chrome-Browser-(Media-History)-part-1/), [Blog #20](https://kyl3song.github.io/artifacts/NEW-Artifact-of-Chrome-Browser-(Media-History)-part-2/)을 통해 다룬 적이 있다.

사실 나도 이 대회를 통해 Media History라는 아티팩트를 처음 접했고, 찾다보니 새롭게 추가된 아티팩트임을 알 수 있었다.

SQLite DB에서 유의미한 정보를 추출할 수 있는 방법은 여러 가지가 있다.

1. 문자열 추출  
   \- Strings, Bintext 등 도구를 이용
2. SQLite DB 복구 도구 사용 (상용/무료 )  
3. DB 구조 분석 및 직접 수정을 통한 복구

SQLite DB 특성상 문자열 추출의 방법은 일반적인 아스키 문자열은 확인이 가능하고 일부 고정 길이의 시간 값 등은 얼추 유추할 수 있으나 문자열의 경우 가변 길이를 많이 사용하기 때문에 개별 컬럼을 딱 구분 짓기 어렵다.

또한 챌린지 특성상 분석에 쏟아붓는 시간도 중요하지만 보고서 작성하는 데 시간을 더 많이 사용해야 하기 때문에 수동으로 일일이 구조를 분석하면서 하기에는 어려움이 있다.

나의 경우 회사일, 대학원 수업 및 시험, 집안일 등 다양한 사유로 챌린지에 집중할 수 있는 시간이 적었기 때문에 다른 방법이 필요했다. (이건 나뿐만 아니라 챌린지에 참여하는 사람 누구나 개인적인 사유가 있다고 생각된다)

내가 떠올린 방법은 **데이터 영역을 이식(Transplant) 하는 방법**이다.

### SQLite Structure Analysis
먼저 Media History DB는 크롬의 새로운 아티팩트이기도 하고 전혀 감이 없었기 때문에 데이터가 어떤 방식으로 남고 저장되는지를 파악할 필요가 있었다.

PC 크롬 브라우저 버전을 업데이트한 뒤 **1) 아무것도 하지 않은 클린한 상태에서 DB 파일을 수집**했다. 그리고 **2) 브라우저를 통해 미디어를 마구잡이로 재생한 뒤 다시 한번 DB를 수집**했다.

두 번째로 수집한 DB를 통해 데이터가 어떻게 남는지 포렌식적으로 의미 있는 데이터는 어디에 있는지를 확인했다.


|Leaf Page Offset  | Table Name          |
|:----------------:|:-------------------:|
|0x7000 ~ 0x7FFF   |Playback             | 
|0x9000 ~ 0x9FFF   |PlaybackSession      |
|0x10000 ~ 0x10FFF |mediaImage           |  



Leaf Page는 실제로 데이터가 저장된 SQLite Page를 의미한다. 여러개의 테이블 중 3개의 테이블이 의미가 있었고 챌린지의 구조가 깨져있는 Media History와 Leaf Page 오프셋이 동일하였다.

이제 내가 테스트용으로 생성한 정상적인 Media History DB에 Leaf Page를 통째로 이식하면 된다. 여기서 중요한 것은 그냥 무턱대고 가져다 붙여 넣으면 안되고 구조를 봐가면서 작업을 해야 한다.

Playback Table Leaf Page 구조를 우선 살펴보자. 데이터는 Big-endian으로 저장된다.

<p align="center">
  <img src="https://i.imgur.com/bCK48e3.png" alt="image"/>
<br>[ Playback Table Leaf Page 수정 전 ]</p>

수정이 필요할 부분이 두 군데로 보인다.

**수정할 부분:**

1. 셀(컬럼) 수: 24개(0x18)  
   \- 셀 오프셋 영역에서 확인한 2bytes씩 12개
2. 셀 시작 지점: 0x09CA  
   \- 셀 오프셋 영역의 마지막 레코드인 09 CA와 동일하게 맞춰줌  
   \- 셀 오프셋은 실제 레코드의 역순으로 저장된다는 이해 필요 ➜ [참고](https://www.ahnlab.com/kr/site/securityinfo/secunews/secuNewsView.do?menu_dist=2&seq=22438)

수정 후 구조의 모습은 다음 그림과 같다.

<p align="center">
  <img src="https://i.imgur.com/5uyACO8.png" alt="image"/>
<br>[ Playback Table Leaf Page 수정 후 ]</p>

동일한 방식으로 PlaybackSession 테이블 페이지를 수정하면 된다.

<p align="center">
  <img src="https://i.imgur.com/Uoagiy6.png" alt="image"/>
<br>[ PlaybackSession Table Leaf Page 수정 전/후 ]</p>

그런데 마지막 mediaImage 테이블의 경우 아래 그림과 같이 셀 오프셋 영역에 0C F3 헥사 값이 반복되는 패턴이 보인다.

<p align="center">
  <img src="https://i.imgur.com/dPUkjog.png" alt="image"/>
<br>[ mediaImage Table Leaf Page ]</p>

이런 경우는 보통 테이블의 행(Row)을 삭제한 경우 셀오프셋이 앞쪽으로 2바이트씩 당겨지기 때문에 이와 같은 패턴이 나타나게 된다.

실제 데이터 영역을 확인하니 총 6개의 컬럼 데이터 영역이 남아있다. (파란색이 구분선)

<p align="center">
  <img src="https://i.imgur.com/Q5F6kLP.png" alt="image"/>
<br>[ mediaImage Table의 데이터 영역 ]</p>

즉 위 mediaImage Table Leaf Page 그림에서 헥사 값 0C F3 묶음으로 6개의 동일한 패턴이 보이고 가장 마지막 0C 09는 이와 동일한 패턴이 아니므로 더 이전에 존재했었던 데이터의 셀 오프셋를 의미한다. 또는 문제 출제위원이 임의로 수정했을 가능성도 존재한다.

참고로 SQL Query를 통해 삭제 테스트를 해보면 이와 동일한 패턴을 확인할 수 있다.

```shell
sqlite> DELETE FROM mediaImage;
```

<p align="center">
  <img src="https://i.imgur.com/2lnBiIL.png" alt="image"/>
<br>[ mediaImage Table 행(row) 삭제 전/후 ]</p>

mediaImage 테이블은 문자열(URL) 이외에 따로 의미 있는 데이터가 있지 않다. 따라서 그냥 스트링을 추출하는 방법으로 해도 되는데 나 역시 이렇게 하였다.

```shell
$ strings Media History > mediaImage.txt
```

<p align="center">
  <img src="https://i.imgur.com/YiEK6LM.png" alt="image"/>
<br>[ mediaImage Table에서 추출된 문자열 ]</p>


### Leaf Page Data Transplant & Results

지금까지 수정한 Playback 및 PlaybackSession 테이블을 이식하는 작업만 남았다. 이식은 크롬을 업데이트 한 이후에 생성된 정상적인 DB 파일에다 붙여 넣는 작업만 하면 된다.

<p align="center">
  <img src="https://i.imgur.com/he3R5IA.png" alt="image"/>
<br>[ Media History DB 이식 프로세스 ]</p>

이렇게 수정된 데이터를 그대로 붙여 넣어주면 오른편 DB 파일의 구조는 정상적이므로 복원된 데이터를 손쉽게(?) 확인할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/fYGdlnc.png" alt="image"/>
<br>[ 복원된 Playback Table ]</p>


<p align="center">
  <img src="https://i.imgur.com/5CQ6rON.png" alt="image"/>
<br>[ 복원된 PlaybackSession Table ]</p>

복원된 테이블을 통해 용의자의 행위 추적을 좀 더 상세히 할 수 있었다.


## Wrap-up
Leaf Page에는 실제 데이터가 저장된다. 따라서 Leaf Page를 잘 가공하여 이식한다면 DB 전체 구조를 바꾸지 않더라도 데이터를 복원할 수 있다.

SQLite DB 이식의 방법을 정리하면 다음과 같다.
1. 구조 분석을 하여 복원하고자 하는 테이블의 Leaf Page 및 데이터 일부를 확인
2. 데이터를 해석할 수 있는 정보가 다르다면 수정 필요 (e.g. 셀 개수, 셀 시작점 등)
3. 정상적인 DB 파일에 수정된 Page 영역을 복사 및 붙여넣기 수행


## ETC - Little Talk
MicroSD 1개만을 가지고 삽입됐던 휴대폰의 번호를 특정할 수 있을까?

예전에 몰래카메라 기기를 어느 장소에 설치한 뒤, 그 장소를 다시 방문하여 영상이 저장된 MicroSD를 빼서 자기 휴대폰에 넣어서 영상을 확인하는 범죄자의 행위가 많았다.

MicroSD를 안드로이드 휴대폰에 삽입하면 앱의 패키지명으로 폴더들이 일부 생성되고 그중 일부 데이터들이 MicroSD 카드에 남게 된다.

**(분석관점)** 자동으로 생성되는 데이터 중 현재는 더 이상 사용하지 않지만 예전에 사용됐던 기본 앱에서 남는 파일 하나의 값을 분석 과정을 거치면 그 휴대폰의 휴대폰 번호를 알 수 있다.

**(수사관점)** 그럼 그 번호로 통신자료제공요청을 통해 피혐의자를 특정할 수 있다.

그런데 그 기본 앱이 사라진지 오래되어 현재는 거의 유용하지 않지만 가끔 MicroSD를 분석하는데 종종 보일 때가 있다.

내가 좀 더 공부하고 내가 좀 더 고생하면 억울하게 피해를 당한 사람에게 도움을 줄 수 있다. 지속적인 테스트도 하고 대회도 참여하면서 공부하는 이유이기도 하다.


## Reference
- <https://kdfs.jams.or.kr/>
- <https://m.etnews.com/20190918000193>
- <https://www.sqlite.org/fileformat2.html>
- <http://forensicinsight.org/wp-content/uploads/2013/07/INSIGHT-SQLite-데이터베이스-구조.pdf>
- <https://www.ahnlab.com/kr/site/securityinfo/secunews/secuNewsView.do?menu_dist=2&curPage=1&seq=22324>
- <https://www.ahnlab.com/kr/site/securityinfo/secunews/secuNewsView.do?menu_dist=2&seq=22438>
- <https://www.ahnlab.com/kr/site/securityinfo/secunews/secuNewsView.do?seq=22494>
- <https://www.ahnlab.com/kr/site/securityinfo/secunews/secuNewsView.do?seq=22583>
- <https://www.acquireforensics.com/blog/sqlite-database-structure.html>


## Copyright (CC BY-NC 2.0 KR)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

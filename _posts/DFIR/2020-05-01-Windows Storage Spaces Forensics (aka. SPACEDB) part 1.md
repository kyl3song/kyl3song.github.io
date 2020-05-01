---
title : "Blog #7: Windows Storage Spaces Forensics (aka. SPACEDB) part 1"
category :
  - DFIR
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - Storage Spaces
  - Storage Direct
  - Software RAID
  - SPACEDB
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
윈도우 저장소 공간 포렌식(SPACEDB) part 1

## 윈도우 저장소 공간 논문 소개 (Digital Forensics Research Paper Review)
윈도우 저장소 공간은 윈도우 OS에 탑재된 소프트웨어 레이드(Software Raid) 기능이다. 즉 다수의 디스크 드라이브를 여러개를 풀(Pool)로 구성하여 하나의 가상 디스크를 만드는 개념이다. ([그림 출처](<https://docs.microsoft.com/ko-kr/windows-server/storage/storage-spaces/storage-spaces-states>))
<p align="center">
  <img src="https://i.imgur.com/QEbqx5I.png" alt="image"/>
</p>

최근에 [치안정책연구](https://psi.jams.or.kr/co/main/jmMain.kci) 학술지에 "**파일시스템을 이용한 윈도우 저장소 공간 복원율 향상 방안**" 제목으로 논문을 작성하였다. 논문은 치안정책연구 홈페이지에서도 확인이 가능하고 [한국학술지인용색인(KCI)](https://www.kci.go.kr/kciportal/main.kci)에서도 검색해서 확인할 수 있다.

논문에서는 저장소 공간을 이루고 있는 다수의 디스크 중 일부 또는 다수가 파손되어 가상 디스크를 구성하지 못하는 경우, 단 1개의 디스크로 복원이 얼마나 가능한지를 확인한 연구 결과이다. 내용에는 저장소 공간의 동작 방식, 획득 방안, 분석 방안을 고루 다루려고 노력했다.

이번 포스트에서는 윈도우 저장소 공간 복원율 향상 방안에 대해 알아보고 논문에 기재하지 못했던 이야기, 그리고 기술 리포트에서 논문으로까지 작성하게 된 계기를 이야기하고자 한다.

<p align="center">
  <img src="https://i.imgur.com/jqZeYke.png" alt="image"/>
</p>

## 논문 작성의 계기 (What brings me do the research)
그동안 디지털포렌식 분석 업무를 수행하면서 수많은 저장매체의 증거물을 분석했다. 분석하면서 평소 분석 건과 다르게 특이했거나 어려움을 겪으며 흔히 말하는 빡쎈(?) 분석을 마무리했던 것들은 기억에 남는다.

윈도우 저장소 공간도 그중에 하나였는데 특이해서 기억에 남는 분석 건이었다. 사실 원래 윈도우 저장소 공간은 내가 배당받은 사건이 아니었다.  

내 기억이 맞다면 2019년 4월쯤 동료 분석관이 노트북을 1대 배당을 받았다. 노트북의 내부에 몇 개의 저장매체가 있었는지는 모르지만, 그중에 2.5 HDD를 탈거하여 획득(이미징)을 진행했다. 그런데 정상적으로 획득을 했음에도 불구하고 상용 도구에서 파일 시스템이 정상적으로 파싱되지 않는 현상이 발생했고, 동료가 '**SPACEDB**' 시그니처를 보고 나한테 연락해서 내가 최초 확인을 했었던 기억이 난다.

사실 난 SPACEDB 시그니처를 처음 본 게 아니었다. 2018년에 나는 Digital Forensics Challenges 2018 (KIISC)라는 포렌식 대회를 뛴 적이 있다. 내 생에 처음이자 마지막으로 참가한 포렌식 대회였다. 그 대회에서 파일 시스템과 관련된 카테고리인 'VOL' 카테고리에서 봤던 문제였다.
물론 나도 그 당시에는 처음 보기도 했고 인터넷을 찾아가며 문제에 접근을 했고 해결하려고 했던 기억이 난다.
<p align="center">
  <img src="https://i.imgur.com/um8pRy8.png" alt="image"/>
</p>

아무튼 SPACEDB 시그니처가 곧 Windows Storage Spaces(윈도우 저장소 공간)에서 생성된 디스크라는 것만 아는 상태였는데 실제로 이걸 증거물로 받는다는 생각하니까 소름이 끼쳤다.
설마 증거물로(?) 이걸 받을 줄이야 했던 안일한 마음이 있었다. 
그때 윈도우 저장소 공간에 대해 전반적인 이해가 필요하다고 생각했다. 이게 첫 계기가 된 것 같다.

## 획득 (Acquisition Workaround)
당시 내장된 디스크를 이미징한 이후 도구에 로드하였을 때 일반적인 파일 시스템이 파싱되어서 보여주지 않았다. 아래는 저장소 공간을 구성하는 디스크 중 일부를 도구에 로드한 결과로 당시 상황은 이와 동일했다.
<p align="center">
  <img src="https://i.imgur.com/9rLM1TH.png" alt="image"/>
</p>
당시 이미징 작업에 문제가 있었을까? 기존 이미징 작업과 동일하게 하드웨어 타입의 이미징 장비로 획득을 수행했으므로 획득하는 과정에는 큰 이슈가 없었다.

쓰기방지장치를 연결하여 매체를 직접 PC에 물려봤다. 역시나 상황은 다르지 않았다. 아무래도 이상하다고 마음속으로 생각했는데 혹시나 해서 다른 PC에 동일한 방법으로 연결을 하였다.
그랬더니 이번에는 자동으로 마운트까지 해주면서 폴더를 화면에 띄워줬다.

두 PC는 운영체제 버전을 빼고 차이가 크게 없는데 이상하다며 생각하는 순간 머리를 스친 생각이 있었다. 2018 포렌식 챌린지 대회 당시 윈도우 저장소 공간을 분석하려고 막 찾아봤던 기억이 났고, 당시 저장소 공간은 윈도우 8이상, 서버 2012부터 윈도우 운영체제에 새롭게 탑재된 소프트웨어 레이드 기능이라는 사실이 떠올랐다.

정상적으로 디스크를 파싱해준 윈도우 10 PC에서 바로 EnCase 툴을 통해 로지컬로 이미징 작업에 들어갔다.
당시에는 일단 되지 않던게 정상적으로 파싱됐으니까 급한 마음에 이미징부터 했던것 같다. 그 이후 분석은 동료 분석관이 했다. 획득이 정상적으로 됐으니 분석은 기존과 동일한 절차로 수행하면 됐기 때문에 획득을 도와주는 것으로 마무리 했다.


## 연구 내용 (Target Contents of Research)
당시 바로 연구를 진행했으면 좋았겠지만 사실 연구는 바로 진행하지 못했다. 시간적인 여유가 없었다고 하면 핑계지만 필자가 근무하는 곳은 사실 디지털 매체가 매우 많이 접수되는 곳이다. 즉, 획득에 어려움이 있어 2~3일을 잡아먹었다고 하면 그만큼 새롭게 들어오는 증거물들이 쌓이는 곳이다.

그럼에도 불구하고 틈틈이 들여다보기 시작했다. 사실 최소한 저장소 공간의 동작 방식이라도 알아두면 도움이 되기에 그 마음가짐으로 하나씩 확인을 해나갔다. 동일한 케이스가 앞으로 있다면 증거분석 업무에 필요한 부분을 생각하며 크게 3가지의 대분류를 잡고 소분류를 하나씩 추가하기로 생각했다.

- 저장소 공간 동작 방식 (How Storage Spaces Work)
- 획득 방안 (Acquisition)
- 분석 방안 (Analysis)
- 기타 등 (etc.)

저장소 공간 동작 방식에 대해 하나하나 확인하다 보니 문득 그런 생각이 들었다. 증거물로 의뢰되는 매체 중에 노후화 또는 고의적인 이유로 매체가 파손된 상태로 생각보다 많이 의뢰된다는 것이었다.

만일 저장소 공간에 사용된 HDD 중 파손의 이유로 가상 볼륨을 구성할 수 없는 경우에 정상적으로 파싱이 되지 않은 디스크의 모습을 볼 수 있게 된다. FTK Imager와 마찬가지로 EnCase에서 확인해도 비슷한 모습이다.
<p align="center">
  <img src="https://i.imgur.com/F9njxkQ.png" alt="image"/>
</p>

따라서 파손된 매체가 있을 경우를 대비하여 저장소 공간을 이루고 있는 개별(단 하나의) 디스크에서 복원율을 최대화하는 방법이 필요한 부분이었다.

다음 포스팅부터는 저장소 공간 복원율 극대화 방법에 대해 포스팅을 할 예정이다. 물론 논문에 작성된 모든 부분을 언급하지 않고 핵심이 되는 부분을 위주로 글을 작성할 예정이다.


## Reference
- <https://docs.microsoft.com/ko-kr/windows-server/storage/storage-spaces/storage-spaces-states>


## Copyright (CC BY-NC 2.0 KR)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
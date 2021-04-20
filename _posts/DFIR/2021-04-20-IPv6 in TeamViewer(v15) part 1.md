---
title : "Blog #27: IPv6 in TeamViewer(v15) Log part 1. [KR]"
category :
  - Artifacts
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - TeamViewer
  - IPv6

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
팀뷰어 포렌식 Tips


## This Post Covers
Teamviewer는 알려진 무료 원격제어 도구 중 사용이 간편해서 IT 종사자 뿐만 아니라 비종사자도 많이 사용하는 프로그램이다. PC에 많은 지식이 없는 친구, 부모님 등 컴퓨터에 설치하여 직접 방문하지 않고 원격을 통해 이슈를 해결할 수 있는 고마운 친구(?)이다.

팀뷰어의 로그에 대한 분석은 이미 예전부터 많은 사람들이 내용을 공개하였다. 구글에서 teamviewer forensics로 검색해도 많이 나온다. 하지만 버전이 업그레이드 됨에 따라 일부 구조가 변경되고 저장 경로가 바뀌고 한 것뿐이지 전체적인 로그의 틀은 동일하다.

최근 사건에서 사용됐던 동일한 버전에서 이것저것 분석을 진행하다가 우연히 IPv6 정보가 확인되어 두 편으로 나눠 포스팅을 남긴다.


## TeamViewer(v15.16.8) Logs
일반적인 수사/분석 관점의 경우는 피해자의 PC에서 가해자의 정보 또는 가해자와 연결될 만한 정보를 찾아내는 것이 중요하다. 이런 관점에서 본다면 팀뷰어에서 큰 의미가 있을만한 로그는 두 가지이다.

1. 원격 접속 로그: **Connections_incoming.txt**
2. 접속 상세 로그: **TeamViewer##_Logfile.log**
   
로그는 기본적으로 설치 폴더에 저장되며 x64 아키텍쳐에 x64 바이너리를 설치했다면 **C:\Program Files\TeamViewer**에 저장되고 x86 바이너리로 설치했다면 **C:\Program Files (x86)\TeamViewer**에 저장이 된다.

Fig 1.과 같이 Device A에서 B로 원격을 시도한 경우 다음과 같이 로그가 남는다.

<p align="center">
  <img src="https://i.imgur.com/oT76Dcs.png" alt="image"/>
<br>[ Fig 1. Remote Support Scenario from Device A to B ]</p>


## Connections_incoming.txt (UTC 0)
### 1. A's TeamViewer Autogenerated ID
A의 팀뷰어 아이디가 남는데 여기서 아이디는 프로그램이 원격 통신을 위해 자동으로 부여하는 임의의 숫자 값을 의미한다. 해당 값은 PC를 종료하거나 프로그램을 종료한 뒤 다시 실행시켜도 그대로 유지되는 값이며 보통 9자리나 10자리로 구성된다.

<p align="center">
  <img src="https://i.imgur.com/I7GVHb9.png" alt="image"/>
<br>[ Fig 2. Teamviewer remote control ID ]</p>

특이한 점은 테스트 당시 프로그램을 재설치 하더라도 값이 변하지 않았다. 컴퓨터 GUID 등 값을 이용해서 ID를 생성하는 것인지는 좀 더 판단이 필요하다.

### 2. A's Display Name or Teamviewer ID(if login)
해당 컬럼은 오해하기 쉬울 수 있다. 주위의 많은 사람들이 단순히 A의 컴퓨터 이름이 기록되는 컬럼이구나라고 흔히 알고 있었다. 그런데 조금 더 정확하게 살펴보면 일단 팀뷰어에 로그인 후에 원격 통신을 한 경우와 로그인을 하지 않고 원격 통신을 한 경우로 나눠진다.

만일 로그인을 하지 않고 프로그램을 사용하면, 팀뷰어는 원격 대상자(B)에게 보일 이름으로 **컴퓨터의 이름으로 초기 설정**된다. 그리고 이 값이 로그에 남는 것이다.

<p align="center">
  <img src="https://i.imgur.com/bt1pkhJ.png" alt="image"/>
<br>[ Fig 3. Display name ]</p>

만일 이 값을 임의로 변경하면 **Connections_incoming.txt** 및 **TeamViewer##_Logfile.log** 로그에서도 **변경한 값으로 로그가 생성**된다.

팀뷰어는 로그인해서 사용할 수도 있는데 로그인을 하면 자신이 자주 사용하는 원격 대상 PC 정보를 저장할 수도 있고, 해당 PC가 현재 켜졌는지 꺼졌는지도 확인할 수 있을 정도로 편한 인터페이스를 제공한다.

그럼 여기에서 팀뷰어 계정으로 로그인을 하게 되면 어떻게 될까? 
로그인을 하면 로그인 계정의 유저명이 우선순위가 더 높아 유저명으로 로그가 생성된다. 유저명은 역시 설정에서 변경이 가능하다.

아래 Fig 4. 그림으로 로그인을 하지 않을 때(왼쪽)와 로그인을 한 경우(오른쪽)로 나눠서 로그가 어떻게 남는지를 표현하였다.

<p align="center">
  <img src="https://i.imgur.com/6ELMP2E.png" alt="image"/>
<br>[ Fig 4. Log generated depending on login/logout status ]</p>

### 3. Remote Support Start Time (UTC 0)
원격을 시작한 일시이며 **dd-MM-yyyy hh:mm:ss** 형태로 기록되며, 중요한 것은 UTC 0로 로깅되니 로컬 타임에 맞춰 시간값을 보정해줘야 한다. 대한민국이라면 9시간을 더하여(UTC+9) 로그를 분석 및 판단하여야 한다.

### 4. Remote Support End Time (UTC 0)
원격을 종료한 일시이며 나머지 내용은 Start Time과 내용은 동일하다.

### 5. B's OS Account (Username)
헷갈릴 수 있으나 이 부분은 참고로 팀뷰어 로그인과는 상관이 없는 부분으로 원격을 받는 시스템 OS의 유저명을 의미한다. 그런데 여기서 한 가지 생각해 봐야 할 문제는 윈도우의 경우, 이메일로 로그인을 하는 경우와 로그인을 사용하지 않았을 때로 나뉠 것이다.

결론부터 말하면 확인해본 결과 MS 계정으로 윈도우 로그인을 하거나 로컬 계정으로 사용하거나 상관없이 무조건 **로컬 사용자 계정명**으로 기록이 남았다.

<p align="center">
  <img src="https://i.imgur.com/0OQFpSa.png" alt="image"/>
<br>[ Fig 5. OS Local Account in connections_incoming Log]</p>


## Wrap-up
Connections_incoming.txt 파일에서 원격으로 접속한 사람의 정보를 일부 확인할 수 있었다. 겉으로 보기에 단순해 보이는 로그로 좀 더 면밀히 살펴보면 판단의 오류를 줄일 수 있다.

1. Teamviewer Autogenerated ID  
  ➔ 팀뷰어에서 자체 생성되는 고유 ID값
2. Display Name or Teamviewer ID  
  ➔ 만일 이름이 PC명이 아니라면 팀뷰어로 로그인한 계정일 가능성 존재하기에 영장으로 가기 전 OSINT(Open Source Intelligence)를 사용할 수 있다.

다음 포스트에는 TeamViewer##_Logfile.log 로그에 대해 자세히 살펴보고 IPv6 정보 및 분석 팁이 될 만한 것들을 위주로 작성할 예정이다.


## Reference
- <https://www.systoolsgroup.com/forensics/teamviewer/>
- <https://www.dataforensics.org/teamviewer-forensics/>
- <https://benleeyr.wordpress.com/2020/05/19/teamviewer-forensics-tested-on-v15/>
- <https://www.champlain.edu/Documents/LCDI/archive/Team-Viewer-Forensics.pdf>


## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
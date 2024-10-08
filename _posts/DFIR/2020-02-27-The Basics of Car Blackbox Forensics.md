---
title : "Blog #2: The Basics of Car Blackbox Forensics part 1"
category :
  - Blackbox
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - Blackbox
  - Dashcam
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
차량용 블랙박스 분석 기본(Part 1)

## Blackbox Post Series
- [Blog #2: The Basics of Car Blackbox Forensics part 1](https://kyl3song.github.io/blackbox/The-Basics-of-Car-Blackbox-Forensics/)
- [Blog #3: The Basics of Car Blackbox Forensics part 2](https://kyl3song.github.io/blackbox/The-Basics-of-Car-Blackbox-Forensics-part-2/)
- [Blog #4: The Basics of Car Blackbox Forensics part 3](https://kyl3song.github.io/blackbox/The-Basics-of-Car-Blackbox-Forensics-part-3/)
- [Blog #5: The Basics of Car Blackbox Forensics part 4](https://kyl3song.github.io/blackbox/The-Basics-of-Car-Blackbox-Forensics-part-4/)

## Car Blackbox (Dashboard Camera)

차량용 블랙박스는 일반적으로 카메라를 통한 영상 녹화 및 기록 장치를 말한다.  
블랙박스는 여러 가지의 이름으로 불리는데 Dashboard Camera(Dashcam)라고도 하고 Car DVR(Digital Video Recorder)이라고 하기도 한다. 우리나라에서는 통상 Blackbox라고 불린다.

요즘은 자동차에 블랙박스를 누구나 설치해서 다니고 있다. 예전과 다르게 최근에는 블랙박스가 없는게 이상할 정도이다. 만일 없더라도 휴대폰 등 모바일 기기의 블랙박스 앱을 이용하여 영상을 촬영하는 경우도 많이 있다. 
실제로 필자의 경우에도 블랙박스 카메라가 고장나서 AS를 받을 동안 블랙박스를 대신하여 휴대폰을 차량에 거치하고 다녔던 경험이 있다.

그럼, 블랙박스를 설치하는 이유는 무엇일까?  
이유는 단순하다. 만일의 사태에 대비하여 영상 및 음성을 기록해놓는 것이다. 특히 자동차 사고 시 정확한 사고의 원인을 찾는 중요한 증거물이 될 수 있기 때문이다. 사람의 기억은 충분히 왜곡될 수 있는데 진술만으로 시비가 될 수 있는 부족한 부분을 보완해주는 역할을 할 수 있다. 
특히 대상자 간 진술이 대립될 때나 보험사의 경우 과실 비율을 계산할 때 중요한 역할을 할 수 있다.


## 파일시스템 (Filesystem)

블랙박스는 메모리를 삽입할 수 있는 메모리 슬롯이 있고 메모리 카드에 영상 파일을 저장한다. 제조사마다 다르지만 영상뿐만 아니라 메모리 카드에 로그파일을 남기는 블랙박스도 있다.

블랙박스의 상당수가 Microsoft의 FAT32 파일시스템을 채용하고 있다. 그 이유는 FAT32 파일시스템의 OS 범용성이 좋기 때문에 녹화된 영상 파일을 보는데 사용자 입장에서 익숙한 컴퓨터 환경을 그대로 사용할 수 있기 때문이다.
즉 사용자가 MicroSD 메모리를 컴퓨터, 휴대폰, 태블릿 등 다양한 기기에서 영상을 볼 수 있다는 장점이 있다.

아래는 EnCase에서 확인한 블랙박스 메모리 128GB MicroSD의 볼륨 정보를 확인한 것이다. 볼륨은 단일 볼륨으로 FAT32 파일시스템을 사용하고 있다.

<p align="center">
  <img src="https://i.imgur.com/GgJmqyR.png" alt="image"/>
</p>

윈도우를 조금 관심 있게 썼거나 64GB 이상 USB를 윈도우 운영체제에서 포맷해본 사람이라면 다 아는 내용이지만 윈도우에서는 기본적으로 64GB 이상의 저장매체를 FAT32로 포맷할 수 없게 되어있다.  
윈도우 탐색기에서 GUI 방식으로 포맷하는 부분은 못하게 했지만, 필요하다면 커맨드 명령어로 강제 포맷할 수 있다.
또 다른 방법으로는 GUI Format과 같은 도구를 이용하여 포맷을 진행하면 직관적이고 훨씬 수월하게 포맷이 가능하다.

본론으로 돌아와서 OEM Version으로 mkfs.fat가 보인다면 리눅스 시스템에서 포맷이 이뤄진 것으로 볼 수 있다. 즉 블랙박스 시스템에서 포맷이 이뤄졌다는 의미이다.

<p align="center">
  <img src="https://i.imgur.com/RhWtiFU.png" alt="image"/>
</p>

OEM ID는 VBR(Volume Boot Record)에 존재하는 값으로 해당 값을 이용하여 저장매체의 볼륨을 포맷한 운영체제를 대략적으로 판단할 수 있다. 아래는 각 운영체제에서 포맷했을 때 확인되는 OEM ID 값이다. 
- MSWIN4.0 (Windows 95)
- MSWIN4.1 (Windows 98)
- MSDOS5.0 (Win2k / XP/ Vista 이상)
- mkdosfs (Linux mkdosfs)
- mkfs.fat (Linux mkfs.fat / mkdosfs)

<p align="center">
  <img src="https://i.imgur.com/cMDJD13.png" alt="image"/>
</p>

만일 제출된 블랙박스 메모리카드의 OEM ID가 MSDOS5.0 등과 같이 윈도우 시스템에서 포맷된 형태라면 일단 의심해볼 여지가 있다.

참고로 **mkfs.fat**은 리눅스의 FAT 파일시스템 포맷 명령어인데 리눅스에서 한 줄의 명령어로 간단히 FAT32로의 포맷이 가능하다.
``` shell
mkfs.fat -F 32 /dev/sdb1
```

블랙박스가 FAT32 파일시스템을 사용하는데 있어 장점만 있는 것은 아니다. 자동차 사고가 발생했다고 가정하자. 사고가 발생하게 되면 갑작스러운 충격이 발생할 것이다. 그럼 이로 인해 블랙박스 기기의 전원이 차단될 수도 있고, MicroSD 메모리 카드가 충격에 의해 기기에서 분리되는 등 다양한 상황이 발생될 수 있다.
누군가는 '충격에 의해 MicroSD가 분리된다고?'라는 의문을 가질 수 있다. 실무상 보면 이런 케이스가 생각보다 많다. 하지만 지금은 많이 보완된 것 같다.

충격으로 인해 메모리가 분리된다면 마지막에 저장되는 영상(즉, 사고 당시의 영상)은 저장되지 않을 수 있다는 치명적인 단점이 존재한다.

보통 영상이 저장될 때 프레임이 순차적으로 저장이 되고, 마지막 프레임이 저장 되었을 때 비로소 파일로 완성이 된다. 비유를 하자면 인터넷에서 파일을 받는 것과 비슷하다. 크롬 브라우저에서 파일을 다운로드하는 중간에는 ***.crdownload*** 확장자를 가지며 다운이 완료되면 파일로 완성이 되는 것과 비슷한 개념이다.

그런데 블랙박스의 경우 이와 같은 방식으로 임시파일 형태로 영상 프레임을 차곡차곡 쌓다가 비정상적으로 전원이 나가는 상황이 발생할 경우, 파일이 완성이 안됐기 때문에 그 상태로 끝나버리는 것이다.
기기 내부적으로는 임시파일(.TMP) 형태로 남는 경우도 있는데, 물론 제조사마다 다르고 모델마다 다르지만, 실제로 파일시스템을 분석하면 정상적으로 영상 파일이 있음에도 불구하고 임시 파일이 그대로 남아 있는 경우를 종종 목격할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/GZCOqT9.png" alt="image"/>
</p>
   
이 경우에는 임시 파일 형태로 영상파일을 저장하다가 완료가 되면 정상적인 영상 파일로 덮어쓰는 형태일 것이다. 덮어쓰기가 완료되면 사용됐던 임시파일은 파일시스템에서 삭제가 되는데 그 과정에서 디렉터리 엔트리의 삭제 플래그 값만 변경하기 때문에 파일이 그대로 남아 있는 경우가 다수이다.

사고 당시의 영상이 저장되지 않는 부분이 단순한 단점일지 모르나 사실 가장 큰 문제로 작용한다. 만일 사고가 발생했는데 충격으로 영상이 저장이 되지 않았다? 그렇게 큰 문제가 아닐 수 없다. 사용하는 블랙박스의 신뢰성에도 문제가 된다.

이 문제점을 보완하기 위해 최근에는 포맷프리(Format Free)라는 저장 방식을 이용하는데 블랙박스에서 포맷프리 방식을 사용하는 경우가 점점 많아지고 있다.
포맷프리 방식은 초기에 많이 사용되었던 TAT(Time Allocation Table) 방식부터 현재는 NxFS 및 FAT32 기반 포맷프리까지 사용되고 있다.


## 포맷프리(Format Free)

다수의 블랙박스 제조사에서 포맷프리 장점에 대해 크게 3가지로 요약하고 있다.
- 녹화 파일 생성 시 데이터를 분산하지 않고 균일하게 저장
- 쓰기(Write) 회수를 최소화하는 기술을 적용하여 메모리 카드의 기본 수명 및 효율성 높여 메모리카드를 포맷하지 않고 오랜 기간 사용 가능
- 녹화 중 외부의 영항으로 강제로 메모리가 제거되거나, 사고 충격으로 인한 녹화 파일 손상 방지

포맷프리 저장 방식은 실시간으로 녹화되는 영상 프레임을 메모리 카드에 실시간으로 순차적으로 저장하는 방식이다. FAT 방식에서의 약 30초 ~ 1분 단위로 파일이 저장되는 방식이 아닌 영상 프레임을 순차적으로 저장하기 때문에 메모리 카드의 단편화를 최소화할 수 있다. 따라서 충격이나 전원 차단 시 영상 파일의 손실 위험은 줄어드는 장점이 있다.

포맷프리 방식에도 단점이 있는데, 사용자가 윈도우 탐색기에서 영상파일을 볼 수 없다는 점이다. 그리고, 영상을 재생하려면 곰플레이어 / 팟플레이어 / KMP등 과 같은 영상플레이어에서 볼 수 없다는 점이다. 물론 제조자의 전용 플레이어를 다운로드 및 설치하면 재생이 가능하다.

이런 사용자 친화적이지 못한 단점을 보완하고자 제조사에서 제공하는 전용 뷰어 없이도 영상을 탐색할 수 있고 재생할 수 있는 FAT32 기반 포맷프리가 나오게 되었다.


## 영상 복원의 단계(Recovery Plan)

파일시스템은 파일을 효율적으로 관리하기 위해 만든 시스템이다. 즉, 다시 말하면 파일시스템을 분석한다는 것은 파일이 저장된 클러스터 위치, 사이즈, 파일명 등 모든 정보를 가시화할 수 있다는 것을 의미한다. 그렇기 때문에 복원은 <span style="color:red">**1) 파일시스템 레벨에서 하는 것이 가장 좋은 방법**</span>일 것이다.

만일 파일시스템이 손상되거나 복원할 수 없는 경우에는 <span style="color:red">**2) 파일 그 자체의 특성을 이용하여 카빙으로 복원하는 방법**</span>이 최선이고, 
그마저도 안될 때는 <span style="color:red">**3)영상 프레임별 복원**</span>을 하는 것이 가장 마지막 단계이다.

위에서 언급한 1, 2 단계까지는 대부분의 파일을 복원하는 단계일 것이다. 영상 복원은 세 번째 단계가 추가되는데 사실 세 번째 단계가 가장 중요한 단계이다.
깨끗한 영상 복원을 위한 파이널 터치(?) 단계를 의미이기도 하다.

하나의 일화로, 예전에 택시 블랙박스 메모리 카드를 분석한 적이 있었다. 분석하는 과정에서 최근 며칠 사이 촬영된 영상 파일(활성 영상) 몇 개만 확인되는 것이었다. 택시라면 분명 차량 운행시간도 많고 블랙박스에 저장된 영상이 많아야 할 것 같은데 뭔가 이상하다 싶어 일단 합리적인 의심을 하고 프레임 단위로 복원을 시작하였다.   
결론적으로 프레임 복원결과, 삭제된 영상 프레임이 복원됐을뿐 아니라 해당 제조사 블랙박스 화면이 아닌 다른 제조사의 블랙박스 영상프레임까지 복원이 된 적이 있었다. 사실 이렇게 되면 둘 중에 하나이다.
- **(의도적)** 다른 차량의 블랙박스의 메모리 제거 > 본인 차에 삽입 > 며칠 촬영 > 메모리 카드 제출
- **(우연적)** 블랙박스 기기를 변경하였지만 동일한 메모리 카드를 사용

필자는 또 다른 의심가는 부분과 연계하여 의도적 증거인멸 부분에 생각의 힘을 실었다. 분석결과보고서에는 내 개인적인 판단을 직접적으로 언급하지는 않았다. 사실 할 수도 없고 해서도 안된다. 분석보고서는 팩트 기반으로 작성되어야 하기에 분석하면서 발견한 것, 확인되는 것 사실 위주로만 작성되어야 한다.

이 사건은 과연 어떻게 됐을까? 처벌은 받았을까? 사실 마무리는 어떻게 됐는지는 잘 모른다. 디지털증거 분석 과정을 거치게 되면 그 내용을 바탕으로 수사가 더 진행되어야 하기 때문이다.  
이 사례와 같이 영상에 대한 이해는 영상을 복원하는 기술적인 부분뿐 아니라 프로파일링 하는데 있어 매우 중요하다.

## Reference
- EnCase Computer Forensics: The Official EnCE: EnCase Certified Examiner Study Guide
- <https://kb.iu.edu/d/bccm>
- [INAVI Format Free Description](http://www.inavi.com/Community/TodaysContents/View?idx=37)
- [Time Allocation Table Description](https://m.blog.naver.com/PostView.nhn?blogId=techno001&logNo=220955022622&proxyReferer=https%3A%2F%2Fwww.google.com%2F)

## Copyright (CC BY-NC)
본 게시글은 CC BY-NC Licence를 따릅니다.  
비영리 목록으로만 사용할 수 있고, 저작자와 출처를 표시하면 언제든지 게시글을 자유롭게 사용할 수 있습니다.

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
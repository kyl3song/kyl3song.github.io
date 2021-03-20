---
title : "Blog #26: Importance of Drive Trim in Forensic Imager part 2. [KR]"
category :
  - Acquisition
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - Falcon-NEO
  - Mirror Clone
  - Drive to Drive
  - Drive Trim
  - EnCase 20.4
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
디스크 복제에서의 Drive Trim의 중요성 part 2.


## This Post Covers
지난 글에서 특정 LBA 크기만큼 해시값을 계산할 수 있는 기능을 확인하였다. 이를 통해 원본(4GB USB) 및 사본 디스크(1TB HDD)의 크기가 서로 다른 상황에서 Drive Trim을 비활성화한 상태로 디스크 복제한 경우(Mirror Clone), 원본 디스크의 LBA 크기만큼만 해시값을 계산하는 옵션을 주어 해시 검증을 직접 해보았다.

이번엔 해시값이 서로 일치한다는 가정하에 분석을 어떤 식으로 하면 좋을지를 살펴볼 것이고 또한 **Drive Trim을 사용할 때 가장 중요한 부분**을 설명하고자 한다.


## How to Do an Analysis with Mirror-cloned Disk w/o. Drive Trim
복제 형태로 획득된 사본 디스크를 어떻게 분석해야 할까? Drive Trim 옵션도 설정되어 있지 않은 상태라 사본 디스크를 연결하면 1TB 전체로 인식이 될 것이고 EnCase, X-Ways Forensics와 같은 도구에서 Process, RVS(Refine Volume Snapshot)를 돌리자니 1TB가 다 돌아가는 것 같고, 정작 내가 필요한 부분은 원본의 딱 4GB 영역인데 말이다.

크게 두 가지로 형태로 나눠볼 수 있을 것 같다.

### Case 1. Destination is not wiped before mirror clone
여기서의 정확한 표현은 사본 디스크에서 복제된 4GB 이외의 영역이 와이핑 되있냐는 의미이다. 만일 예전에 사용했던 디스크를 와이핑 하지 않은 상태에서 복제하였다면 예전의 데이터가 복원될 가능성이 있고, 원본의 분석 대상과 섞일 수 있기 때문에 이런 경우에 분석은 의미가 없다.

그럼 만일 이전 데이터가 남아있다는 가정하에 생각해 볼 문제는 분석 도구에서도 특정 영역만 지정하여 해시값 검증이 가능한지, 특정 영역만 분석이 가능한지 여부가 중요하다.

**1. Possibility of hash verification based on LBAs in tools**

포렌식 도구에 획득된 사본을 로드하면 해시 검증작업을 거쳐 검증된 해시값을 보고서에 넣을 것이다. 대부분의 포렌식 실무자들은 이렇게 업무를 하는데 도구에서 특정 LBA까지 해시 검증을 할 수 있는 기능이 있는지도 중요하다.

EnCase v20.4의 경우 특정 LBA까지 해시 검증을 할 수 있다. 그런데 여기서 중요한 것은 도구마다 LBA 카운트로 기준을 잡을 수도 있고 섹터 시작/종료 지점을 기준으로 잡을 수 있다는 점이다.

즉, Falcon-NEO에서 LBA Count가 **7,831,552**라고 되어 있어도 막상 EnCase 도구에서 동일한 숫자를 입력한 뒤 해시 검증을 하면 맞지 않는다. 0 ~ 7831551 sector가 전체 7,831,552개의 LBA이기 때문에 EnCase에서는 Fig 1.처럼 Stop sector에 **LBA Count-1** 값인 **7,831,551** 이라고 입력해야 해시값이 정확히 일치한다.

<p align="center">
  <img src="https://i.imgur.com/sjmTmAi.png" alt="image"/>
<br>[ Fig 1. Drive Hash with LBA Counts ]</p>

만일 다른 도구에서 디스크 해시 검증하는 옵션에 시작/종료 섹터를 입력하는 것이 아닌 LBA 개수를 입력하는 것이라면 Falcon 로그에서 보여준 수치대로 입력해야 된다. Fig 2는 검증이 정상적으로 된 모습이다.

<p align="center">
  <img src="https://i.imgur.com/y1npFv6.png" alt="image"/>
<br>[ Fig 2. Hash Verified by EnCase v20.4]</p>


**2. Possibility of an anlaysis within LBAs in tools**

도구는 프로세스 기능을 통해 일반적인 부분을 자동화하여 분석해 준다. 그렇지만 대부분의 도구에서 특정 LBA까지만 프로세싱 작업을 할 수 있는 옵션을 가진 도구는 거의 없을 것이다.

물론 방법은 아예 없는 것은 아니다. 스크립트를 작성하여 특정 영역만 스캔할 수 있는 방법은 있으나 신경 써야 할 것도 많고 많은 노동이 들어가는 일이다.

**3. Best Practice in analysis phase**

따라서 Case 1 상황에서는 정해진 범위 내 분석이 일반적으로 어렵기 때문에 최선의 방법은 원본 영역만 이미징을 다시 획득하는 방법이다.

즉 1TB의 사본 디스크에서 원본 영역(4GB) 만큼 LBA를 지정하여 e01로 다시 획득하면, 획득된 이미징 파일을 가지고 분석을 진행하는 게 가장 최선의 방법이 아닐까 싶다. 물론 획득을 한 번 더 수행한 것에 대한 것은 보고서에 남겨야 할 것이다.


### Case 2. Destination is wiped before mirror clone
두 번째 케이스는 사본 디스크에서 복제된 4GB 이외의 영역이 와이핑이 되었다는 의미이다. 이 경우에 1TB 용량의 사본 디스크를 프로세싱 돌리면 크게 문제 될 것 같지는 않다. 하지만 분석하는데 쓸모없는 시간이 좀 더 걸릴 것이다.

도구에서는 아무래도 나머지 와이핑 되어있는 영역까지 슬랙으로 판단하기 때문에 바이트 레벨로 카빙하는 경우 시간이 오래 걸릴 수 있다. 따라서 여기에서도 최선의 방법은 원본 영역만 이미징 파일로 획득하는 방법일 것이다.


## Cautions for Drive Trim
Drive Trim을 사용할 때 반드시 알아야 할 점이 있다. 먼저 상황을 살펴보자.

Drive Trim 기능을 테스트를 했을 때 Fig 3.처럼 Drive Trim 부분을 보면 1TB의 사본 HDD가 정상적으로 옵션이 설정되어 복제된 것을 Pass 결과로 확인할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/FtIuoBn.png" alt="image"/>
<br>[ Fig 3. Falcon Log (Mirror Clone with Drive Trim option) ]</p>

이번에는 SATA 형태의 HDD가 아닌 USB 외장하드를 통해 디스크 복제를 할 것이다. 좌측에는 원본 4GB USB, 우측에 250GB Samsung 외장하드를 사본쪽에 연결하였다.

<p align="center">
  <img src="https://i.imgur.com/BQoPGQ6.png" alt="image"/>
<br>[ Fig 4. Falcon Neo with Source(4GB, left) & Destination(250GB, right) ]</p>

그리고 Drive Trim의 기본 옵션이 No(disable)로 되어있는 것을 Yes로 변경하고 복제를 시작하였다.

<p align="center">
  <img src="https://i.imgur.com/yxAFEto.png" alt="image"/>
<br>[ Fig 5. Mirror Clone Option Menu ]</p>

복제 작업이 완료된 후 로그를 살펴보니 **USB_D1**로 정상적으로 USB 타입의 외장하드가 잘 연결된 것을 확인할 수 있었고 용량도 250GB로 정확히 인식되었음을 알 수 있다. 그런데 Drive Trim 관련 로그를 보면 **Type: Not Applicable**로 결과가 **<span style="color:red">Skipped</span>** 되었다는 것을 확인할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/EMaGvSn.png" alt="image"/>
<br>[ Fig 6. Drive Trim not applicable with USB portable HDD ]</p>

혹시 몰라 250GB 외장하드를 PC에 연결해봐도 250GB 전체로 인식하였다. 로그에서 나타난 대로 Drive Trim 옵션이 적용되지 않은 상황이었다.

### Why not applicable & skipped?
Drive Trim 옵션은 ATA 인터페이스로 통신할 때만 적용된다. 첫 번째 글에서 언급한 것처럼 디스크의 크기를 논리적으로 변경하기 위해 HPA(Host Protected Area), DCO(Device Configuration Overlay) 영역을 조작해야만 한다. 이는 명령어 세트를 이용하여 디스크 컨트롤러와 직접적인 통신을 하기 때문에 가능한 것이다.

- HPA: SET MAX ADDRESS
- DCO: DEVICE CONFIGURATION SET / DCO MODIFY
- ACS3: ACCESSIBLE MAX ADDRESS (ATA/ATAPI Command Set)

따라서 USB, PCIe 등 기타 포트나 SAS Drive의 경우에는 Drive Trim 기능이 적용되지 않는다.

일반적으로 USB 타입의 외장하드가 휴대하기도, 사용하기도 편하기 때문에 이를 가지고 디스크 복제를 했다면 Drive Trim 기능을 선택했어도 적용되지 않기 때문에 반드시 알고 있어야 한다.

Falcon-NEO 매뉴얼에서는 다음과 같이 설명하고 있다.

> Falcon-NEO Manual(Drive Trim):  
> Drive Trim only works with ATA drives connected to the 
SAS/SATA Destination ports. Drive trim will not work with 
SAS drives or drives connected to the USB, PCIe, or I/O ports.


## Wrap-up
지금까지 Drive Trim 옵션이 적용되지 않은 디스크 복제 방식으로 획득했을 때 분석하는 최선의 방법과 Drive Trim 기능을 사용할 때 필수적으로 알고 있어야 할 부분에 대해서 설명하였다. 다시 한번 짧게 정리하고 글을 마친다.

### 1. Method of Hash verification in tools
- 기본적으로 LBA Count로 원하는 블록만 해시 검증이 가능
- LBA Count 인지 Sector 시작, 종료 지점인지 도구에 맞는 단위를 파악하고 LBA 숫자를 입력하는데 주의 필요

### 2. Best Practice in anlysis phase
- LBA Count 만큼 블록을 지정하여 자동 분석을 할 수 있는 기능은 대부분의 도구에서 구현이 되지 않아 가능하면 해당 블록만큼 이미징 파일을 다시 한번 획득하여 분석할 수 있음
- 분석 보고서에 재 사본을 한 사유, 어떻게 했는지 등에 대한 상세한 설명 기재

### 3. Cautions for Drive Trim
- ATA 통신으로 디스크 컨트롤러에 명령어 세트를 이용하여 HPA/DCO/ACS3를 조정하는 기능임
- SAS 드라이브나, USB 포트, PCIe 등 다른 포트로 연결된 드라이브에는 Drive Trim 옵션이 적용되지 않아 주의가 필요함


## Reference
- <https://www.logicube.com/wp-content/uploads/2019/10/MAN-Falcon-NEO-v2.3-FIN-2.pdf>
- <https://www.logicube.com/wp-content/uploads/2017/08/MAN-Falcon-v3.2-FIN.pdf>
- <http://forensic-proof.com/archives/284>
- <http://www.ktword.co.kr/abbr_view.php?m_temp1=2108>
- <https://en.wikipedia.org/wiki/Host_protected_area>


## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

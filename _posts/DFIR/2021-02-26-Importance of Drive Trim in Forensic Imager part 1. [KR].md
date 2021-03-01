---
title : "Blog #25: Importance of Drive Trim in Forensic Imager part 1. [KR]"
category :
  - Acquisition
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - Falcon-NEO
  - Mirror Clone
  - Drive to Drive
  - Drive Trim
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
디스크 복제에서의 Drive Trim의 중요성 part 1.


## This Post Covers
최근에 사건을 접수 및 배당을 받았다. 증거물은 피의자 컴퓨터 SSD, HDD를 각각 이미징한 파일이 저장된 외장하드 2개였고 팔콘 장비로 이미징 후 생성된 로그와 필요한 서류를 바탕으로 접수가 되었다.

가장 먼저 이미징이 정상적으로 획득됐는지 외장하드에 쓰기방지장치를 연결하여 확인을 하였다. 근데 윈도우 상에서 볼륨이 자동 마운트 되면서 Program Files, Windows 폴더 및 파일들이 보였고 원래 예상했던 E01, Ex01과 같은 이미징 파일은 없었다.

수사관이 착각해서 접수할 때 다른 외장하드를 가져왔나 생각이 들어 나머지 외장하드 1개도 동일한 방식으로 확인하였더니 역시 마찬가지로 폴더만 잔뜩 보이는 것이었다.

팔콘 로그를 천천히 살펴보니 **Drive to Drive(MirrorClone)** 방식이다. 이번엔 로그에서 Trim 여부를 살펴보니 아니나 다를까 **False**로 되어있다.

해시값 검증을 해봐야 알겠지만 느낌상 이미징 작업을 다시 해야할 수 있다는 생각이 밀려왔다.
결론적으로는 산출된 해시값이 로그상 확인되는 해시값과 일치하지 않아 다시 이미징 작업 요청을 하였다.

이번 포스트에서는 ***Drive Trim*** 기능에 대해 확인하고 왜 필요한지에 대해서 살펴보도록 한다.


## Imaging Mode
디스크를 복제하는 방법은 여러 가지가 있다. 물론 이미징 장비마다 부르는 명칭이나 기능적인 차이가 있으나 [Falcon-NEO](https://www.logicube.com/shop/forensic-falcon-neo/?v=38dd815e66db)를 기준으로 설명하면 다음과 같다.

<p align="center">
  <img src="https://i.imgur.com/7GUkBjY.png" alt="image"/>
<br>[ Fig 1. Imaging Mode in Falcon-NEO ]
<br>(Source: Logicube Forensic Falcon-NEO User Manual) </p>


### 1. Drive to File
Drive to File 방식은 우리가 흔히 사용하는 하드디스크를 이미지 형태의 파일(DD, E01, Ex01 등)로 만드는 방식이다. 현업에서 가장 많이 사용하는 방식으로 특별한 사정이 있지 않는 한 이 방식을 선택한다. 흔히 이미징이라고 불린다.

### 2. Drive to Drive (Mirror Clone)
원본 디스크 드라이브를 비트 레벨로 사본 디스크에 복제하는 방식으로 디스크가 완벽히 복제되는 방식이다. 1번과의 차이는 1번은 파일 형태로 만드는 것이고 2번은 그냥 동일한 디스크가 하나 더 만들어지는 개념이다. 이번 글 내용과 관련 있는 옵션이니 눈여겨보면 좋을 것 같다.

### 3. File to File
사실 3번부터는 잘 사용되지는 않지만 그래도 짚고 넘어가고자 한다.

이 모드는 소스 드라이브에서 특정 확장자를 가진 파일만 선별하는 등 필터링을 통해 특정 파일들을 획득하는 방식이다. L01, Lx01, ZIP 등 형태로 파일이 만들어지고 Logical Imaging 방식이다.

### 4. Partition to File
소스 드라이브의 특정 파티션만 획득하는 방식으로 E01, Ex01, DD 등 형태로 이미지를 만들 수 있다. BitLocker가 설정된 파티션의 경우에도 비밀번호를 입력하면 복호화 해서 이미지를 생성해 주기도 한다. 역시 Logical Imaging 방식이다.

### 5. Net Traffic to File
Falcon-NEO에서는 네트워크 트래픽을 수집하는 기능도 제공한다. 네트워크 패킷을 수집하여 .pcapng 파일로 저장된다. 이 기능은 단 한 번도 사용한 적은 없는데 장비에서 제공하는 만큼 수집이 종료되면 해시값이 포함된 로그도 생성해 주지 않을까 생각한다.

(To-Do List) 추후 테스트를 해서 업데이트를 해보도록 하겠다.

### 6. File to Drive
획득한 이미지 파일(DD, E01, Ex01 등)을 디스크로 복원(Restore)해주는 개념이다. 나는 이 기능을 되게 유용하게 한번 써먹은 적이 있었다.

예전에 친한 형(수사관)이 증거물 원본 디스크를 자신의 PC에 붙여서 잠깐 파일 스캔 작업을 할 필요가 있었는데 이때, 쓰기방지장치가 문제가 됐었는지 디스크의 파티션이 다 날라가서 부랴부랴 안된다고 이거 원본이라며 제발 살려달라고x10 부탁을 한 적이 있었다.

당시 일단 내부 구조를 살펴보니 전체적으로 1~2 섹터가 밀려있는 현상이 발생되었고, 정확히 어떤 것이 그런 현상을 만들었는지 원인은 파악하지 못했다. 그렇다고 이걸 직접 수정하기엔 이슈가 있을 수 있었고, 획득된 이미징 파일이 있으면 그 파일로부터 Restore가 된다는 것을 알고 있었기에 복원시켜줬다. 물론 해시값도 원래 상태 그대로 검증되었다.

말이 길었지만 이럴 때 유용하게 사용할 수 있다.


## What is Destination Drive Trim?
오늘 주제인 디스크 복제에 대해서 좀 더 살펴보자. DVR(CCTV) 분석 등 복제본 디스크로 시스템을 부팅시켜 동작 방식 등 확인해야 될 필요가 있는 경우 원본을 이용하는 것이 아닌 복제본 디스크 만들어 장착 후 사용하게 된다.

이와 같이 디스크 복제 모드를 사용할 경우 사본 드라이브(Destination Drive) Trim이라는 옵션이 있다.

<p align="center">
  <img src="https://i.imgur.com/IuxWzlo.png" alt="image"/>
<br>[ Fig 2. Drive Trim Option ]</p>

Drive Trim 기능은 디스크 복제 모드에서만 활성화가 되는데 Falcon과 같은 이미징 장비가 사본 디스크를 대상으로 **DEVICE CONFIGURATION SET** 명령어를 이용하여 DCO(Device Configuration Overlay) 영역을 조작하거나 **SET MAX ADDRESS** 명령어를 이용하여 HPA(Host Protected Area) 영역을 조작하는 기능이다.

사본 디스크의 HPA, DCO 영역을 조작하는 가장 큰 이유는 원본 디스크와 사본 디스크의 크기가 다를 때 원본 디스크와 크기를 동일하게 맞춰주기 위함이다.

<p align="center">
  <img src="https://i.imgur.com/ZFFcR6g.png" alt="image"/>
<br>[ Fig 3. Before & After Drive Trim ]</p>

꼭 1TB/2TB 차이가 아니더라도 같은 1TB라도 제조사마다 사이즈가 상이할 가능성이 높기 때문에 디스크 복제 방법으로 획득할 경우 옵션은 반드시 넣어주는 게 좋다.

그럼 왜 동일하게 맞춰줄까? 말 그대로 우리는 원본 디스크의 복제본을 원하는 것이다. 만일 디스크의 크기가 다른데 그대로 복제를 했다고 가정하면 Fig 3.처럼 2TB 중 나머지 1TB는 원본 디스크의 데이터가 아니기 때문이다.

특히 나머지 1TB 영역이 와이핑 된 상태가 아닌 기존에 사용하던 디스크라고 가정하면 분석에 더욱 혼란을 줄 뿐 아니라 원본과 해시값 자체도 달라져 버리기 때문에 증거로 활용되지 못하는 아무 의미 없는 데이터이다. 그리고 1TB 영역이 와이핑이 된 상태라고 할지라도 원본과 사본이 크기가 다르기 때문에 해시값이 달라질 수밖에 없다.

이러나 저러나 원본과의 해시값이 달라지기 때문에 디스크 복제 작업을 할 때는 Drive Trim 옵션을 반드시 넣어주어 원본과 동일한 크기로 맞춰주는 작업이 필요한 것이다.

디스크 복제 작업이 아닌 E01, Ex01 등 파일 형태로 이미징하는 Drive to File 형태라면 Drive Trim 기능 자체가 없다. 이미징은 원본 디스크를 복제하여 파일 형태로 저장하는 작업이기 때문에 사본 디스크의 물리 사이즈와는 관계가 없기 때문이다.

그럼 만일 디스크 복제(Drive to Drive) 작업에서 Drive Trim 기능을 빼먹고 복제 작업한 경우 원본과 같은 영역만 해시값을 검증할 수 있는 가능성은 없을까?

## Drive to Drive(Drive Clone) with Drive Trim
테스트로 사용한 Source Drive는 4GB USB이고, Destination Drive는 1TB HDD로 사용했다. Drive Trim 옵션을 적용하여 복제가 완료된 이후 로그는 다음과 같이 출력된다.

<p align="center">
  <img src="https://i.imgur.com/JsgNnRj.png" alt="image"/>
<br>[ Fig 4. Audit Log with Drive Trim ]</p>

하이라이트 된 영역만 보고도 디스크 복제 방법 및 Drive Trim 옵션 선택 여부를 한 번에 판단할 수 있다.

- Mode: **DriveToDrive**
- Method: **MirrorClone**
- Drive Trim: **True**

현재 Drive Trim 기능이 적용됐기 때문에 사본 디스크는 물리적으로 1TB의 용량을 가졌지만 현재는 4GB로 인식될 것이다. 그럼 복제본 디스크를 분석 의뢰했다는 것을 가정하고 분석관으로서 가장 먼저 해야 될 일은 해시값 검증일 것이다. 사본 디스크를 Falcon 장비 원본쪽에 물려놓고 디스크 해시값을 계산해보자.

<p align="center">
  <img src="https://i.imgur.com/EgcxKmw.png" alt="image"/>
<br>[ Fig 5. HASH verified with Drive Trim ]</p>

정확히 일치하였다. 만일 쓰기방지장치가 없는 상태에서 복제가 정상적으로 됐는지 다른 PC에 연결해서 확인하는 작업을 했다면 이미 해시값이 틀어졌을 확률이 매우 높다.


## Drive to Drive(Drive Clone) without Drive Trim
이번에는 Drive Trim 옵션을 설정하지 않고 디스크 복제를 하였다. 팔콘 로그에 **<span style="color:red">Drive Trim: False</span>**로 명확히 나온다. 내가 배당받은 케이스와 동일한 상태이다.

<p align="center">
  <img src="https://i.imgur.com/twg0tjN.png" alt="image"/>
<br>[ Fig 6. Audit Log without Drive Trim ]</p>

이럴 경우 사본 디스크인 1TB는 Drive Trim 기능을 선택하지 않았기 때문에 그대로 1TB로 인식이 된다.
1TB 디스크 해시값은 사실 구할 필요도 없다. 4GB와 1TB는 용량조차도 다르기 때문에 디스크 해시를 계산한다면 당연히 다르기 때문이다.

Drive Trim 기능은 Default 값이 **<span style="color:red">NO</span>**로 되어있다. 따라서 해당 기능을 잘 모르거나 실수로 ***YES***로 바꾸지 못하는 등 충분히 실수를 유발할 수 있는데 이런 상황에서 증거물로 접수가 됐다면 어떻게 처리해야 될까?

원본과 해시값을 동일하게 검증할 수 있는 방법이 있을지 확인해보자.


## Drive Hash Based on the Number of LBAs
디스크 해시값을 산출하는데 기본적으로 LBA(Logical Block Address) 기준으로 계산이 된다. 이미징 장비에서 LBA 카운트 수 기준으로 해시값을 산출할 수 있는 옵션을 제공하는데 이 기능을 사용하면 특정 LBA까지만 해시값을 산출할 수 있다.

즉, 1TB의 HDD에서 4GB에 해당하는 영역만 해시값을 산출할 수 있다는 의미이다. Fig. 6 로그에서 LBA Count는 7831552이다. 이는 원본 4GB USB에 해당하는 영역이므로 아래 Fig 7.처럼 옵션을 줘서 1TB 디스크에서 해당 블록만 해시값을 산출하면 된다.

<p align="center">
  <img src="https://i.imgur.com/bl7XWah.png" alt="image"/>
<br>[ Fig 7. Confiure Target LBA Counts for HASH ]</p>

이렇게 구해진 해시값의 로그를 살펴보면 원본인 4GB USB의 해시값과 동일하다는 것을 확인할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/XqSAFTh.png" alt="image"/>
<br>[ Fig 8. Hash verified whithin LBA Counts ]</p>


## Wrap-up
지금까지 원본 디스크와 사본 디스크의 크기가 다른 상황에서 Drive Trim 없이 디스크 복제된 증거물에 대해 해시값이 동일한지 검증하는 방법에 대해 확인하였다.

그럼 그다음 단계로 분석은 어떻게 해야 될지 의문이 생길 것이다. 그리고 Drive Trim 옵션 적용 시 반드시 알고 있어야 하는 부분을 다음 part 2.로 포스팅할 예정이다.


## Reference
- <https://www.logicube.com/wp-content/uploads/2019/10/MAN-Falcon-NEO-v2.3-FIN-2.pdf>
- <https://www.logicube.com/wp-content/uploads/2017/08/MAN-Falcon-v3.2-FIN.pdf>
- <https://www.logicube.com/shop/forensic-falcon-neo/?v=38dd815e66db>
- <http://forensic-proof.com/archives/284>
- <http://egloos.zum.com/sutdaeng/v/2831491>
- <https://ata.wiki.kernel.org/index.php/Developer_Resources#ATA_command_set>


## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

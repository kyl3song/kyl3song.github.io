---
title : "Blog #5: The Basics of Car Blackbox Forensics part 4"
category :
  - DFIR
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - Blackbox
  - Dashcam
  - AVI
  - MP4
  - Video Container
  - 블랙박스
  - MPEG4
  - codec
  - H.264
  - AVC1
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
#published : true
---
차량용 블랙박스 분석 기본(Part 4 - The Final)

## Blackbox Post Series
- [Blog #2: The Basics of Car Blackbox Forensics part 1](https://kyl3song.github.io/dfir/The-Basics-of-Car-Blackbox-Forensics/)
- [Blog #3: The Basics of Car Blackbox Forensics part 2](https://kyl3song.github.io/dfir/The-Basics-of-Car-Blackbox-Forensics-part-2/)
- [Blog #4: The Basics of Car Blackbox Forensics part 3](https://kyl3song.github.io/dfir/The-Basics-of-Car-Blackbox-Forensics-part-3/)
- [Blog #5: The Basics of Car Blackbox Forensics part 4](https://kyl3song.github.io/dfir/The-Basics-of-Car-Blackbox-Forensics-part-4/)

지난 포스트에서 코덱에 대해 좀 더 살펴보고 MP4 파일에 대한 구조 및 카빙 복원 방법을 확인하였다. 이번 포스트는 블랙박스 분석의 마지막 시리즈로 실제 블랙박스 샘플 영상 파일에서 프레임의 구조 및 복원 단계를 알아보도록 할 예정이다. 샘플은 필자의 차량에서 몇 년 운행한 블랙박스 메모리 카드를 대상으로 분석할 예정이다.

## 파일시스템 분석 (Filesystem)
### FAT32 Filesystem
도구에서 블랙박스 이미징 파일을 로드하였더니 파일시스템은 FAT32를 사용하고 있는 것으로 확인된다. 모양은 FAT32처럼 생겼는데 만일 내부에 영상파일이 하나도 보이지 않는다면 TAT나 NxFS 저장 방식을 의심해봐야 한다.
<p align="center">
  <img src="https://i.imgur.com/uEh9c51.png" alt="image"/>
</p>

다행히 파일시스템이 깨지거나 그런 현상은 보이지 않는다. 파일시스템이 깨지면 파일/폴더 이름이 정상적이지 않거나 디렉터리 구조가 일부 또는 전체가 보이지 않을 수 있다.
<p align="center">
  <img src="https://i.imgur.com/U6ltsOL.png" alt="image"/>
</p>
MicroSD 내부에 뷰어 설치 파일(viewer.exe)도 있어서 설치 후 뷰어를 실행시키면 자동으로 MicroSD에 있는 영상 파일들을 로드하여 사용자에게 보여준다.

### Log File
영상, 음성 이외 제조사나 기기에 따라 로그를 다르게 남기는데 좀 더 상세하게 남기는 경우도 있고 아예 남기지 않는 경우도 있다. 만일 MicroSD에 남지 않았다면 기기 보드 자체에 어떠한 형태로라도 남아 있을 것이다. 개발사 입장에서는 어떤 문제가 생기면 문제의 원인을 파악해야 하는데 로그가 없으면 파악할 수 없기 때문이다.
<p align="center">
  <img src="https://i.imgur.com/cgJRpFw.png" alt="image"/>
</p>
샘플 블랙박스의 경우 펌웨어 업데이트 시간, 앱 구동 시간, SD 장착 여부 등과 관련된 로그를 남기고 있었다. 어떤 블랙박스들은 '주차모드' → '상시모드', '상시모드' → '주차모드'로 진입한 날짜·시간까지 로그로 기록한다.

## 영상 파일 분석 (Video File Analysis)
### 기본적인 확인 (Skimming)
해당 영상 파일의 확장자는 일반적인 avi, mp4 형태가 아니라 제조사 확장자의 형태를 가지고 있다. 해당 제조사의 확장자를 직접적으로 언급할 수가 없어 .abc로 마스킹 처리를 하였다. 파일명도 00000000.abc 형태로 되어있다. 즉 영상 파일명 자체로는 언제 촬영됐는지 확인할 방법이 없다.
영상 파일을 영상 플레이어에 넣고 재생을 하면 오류 메시지를 보여준다.
<p align="center">
  <img src="https://i.imgur.com/c4rENhD.png" alt="image"/>
</p>

다른 플레이어나 영상을 재생할 수 있는 도구를 사용하면 재생은 되나 영상 자체가 깨지거나 뭉개져서 재생되는 모습도 확인된다. 뭉개져서 재생되는 가장 큰 원인은 하나의 파일에 채널이 다른 영상 프레임들이 섞여있다는 점 때문이다.
- CH1, CH2, CH1, CH2..
- CH1, CH1, CH1, CH2, CH2, CH2..
  
아래와 같이 영상이 재생되면서 중간중간 식별할 수 있는 부분은 있지만 명확하지 않고 보고 있으면 눈이 아플 정도이다.

[전방 영상]
<p align="center">
  <img src="https://i.imgur.com/kvqeMtV.png" alt="전방"/>
</p>

[후방 영상]
<p align="center">
  <img src="https://i.imgur.com/mVHiyP3.png" alt="후방"/>
</p>

이처럼 대부분의 영상 플레이어들이 재생되지 않거나 일그러진 형태로 겨우 재생이 된다.

물론 이 샘플 파일은 정상적인 영상 파일이므로 블랙박스 전용 플레이어에서는 정상적으로 재생이 된다. 복원을 하기 위해서는 기본적으로 정상 영상의 구조를 파악 해야 한다. 구조를 파악하면 이렇게 왜곡되어 재생되는 영상을 변경하여 영상 플레이어에서도 재생할 수 있게 만들 수 있다.

### 파일 구조 분석 (부가적 영역 정보 확인)
파일의 첫 부분을 먼저 살펴보자. 파일의 시작은 '1NAJ'로 시작한다. 
<p align="center">
  <img src="https://i.imgur.com/oSdjY8w.png" alt="HEX구조"/>
</p>
스크린샷 아래쪽을 보니 '1VEJ'의 비슷한 형태가 확인된다. 분석을 어느 정도 하다 보면 대충 이런 비슷한 형태가 시그니처임을 직감적으로 알 수 있다.
1NAJ 시그니처 부터 1VEJ 시작점까지의 사이즈는 0x200(512byte)으로 바로 확인되고 그 사이즈 값은 512 byte의 맨 마지막 4자리로 판단할 수 있다.

- 0x1FC ~ 0x1FF: 0x200(512 byte), little-endian

무슨 근거로 왜 맨 마지막이 사이즈 값이라고 판단하는가? 오프셋 0x07~0x0A에도 동일한 값이 있는데 그것이 사이즈가 아니냐고 반문할 수도 있다.
이 부분은 다음 블럭에서 좀 더 설명하도록 하겠다.

아마도 이 부분은 블랙박스 기기의 정보를 포함하는 부분이라고 생각하면 될 것 같다.
여기에서 확인할 수 있는 스트링 값이 몇 개 확인된다.
- admin: 사용자 정보(ID)
- 12오 2XXX: 내 차량 번호
- Kyle Song: 블랙박스 기기에 저장된 이름  
  \- 여기서 번호판에 표현되는 한글은 EUC-KR(51949)로 인코딩을 변경해야 제대로 한글이 표현된다. Winhex의 Default 값은 ANSI ASCII 값으로 표현되기 때문에 한글이 깨져서 보인다. 한글이 보이지 않는다고 간과해서는 안 된다.

<p align="center">
  <img src="https://i.imgur.com/moA4veF.png" alt="EUC-KR"/>
</p>

즉 영상 파일이 생성될 때 기본적으로 기기의 정보가 포함된다는 것을 알 수 있다. 이런 영상 파일이 흔하지 않다. 대부분이 영상, 음성, 부가정보 등으로 구성되는데 해당 샘플은 기기에 저장된 정보까지 제공해주는 포렌식 입장에서 매우 유용할 수 있을 만한 정보를 제공해주고 있다. 영상 파일 하나만으로 차량에 대한 정보를 확인할 수 있다.

아마 블랙박스 설정 정보에 포함된 내용이 영상 파일에 일부 포함되는 것 같다.
<p align="center">
  <img src="https://i.imgur.com/nmWeX9x.png" alt="EUC-KR"/>
</p>

IVEJ 시그니처로 시작하는 다음 블럭을 확인해보자. 이번 블럭은 길이가 길어서 좀 자른 뒤의 모습이다.

<p align="center">
  <img src="https://i.imgur.com/4YLuxRz.png" alt="IVEJ"/>
</p>

IVEJ 시그니처 뒤에 또 비슷한 모양의 1BEJ 시그니처가 확인된다. 그럼 위에서 이미 한번 확인했던 것처럼 1BEJ 전까지가 하나의 덩어리일 가능성이 크다. IVEJ 덩어리의 가장 마지막 4바이트를 확인해 보면 0x400(1024 byte)으로 크기를 나타내고 있다. 즉, 위에서 어떤 근거로 맨 마지막 4바이트가 크기라고 단정했는가에 대한 답을 설명할 수 있다.
많이 보다 보면 이런 부분은 눈으로 볼 수 있게 된다.

나도 사실 그동안 분석을 하면서 현재 설명하는 부분까지는 주의 깊게 보지 못했었다. 블로그를 작성하면서 샘플을 보는 도중 파일 전체를 파악하는 중이다. 분석할 때 핵심 부분인 영상, 음성 부분을 주로 본다. 이런 세세한 부분까지 주의 깊게 보기엔 아무래도 영상/음성 영역에 비해 중요성이 떨어지기 때문이다.

다시 돌아와서 위의 IVEJ 영역에서 <span style="color:red">**빨간색**</span>, <span style="color:skyblue">**하늘색**</span>, <span style="color:lightgreen">**연두색**</span>으로 알록달록하게 표시가 된 것을 볼 수 있다. IVEJ 블록에서 가장 중요한 부분일 것이다.

이 부분은 시간 값을 의미한다. 자세히 뜯어보면 아래와 같다.

<span style="color:red">**0x7E3 0x02 0x02 0x0F 0x1F 0x34 (Little-Endian)**</span>

- <span style="color:red">0x7E3: **2019년**</span>
- <span style="color:red">0x02: **2월**</span>
- <span style="color:red">0x02: **2일**</span>
- <span style="color:red">0x0F: **15시**</span>
- <span style="color:red">0x1F: **31분**</span>
- <span style="color:red">0x34: **52초**</span>
- <span style="color:red">**2019. 2. 2. 15:31:52**</span>

<span style="color:skyblue">**0x7E3 0x02 0x02 0x0F 0x21 0x00 (Little-Endian)**</span>
- <span style="color:skyblue">0x7E3: **2019년**</span>
- <span style="color:skyblue">0x02: **2월**</span>
- <span style="color:skyblue">0x02: **2일**</span>
- <span style="color:skyblue">0x0F: **15시**</span>
- <span style="color:skyblue">0x21: **33분**</span>
- <span style="color:skyblue">0x00: **0초**</span>
- <span style="color:skyblue">**2019. 2. 2. 15:33:00**</span>

<span style="color:lightgreen">**0x7E3 0x02 0x02 0x0F 0x22 0x08 (Little-Endian)**</span>
- <span style="color:lightgreen">0x7E3: **2019년**</span>
- <span style="color:lightgreen">0x02: **2월**</span>
- <span style="color:lightgreen">0x02: **2일**</span>
- <span style="color:lightgreen">0x0F: **15시**</span>
- <span style="color:lightgreen">0x22: **34분**</span>
- <span style="color:lightgreen">0x08: **8초**</span>
- <span style="color:lightgreen">**2019. 2. 2. 15:34:08**</span>

이 시간 값이 어디에서 사용되고 있는지 전용뷰어를 통해 확인해보자.
<p align="center">
  <img src="https://i.imgur.com/pCiOcPz.png" alt="뷰어"/>
</p>

블랙박스 전용 뷰어에서 영상 파일 1개를 로드하면 총 3개의 시간 목록이 나오는데 이는 영상의 시작 시간을 의미한다.  
뷰어에서 재생하면 그 시간부터 재생이 되는 것을 확인할 수 있다. 그리고 약 1분가량 재생이 된다. 즉 전체 3개의 목록이 있으니 총 3분의 영상이 1개의 파일에 저장되어 있다는 것을 알 수 있다.

사실 블랙박스를 분석 초심자에게는 가장 어려운 부분이 시간 값을 판별하는 부분이다. 
시간 값은 영상마다 정말 다양하게 저장되어 있다. 가장 쉽게는 스트링 형태로 사람의 눈으로 바로 읽을 수 있게 저장된 것부터 비트를 쪼개서 계산해야만 정밀한 시간값을 확인할 수 있게 저장된 형태도 있다. 물론 시간값이 아예 Hex값으로 저장이 안되어 있는 영상도 많다.  
시간 값 판별은 나도 지금까지 수많은 삽질(?)을 하면서 가장 어려웠던 부분이기도 하고 아직도 어렵다. 

영상에서 시간 값이 저장된 패턴을 확인했다면 그 뒤부터는 거의 동일 패턴으로 시간 값을 저장한다. 개발자 입장에서 생각해보면 납득이 갈 것이다.

### 영상 프레임 구조 분석
영상 프레임의 구조를 확인해보자.
<p align="center">
  <img src="https://i.imgur.com/FA9GmTm.png" alt="전방영상sig"/>
</p>

- 01VI: **영상 프레임 시그니처**
- 0x620D: **프레임 사이즈**
- 0xE07: **2019년**
- 0x02: **2월**
- 0x02: **2일**
- 0x0F: **15시**
- 0x1F: **31분**
- 0x34: **52초**

위에서 한 번 정리를 해서 아마 처음보다 보는데 조금 더 익숙해졌을 것이다. 샘플 블랙박스는 개별 프레임에 대한 시간 값이 들어있다. 시간 값이 없는 블랙박스가 대부분인데 이 경우에는 운이 좋은 케이스이다.

사이즈는 일반적으로 H.264 NAL Unit(SPS)부터 시작되므로 시작점부터 크기만큼을 잘라서 사진으로 변환할 수 있다. 잘 모르겠다면 프레임 시그니처부터 사이즈만큼 추출해도 크게 문제는 되지 않는다.

<p align="center">
  <img src="https://i.imgur.com/ustYzbr.png" alt=""/>
</p>

``` shell
> ffmpeg -i frame frame.jpg
```
변환된 사진 파일을 보면 후방 영상의 영상 프레임으로 확인된다.
<p align="center">
  <img src="https://i.imgur.com/x0dE7nQ.png" alt=""/>
</p>

그럼 전방 영상 프레임도 있을 것인데 과연 어디에 있을까? 영상 파일을 다시 확인해보자.
<p align="center">
  <img src="https://i.imgur.com/IXHIuaw.png" alt=""/>
</p>

후방 영상 프레임과 동일하게 각 값을 해석할 수 있고 사이즈만큼 추출하여 사진 파일로 변환하면 전방 영상을 확인할 수 있다.
<p align="center">
  <img src="https://i.imgur.com/MfhUyVb.png" alt=""/>
</p>

- 00VI: **영상 프레임 시그니처(전방)**
- 0x5531: **프레임 사이즈**

여기서 전반적인 영상 프레임의 구조가 파악된다. 00VI, 01VI 시그니처는 전방 후방 영상 프레임의 구분자로 사용할 수 있다.
즉 전방 영상을 모으고 싶으면 00VI로만 검색해서 영상 프레임을 합칠 수 있고, 시간 값을 파싱하여 언제 촬영된 영상인지도 확인할 수 있다는 결론이 나온다.

사실 좀 더 깊게 들어가면 재밌고 흥미로운 내용이 많이 있으나 블로그에서 언급하기에 너무 양이 많아 구조 분석은 여기서 마무리 짓는다.

블랙박스 분석을 하다 보면 파일 시스템이 깨져있는 경우도 존재한다. 예전 어떤 케이스 중 파일시스템 데이터가 섹터 단위로 밀려 써지고 다른 데이터로 덮어쓰인 경우도 있었다. 이럴 때는 얼마나 밀려있는지, 당겨져 있는지를 파악하고 만일 회복하기 어려운 상태라면 FAT32 Root Directory(Cluster 2)까지 도구에서 읽을 수 있게 위치를 맞춰주는 작업을 한다면 Data 영역의 영상이 일부 복원 가능하도록 시도할 수 있다. 
즉 케이스에 따라 다를 수 있지만 일반적으로 파일시스템 레벨에서의 복원 방법을 최대한 사용하는 것을 추천한다.

위에서 언급한 것과 같이 포스트에서 소개한 샘플 블랙박스 영상은 운이 좋은 경우이다. 원래 시간 값이 없는 영상이 대부분이거나 따로 인덱스 형태로 관리하는 경우도 많다. 이렇게 시간 값이 없는 경우 따로 관리하는 영역을 찾아서 복원에 사용하든지 아예 시간 값이 존재하지 않는 경우는 또 다른 방법으로 복원을 해야한다.


## 음성 복원의 중요성 (Importance of Audio Recovery)
블랙박스는 DVR과는 다르게 음성이 저장된다. 블랙박스 기기에서 음성 설정을 임의로 끄지 않는 이상 음성은 디폴트로 ON 상태로 되어있다.

영상 복원만큼 음성 복원이 중요할 때가 있다. 특히 차량 내에서 성범죄가 발생했을 경우 음성 복원은 필수적이다. 택시에서 범죄가 발생하는 경우도 있고 일반 개인 차량에서 발생하는 경우도 있다. 특히 일반 차량과 다르게 택시 블랙박스의 경우 전방, 차량 내부, 후방 3개의 채널을 촬영하는 경우가 있고 전방 또는 내부 채널 2개를 촬영하는 경우가 있다.
따라서 사건에 따라 영상 채널, 음성 복원의 중요성이 결정된다.

실제로 택시 내부에서 성범죄가 발생되어 분석이 의뢰된 케이스가 있었다. 정상적인 파일의 오디오 코덱 등 정보를 확인해서 동일한 오디오 코덱과 스펙을 맞춰 음성 복원을 하는데 정보를 동일하게 해줘도 잡음이 심한 경우였다.
일단은 영상 부분을 샅샅이 뒤져봤으나 덮어써진건지 녹화되지 않은 건지 복원이 되지 않는 케이스였고 음성 쪽에 사활을 걸었었는데 복원된 잡음 심한 음성을 계속 듣고 있으니 나중에는 마치 여성의 비명소리처럼 들리는 환청을 경험하였다. 그 뒤로 며칠간 삽질을 해가며 깨끗하게 복원된 음질을 원했지만, 해당 케이스는 그렇게 마무리를 지을 수밖에 없었던 아쉬운 사건이었다.


## Blackbox vs. DVR
블랙박스와 DVR의 분석은 비슷하면서도 차이가 있다. 먼저 두 개를 놓고 비교하자면 DVR이 훨씬 복잡하고 난이도도 높다. 난이도가 높은 몇 가지 이유를 아래와 같이 정리할 수 있다.
- **Channel:**  
  블랙박스는 채널이 전방 또는 후방 2개로 정말 많아 봐야 4~5개 정도로 비교적 명확하다. DVR은 채널을 가늠할 수 없다. DVR의 채널은 최소 4개에서 호텔/클럽에서 사용하는 DVR 채널은 60개 이상인 것도 본 경험이 있다.
- **Timeframe:**  
  블랙박스 영상에 저장된 시간 값은 대략 UnixTime, hfstime, 일반 스트링 형태, 위 샘플 파일과 같이 '연월일시분초'가 헥사값으로 분리되어 저장되는 등 대략 몇 가지로 한정되어 있다. DVR은 시간 값 저장 형태가 본 포스트에서 소개한것 외로 더 복잡하게 저장되어 있는 경우도 많다. 심지어 시간 값을 구하려면 비트연산을 해야 하는 경우도 있다.
- **Size:**  
  DVR은 사이즈에 해당하는 값이 기상천외한 곳에 붙어 있는 경우도 있고 아예 없는 경우도 있다.
- **Resolution:**  
  단일 해상도를 사용하는 블랙박스와는 달리 DVR의 경우 해상도별 영상이 저장된 경우가 있다. 해상도별 영상이 저장되어 있는 경우는 일반적으로 큰 해상도, 작은 해상도로 나뉘어 지고 더 세부적으로 나뉘는 DVR도 있다. 영상을 해상도별 저장하는 가장 큰 이유는 네트워크를 통해 모바일 기기에서 영상을 볼 때 사용하는데 작은 화면에서 굳이 1920 x 1080 해상도 영상을 보여줄 필요가 없기 때문이다.
- **저장 방식:**  
  일반적으로 데이터가 저장될 때 Little-Endian 또는 Big-Endian으로 데이터가 저장된다. 보통 한가지 형태로 저장이 되는데 DVR의 경우 어떤 것은 Big으로 읽어야 할 때가 있고 어떤 부분은 Little로 읽어야 해석이 되는 경우가 있다.


## 영상 분석 시 어려운 점 (Difficulties in Video Forensics)
디지털증거를 분석할 때 어려운 점은 심리적으로 충격을 받을 수 있다는 점이다. 특히 사건이 무거울수록 심리적인 부담이 커지기 마련인데 요즘에 뉴스에 자주 나오는 어린이집에서 아동 학대 사건 등과 같은 케이스의 경우에 일단 한번 마음을 먹고 접해야 한다. 분석을 하다 보면 무고한 아이들이 맞는 장면을 볼 수도 있기 때문이다.

예전에 전동차 사고로 인한 사망사건 열차 블랙박스를 분석한 적이 있는데 사실 분석 하기도 전에 고민이 됐다. 자칫하면 내가 사망 당시의 끔찍한 영상을 볼 수도 있기 때문이다.
이렇게 모든 업무에서 이면이 존재하듯 디지털포렌식이라는 업무에는 이러한 어려운 점이 존재하는 것 같다.

블랙박스 분석 시리즈를 작성하면서 많은 부분을 지나친 것 같다. 욕심 같아서는 하나하나 다 언급하고 짚고 넘어가고 싶지만 그럴 수 없어서 뭔가 아쉬운 것 같다. 혹시라도 앞으로 시간이 되면 더욱 알차게 영상 분석 관련된 내용으로 포스팅 하도록 하겠다.

## Copyright (CC BY-NC 2.0 KR)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

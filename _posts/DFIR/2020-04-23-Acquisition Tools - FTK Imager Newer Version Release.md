---
title : "Blog #6: Acquisition Tools - FTK Imager Newer Version Release"
category :
  - Acquistion
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - FKT Imager
  - FTK Imager 4.3
  - Forensic Falcon
  - EnCase
  - Matplotlib
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
획득 도구 - FTK Imager 4.3. 릴리즈

## FTK Imager Release
두 달전쯤 [Cut Your Imaging Time In Half With FTK® Imager 4.3](https://www.forensicfocus.com/News/article/sid=3852/)라는 제목의 뉴스를 보게 되었다. FTK Imager의 새로운 버전(4.3)이 릴리즈 되었다는 기사였다.

<p align="center">
  <img src="https://i.imgur.com/qTikPiy.png" alt="image"/>
</p>

그중에서 가장 눈을 사로잡은 건 바로 이미징 속도였는데 기존 4.2 버전보다 거의 절반 또는 절반 이상이 빠른 부분이었다.

**FTK Imager version 4.2.x and earlier:**  
>Compressed images: 2:21:00  
 Non-compressed images: 4:16:23

**FTK Imager version 4.3.x and newer:**  
>Compressed images: 1:03:41  
 Non-compressed images: 1:29:25  

이미징 속도가 대폭 향상된 부분과 MAC OS의 APFS 파일 시스템을 해석할 수 있다는 점에 한 번 써보고 싶다는 생각이 들었다.

오늘의 포스팅 주제로 획득 도구에 관한 이야기를 해보려고 한다. 실제로 새롭게 릴리즈된 FTK Imager 버전과 비교도 하면서 말이다.

## 이미지 획득 도구 (Acquisition Tools)
복제이미지 획득 도구는 압수 현장에서 행해지는 디지털증거분석 일련의 과정 중 첫 단계를 의미한다.

획득 도구는 크게 두 가지 타입이 있는데 [Logicube社 Forensic Falcon](https://www.logicube.com/shop/falcon/?v=38dd815e66db)과 같은 하드웨어 타입의 데이터 획득 도구와 [ACCESSDATA社 FTK Imager](https://accessdata.com/product-download)와 같은 소프트웨어 타입의 획득 도구로 나눌 수 있다.

하드웨어 타입의 도구로 데이터 획득 시 시간을 절약할 수 있으면서 안정적으로 데이터에 대한 획득을 진행 할 수 있다. 만일 하드웨어 타입의 전용 획득 도구 부재 시 소프트웨어 타입의 획득 도구를 사용하여 데이터 획득을 진행할 수도 있다. EnCase와 같은 대부분의 상용 분석 도구에서도 획득 기능을 지원한다.

**Falcon 획득 설정**
<p align="center">
  <img src="https://i.imgur.com/Yco6ZYi.png" alt="image"/>
</p>

**FTK Imager 획득 설정**
<p align="center">
  <img src="https://i.imgur.com/9N4RUEz.png" alt="image"/>
</p>

**EnCase 획득 설정**
<p align="center">
  <img src="https://i.imgur.com/kiKcOrm.png" alt="image"/>
</p>


## 에러 처리 기능 (Error Handling)
기본적으로 하드웨어, 소프트웨어 타입 획득 도구에는 획득을 진행하면서 오류가 발생할 수 있는 부분에 대해 처리할 수 있는 기능이 있다.
특히 원본 디스크에 배드섹터가 있는 경우 이를 스킵할 수 있는 기능이 있는데, Falcon의 경우 소스 드라이브(원본 디스크)에서 배드섹터가 발견될 경우 이미징 작업을 종료할 것인지, 스킵할 것인지를 작업을 시작하기 전에 설정하는 옵션이 있다.

만일 이미징 작업 중 배드섹터를 만나게 되면 섹터를 읽지 못하는 경우 최대 7번의 Retry를 시도하고 그래도 읽지 못하는 경우는 해당 섹터를 '0x00'으로 채운다.

[Forensic Falcon 매뉴얼](https://www.logicube.com/wp-content/uploads/2017/08/MAN-Falcon-v3.2-FIN.pdf) 참고
> When bad sectors are encountered, and error handling is set to
Skip, Falcon will write a zero on the corresponding sector or
position in the Destination drive or file.

에러 처리를 할 때 몇 섹터를 점프할 것인지에 대한 설정도 있는데 이 부분은 기기의 펌웨어에 따라 다른데 기본은 1섹터로 되어 있다. 사용자가 에러처리 부분을 선택할 수 있도록 옵션화되어 있는 도구도 있고 그렇지 않은 도구도 있다.

위에서 언급한 세 가지의 도구에서 에러 발생시 스킵할 수 있는 범위는 표와 같다.



| No. | Type | Product Name | Error Handling (Sector) |
|:---:|:------------------------:|:------------------------:|:------------------------:|
| 1 | Hardware Type | Forensic Falcon | 1,8, 128 |
| 2 | Software Type | EnCase Forensic | 64 ~ 1024 (power of 2) |
| 3 | Software Type | FTK Imager      | Unknown (Not Optional) |


## 부가 기능 (Additional Features)
하드웨어 획득 도구는 전용 장비로 이루어진 만큼 소프트웨어 획득 도구에 비해 조금 더 많은 기능이 존재한다. 예를 들면 장비를 쓰기방지장치를 대신하여 사용하면 압수 현장에서 유용하게 사용할 수 있다. 그 외의 다양한 기능이 존재한다.

- 네트워크 공유폴더 획득 기능
- 병렬 이미징 가능
- 동시에 매체를 이미징 및 복제(하드카피) 가능
- 소거(Erase) 기능

하드웨어 획득 도구의 매뉴얼을 참고하면 더 자세한 내용을 확인할 수 있다.
서두는 여기서 마치기로 하고 본격적으로 FTK Imager 새로운 버전에 대해 테스트를 진행하고 비교를 해보겠다.


## FTK Imager Installation
### Download URL
AcessData FTK Imager는 [Download Link](https://accessdata.com/product-download/ftk-imager-version-4-3-0)에서 다운로드 할 수 있다.
다만 다운로드를 하려면 이메일 등 몇 가지 정보를 적어야만 다운로드가 가능하다.

<p align="center">
  <img src="https://i.imgur.com/wrzxIc5.png" alt="image"/>
</p>

### FTK Imager Version Info
설치가 완료된 후 버전은 4.3.0.18이고 만일 그 전 버전(4.2.X)이 설치가 되어 있는 상태에서 추가로 설치되지 않는다. 버전이 높든 낮든 기존 버전을 지워야만 설치가 가능하다.
<p align="center">
  <img src="https://i.imgur.com/KvEfMzI.png" alt="image"/>
</p>


## 테스트 진행 개요 (Test Scenario Overview)
범용적으로 사용하는 소프트웨어 툴(EnCase, FTK Imager)을 이용하여 획득 시 안정성 및 시간 소요의 2가지에 초점을 맞췄다.  
테스트는 동일한 컴퓨터의 동일한 환경에서 1TB HDD를 대상으로 획득을 진행함으로써 테스트의 신뢰성을 확보하였다. 획득 이후 검증 시 HASH는 MD5, SHA1 모두 생성하였다.  
또한 하드웨어 도구(Forensic Falcon)는 동일한 환경은 되지 못하나 결과를 같이 비교하고자 동일한 매체를 대상으로 획득을 진행하였다.

- 테스트 환경: Intel(R) Xeon(R) Gold 6126 CPU 2.60GHz 2.59GHz / RAM 128GB
- 대상 매체: 1TB HDD (파일로 거의 가득찬 하드디스크) * 1EA
- 이미지 파일 분할 옵션: 4GB
- 검증할 해시값: MD5, SHA1
- 쓰기방지장치를 연결할 상태에서 이미징 수행

### 실험할 도구 버전 (Product Version)
FTK Imager 4.2.0.13을 이용하여 데이터 획득을 진행하고, 개선되어 새롭게 출시된 FTK Imager 4.3.0.18을 이용하여 추가로 데이터 획득을 진행한 뒤 서로를 비교하고자 한다.  
그리고 EnCase 도구를 이용하여 획득한 결과와 Forensic Falcon을 이용한 획득 결과도 비교해 보고자 한다.

|No.|Type|Product Name|Version|
|:---:|:------------------------:|:------------------------:|:------------------------:|
|1|Hardware Type|Forensic Falcon|3.2u1|
|2|Software Type|EnCase Forensic|8.08|
|3|Software Type|FTK Imager|4.2.0.13|
|4|Software Type|FTK Imager|4.3.0.18|

### 이미징 작업 수행
각 도구별 이미징 작업을 수행이 완료된 후 해시값 비교 방법으로 획득 파일에 대해 검증을 하였고, 확인 결과 획득된 모두 도구에서 해시값이 일치하였다.

[Forensic Falcon]
<p align="center">
  <img src="https://i.imgur.com/gCIJKEg.png" alt="image"/>
</p>

[FTK Imager 4.2.0.13]
<p align="center">
  <img src="https://i.imgur.com/kIaYx4i.png" alt="image"/>
</p>

[FTK Imager 4.3.0.18]
<p align="center">
  <img src="https://i.imgur.com/Uu7tzo9.png" alt="image"/>
</p>

[EnCase 8.08]
<p align="center">
  <img src="https://i.imgur.com/fbtIwe8.png" alt="image"/>
</p>

### 획득 시간 결과 (Acquisition Time Elapsed)

- **Forensic Falcon**  
  \- Total: 3:50:29
- **EnCase Forensic 8.08**  
  \- Imaging: 2:52:26  
  \- Image Verification: 2:38:42  
  \- Total: 5:31:08
- **FTK Imager 4.2.0.13**  
  \- Imaging: 2:20:31  
  \- Image Verification: 2:07:29  
  \- Total: 4:28:00
- **FTK Imager 4.3.0.18**  
  \- Imaging: 2:06:53  
  \- Image Verification: 1:53:07  
  \- Total: 3:59:00

획득이 완료된 시간을 기준으로 빠른 순서대로 나열하면 다음과 같다.
``` shell
Forensic Falcon > FTK Imager 4.3.0.18 > FTK Imager 4.2.0.13 > EnCase 8.08
```

결과를 도식화하여 표현하면 결과 비교를 좀 더 쉽게 할 수 있다. 도식화는 Python Matplotlib 라이브러리를 이용하여 시각화한 것이고 기본 코드는 [블로그](https://m.blog.naver.com/PostView.nhn?blogId=cjh226&logNo=221266254398&proxyReferer=https:%2F%2Fwww.google.com%2F)를 참고하여 나름에 맞게 수정하였다.  
참고로 Falcon의 경우 획득 시간, 검증 시간을 따로 보여주지 않고 전체 획득이 완료된 시간만 획득 결과를 보여주기 때문에 전체 시간을 반으로 나눠 획득 시간, 검증 시간으로 그래프를 설정하였다.
<p align="center">
  <img src="https://i.imgur.com/RsH2rbD.png" alt="image"/>
</p>

관련 코드는 [github](https://github.com/kyl3song/Python-Code-Snippet/tree/master/Github_Blog/Blog6)에 업로드 했고 필요한 경우 언제든 수정하여 사용할 수 있다.

그런데 결과를 보면 생각보다 FTK Imager간 많은 성능 차이를 보이지 못하고 전체 시간 기준 약 30분 차이의 성능을 보였다. 실험 환경이 달라서 결과가 이와 같이 산출됐을 가능성이 있다. 정리하자면 릴리즈된 FTK 4.3 버전은 이전 버전보다 획득 성능이 향상된 것은 명확하다.

AccessData에서 실험을 어떤 방식으로 했는지는 알 수 없지만 필자가 실험한 방식은 증거분석실에서 일반적으로 사용하는 방식인 하드웨어타입 쓰기방지장치에 디스크를 연결한 뒤 획득을 진행한 방식이라 결과는 충분히 다를 수 있다.


## Reference
- <https://www.logicube.com/shop/falcon/?v=38dd815e66db>
- <https://accessdata.com/product-download>
- <https://www.logicube.com/wp-content/uploads/2017/08/MAN-Falcon-v3.2-FIN.pdf>
- <https://m.blog.naver.com/PostView.nhn?blogId=cjh226&logNo=221266254398&proxyReferer=https:%2F%2Fwww.google.com%2F>
- <https://matplotlib.org/>


## Copyright (CC BY-NC 2.0 KR)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
---
title : "Blog #30: Why we need Deep Learning in Video Forensics [KR]"
category :
  - Video Forensics
tag : 
  - Machine Learning
  - Deep Learning
  - Video Forensics

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
영상 포렌식에서의 딥러닝이 필요한 이유


## This Post Covers
수많은 영상 영상 프레임 데이터가 하나의 영상 파일을 만든다. 따라서 영상 파일을 복원한다는 의미는 과정은 이 프레임을 모두 합쳐 흩어진 조각을 하나로 모은다는 의미이기도 하다.
만일 영상 프레임 데이터(바이너리) 내 시간 값이 저장되어 있지 않은 경우 이들을 합치고 정렬할 수 있는 기준이 없어 영상으로 만들기 어렵다. 이럴 경우는 어떻게 복원을 할 수 있을까? 나도 많은 고민을 했던 내용이다. 오늘은 2019년에 겪었던 이야기를 꺼내보고 앞으로의 방향에 대해 생각해 보고자 한다.

## Issues of Frame-based Recovery
우리는 DVR보다 차량용 블랙박스를 쉽게 접할 수 있다. 블랙박스에 삽입된 Micro SD를 빼서 컴퓨터에서 영상을 재생시켜봤다면 굳이 뷰어를 사용하지 않고서도 영상을 더블 클릭만으로도 손쉽게 볼 수 있었을 것이다. 이는 우리에게 익숙한 AVI, MP4 등 영상 컨테이너 형태로 만들어져 있기 때문에 일반적인 영상 플레이어도 재생이 가능한 것이다.

DVR의 경우는 좀 다르다. DVR는 일반적으로 회사, 영업점, 아파트 단지 등 대부분 공공의 이익을 위해 모니터링을 하는 경우가 많다. 따라서 이를 관리하는 관리자가 따로 있기 마련이다. DVR은 일반적으로 내부에 HDD를 사용하고 HDD를 탈착하여 컴퓨터에서 보면 쉽게 보이지 않는다. 파일 시스템이 아예 없는 경우도 있고, 있더라도 제조사마다 영상의 자체 포맷을 사용하기 때문에 그들이 제공하는 전용 뷰어를 설치하지 않는 이상 일반적인 영상 플레이어를 통해 재생할 수 없는게 거의 대부분이다.

이런 진부한 얘기를 하는 이유는 블랙박스는 일반 사람들이 많이 사용하기 때문에 영상이 촬영된 날짜, 시간이 영상에 삽입되어 있다는 사실에 주목할 필요가 있다. DVR는 전용 플레이어를 통해 영상 파일(자체 포맷)에 저장된 시간 값을 읽어 화면에 현출해 주는 방식이 더 많고 블랙박스는 그 반대로 대부분 촬영 일시가 영상에 아예 삽입되어 있는 경우가 더 많다.


<p align="center">
  <img src="https://i.imgur.com/hjimf4Q.png" alt="image"/>
</p>


물론 이 내용은 경우의 수가 더 많다는 의미이다. 그럼 블랙박스 영상 프레임에서 시간 값이 없는 경우는 프레임만이라도 복원할 수밖에 없다. 영상 프레임 복원을 경험해 본 사람을 알겠지만 프레임 수는 생각보다 매우 많이 나온다. 이를 전달받는 사람은 수십만 장의 사진으로 회신 받을 수도 있다.


## Try OCR
2019년 내가 영상 포렌식반으로 이동하여 영상 분석을 시작한 지 얼마 안 됐을 때 증거 분석을 의뢰한 사람이 수십만 장을 직접 눈으로 보기에 좀 가혹한 일이 아닐까라는 생각을 말이다. 그러다가 갑자기 OCR이 떠올랐고 Tesseract를 설치하여 바로 시도해 봤다.

<p align="center">
  <img src="https://i.imgur.com/b9RB6Hm.png" alt="image"/>
</p>

나름 성능이 괜찮았다. 코딩을 통해 다음과 같은 작업을 자동화했다.
- 전체 프레임(사진) 파일 OCR 돌리기
- OCR 결과 중 촬영 일시만 추출
- 시간값 및 파일명 저장 (정렬은 Optional)

수많은 복원된 프레임 중 원하는 시간대의 영상 프레임을 찾아갈 수 있는 인덱스 역할을 제공한 셈이다.

<p align="center">
  <img src="https://i.imgur.com/0FSfyqh.png" alt="image"/>
</p>

그런데 여기서 문제가 발생한다. Tesseract는 아주 잘 만들어진 프레임워크지만 시간 값을 정확히 인식을 못하는 경우도 있었다. 특히 블랙박스 제조사마다 다른 폰트를 사용하고 있기 때문에 OCR로 잘 읽혀지는 폰트가 있는가 하면 전혀 엉뚱한 시간으로 읽어들이는 폰트도 존재했다.

특히 0, 6, 8을 구분을 잘 못 지었고 구분자인 '/'를 숫자 1로 인식한다든지 하는 경우도 많았다. 즉, 해당 케이스에는 성공적으로 인덱스를 제공하여 사건을 마무리했지만 다른 경우에는 사용할 수 없었다.


## Train Tesseract
Tesseract 4.0부터 딥러닝 모델인 LSTM이 들어갔다. 즉 문맥을 이해할 수 있고 이를 기반으로 OCR을 할 수 있다는 의미이다. 그러나 내가 필요했던 건 숫자 + 구분자('/', ':', '-' 등)였기 때문에 문맥이 그리 필요 없었다.
tesseract에는 인식률을 높이기 위한 다양한 옵션들이 존재한다. 블랙박스는 텍스트 영역이 대부분 한 줄로 표현되어 --psm 6 또는 7 옵션을 주로 사용하고 OCR 엔진은 Legacy로 사용이 제일 무난했었고, 다른 옵션들도 섞어봤지만 별다른 개선 사항은 없었다.

<p align="center">
  <img src="https://i.imgur.com/yOJ8aUw.png" alt="image"/>
</p>

찾아보니 Tesseract를 학습 할 수 있다는 내용이 보이기 시작했고 [jTessBoxEditor](https://softfamous.com/jtessboxeditor/download/) 등 도구들을 잔뜩 뒤져가며, 시도했으나 모두 만족할 만한 결과를 내지 못하였다. 나의 OCR을 이용한 첫 번째 케이스는 인덱스 파일을 주는 형태로 마무리되었다. 


## Meeting OpenCV
우연히 인터넷에서 [딥러닝과 OpenCV를 활용해 사진 속 글자 검출하기](https://d2.naver.com/helloworld/8344782)라는 글을 보게 됐는데 내가 원하는 것과 거의 일치했다. 당시 영상 처리 기술은 처음 접하는 내용이었지만 해당 글에서 이미지 전처리 절차의 중요성을 배우게 됐고, OpenCV를 꾸준히 학습해 나갔다.
그리고 2019. 11. 조금 더 개선된 도구를 만들게 됐다.

그 무렵에 OCR을 이용한 두 번째 케이스를 분석하게 됐다. 당시 복원이 필요한 영상 범위는 2019. 10. 10.이었고 미할당 영역을 대상으로 남아있는 영상 프레임을 도식화하면 다음과 같았다.

<p align="center">
  <img src="https://i.imgur.com/ANcKMFE.png" alt="image"/>
</p>

도구는 다음과 같은 절차로 로직을 구현하였다.

### 1. 미할당 영역을 대상으로 영상 프레임 복원  
시간에 구애받지 말고 프레임을 그냥 합쳐 영상 통으로 만든다. 시간 값이 뒤죽박죽이라 이 상태에서는 영상을 정상적으로 보기는 힘들다. 봐도 1일꺼 나왔다가 5일꺼 나왔다가 뒤죽박죽으로 나와 영상으로의 의미가 없다.

<p align="center">
  <img src="https://i.imgur.com/rF0kAZ8.png" alt="image"/>
</p>
   
### 2. 영상 파일 로드 및 대기
이 단계에서는 영상 파일의 1개의 프레임을 읽어 사용자 모니터에 현출시킨 후 사용자의 행위를 대기한다.

### 3. ROI 지정 및 CROP
ROI는 Region of Interest로 내가 원하는 처리할 관심 영역이라는 의미이다. 위 ROI를 설정해 주는 이유는 프레임 전체에서 문자열을 읽는 것보다 관심 영역을 상대로만 문자열을 읽는 것이 정확도가 높고, 이미지 전처리 과정에 있어서도 수월하기 때문이다. 내가 원하는 부분만 좌표로 지정되면 해당 영역만 이미지 프로세싱을 할 수 있기 때문이다.

<p align="center">
  <img src="https://i.imgur.com/eUyUMHN.png" alt="image"/>
</p>

ROI는 마우스 드래그로 하면 되고 연두색 색상으로 실시간 표시가 된다. 드래그 했을 때의 좌표값을 기준으로 이미지를 잘라내어(Crop) 분석관에게 보여준다.

### 4. Image Preprocessing
복원된 영상에서 프레임을 한 개씩 읽어들여 프레임 전처리 작업을 수행한다. 전처리 작업은 OCR의 정확도를 높일 수 있기에 필요하다.
- Grayscaling: 컬러 이미지를 흑백 이미지로 변환
- Blurring: 흑백 이미지를 블러링하여 뿌옇게 만듦 (이미지의 작은 노이즈들 제거)
- Thresholding: OCR로 읽을 숫자를 선명하게 하기 위해 숫자가 가지고 있는 색상을 제외한 나머지 색상을 대비시키는 작업

<p align="center">
  <img src="https://i.imgur.com/tXCJIM3.png" alt="image"/>
</p>

전처리를 하는 이유는 단순하다. OCR을 통해 잘 읽을 수 있게 준비작업을 해주는 과정이다. 어차피 동일한 OCR을 사용한다면 전처리를 수행하여 더 잘 차려진 밥상을 먹게 해주는 개념이라고 보면된다.

### 5. OCR 수행
영상 복원 요구 기간에 해당하는 영상 프레임이면 영상으로 저장하고, 기간에 해당하지 않는 영상 프레임은 스킵하는 과정이다. 또한 OCR 수행 중 현재 어느 시간을 읽고 있는지 분석관에게 현출해 주는 로직을 넣었다.

<p align="center">
  <img src="https://i.imgur.com/isXknQ6.png" alt="image"/>
</p>


복원은 나름 성공적이었고 생각보다 정교했다. 딱 2019. 10. 10. 영상만 추출하여 복원이 되었다. 그렇게 두 번째 OCR 케이스를 마쳤다.

<p align="center">
  <img src="https://i.imgur.com/2yvbA3l.png" alt="image"/>
</p>

이게 끝은 아니었다. 여기에도 한계는 존재했었다. ROI를 아무리 지정해도 자동차가 주행하면서 영상의 배경이 바뀌는데, 이를 전처리 하기가 생각보다 쉽지 않았고 그에 따른 OCR이 중간 중간 이상하게 반응을 하는 게 문제였다.


## Need Deep Learning
딥러닝이 나올 차례가 됐다. 딥러닝은 밑바닥부터 개념이 잡혀있어야 하기 때문에 Neural Network부터 알아야 하고 딥러닝에 필요한 수학 학습이 반드시 필요하다. 그리고 원래 내 목표는 딥러닝을 통해 영상 복원을 하는게 목표였다. 그리고 더 나아가 내 논문 주제였다.

하지만 내가 영상 분석반에서 다른 반으로 옮기는 바람에 영상 데이터를 샘플을 모으는 것이 어려워졌고 현재 업무와 연관이 돼야 더 시너지가 날 수 있는데 그렇지 못해 포기할 수 밖에 없었고 아쉬운 마음이 있었다.

현재 학회에서 알아주는 OCR 기술은 네이버 CLOVA OCR이다. 그만큼 컴퓨터 비전쪽인 CVPR, ICCV 학회에 논문도 많이 내고 세계 최고의 수준의 정확도라고 해도 과언이 아니다.

지금은 [CLOVA OCR API](https://guide.ncloud-docs.com/docs/ko/ocr-ocr-1-1)를 제공하고 있다. 물론 무료가 몇건 있고 나머지는 유료이니 테스트로 한번 사용해보는 것도 좋을 것 같다.

<p align="center">
  <img src="https://i.imgur.com/dixRz9U.png" alt="image"/>
</p>


## Wrap-up
기존의 영상 분석은 대부분 Low-level인 바이너리 형태로 모든 것을 분석하고 복원하는 과정을 거친다. 매우 중요한 일이고 영상 복원에 기초가 되며 대부분의 작업이 그렇게 이뤄진다.

그러나 난 좀 다르게 접근을 해보고 싶었다. 그동안 모두 바이너리쪽만 바라보고 있는데 상위 레벨인 영상 프레임, 사진 자체에서 접근이 필요하다고 생각했고 지금도 마찬가지로 그쪽 연구가 더 필요하다고 생각한다. Plan A가 망가졌을때 B가 있으면 이를 보완해줄 수 있기 때문이다.

머신러닝/딥러닝은 영상 분야뿐 아니라 생각만 잘 해보면 블랙박스에 저장된 음성도 딥러닝을 적용실 수 있다. 그리고 다른 디지털포렌식 분야에도 다양하게 적용시킬 수 있어 한 번 고민해 보면 좋을 것 같다. 


## Reference
- <https://softfamous.com/jtessboxeditor/download/>
- <https://d2.naver.com/helloworld/8344782>
- <https://guide.ncloud-docs.com/docs/ko/ocr-ocr-1-1>


## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

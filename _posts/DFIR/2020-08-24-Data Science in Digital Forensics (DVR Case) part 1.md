---
title : "Blog #14: Data Science in Digital Forensics (DVR Case) part 1"
category :
  - Data Science
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - CCTV
  - DVR
  - Pandas
  - Numpy
  - Data Science
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
디지털포렌식에서의 데이터 과학 (DVR Case) part 1


## Preface
[데이터 과학(Data Science)](https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%82%AC%EC%9D%B4%EC%96%B8%EC%8A%A4)의 정의를 살펴보면 정형, 비정형 형태를 포함한 다양한 데이터로부터 지식과 인사이트를 추출하는데 과학적 방법론, 프로세스, 알고리즘, 시스템을 동원하는 융합분야이다.

뭔가 말이 좀 어렵고 광범위하다. 내가 느끼기에 디지털포렌식 분야에서 Data Science를 적용시킨다는 것은 정제되어 있지 않은 데이터를 이용하여 가공하여 또 다른 데이터를 산출하는 것을 의미한다.

이번 포스트 시리즈에서는 Data Science가 포렌식 분야에서 어떻게 활용될 수 있는지 필자의 실제 경험을 공유하고 이를 바탕으로 활용범에 대해 살펴보고자 한다.

## Experience in DVR Case
### Limitations in DVR Analysis
DVR(CCTV)은 제조사마다 다르지만 분석하다 보면 복원된 영상 프레임 자체를 파일로 조립하기 어려운 경우가 있다.

영상 프레임 자체에 채널, 시간, 크기 등 정보가 있으면 가장 좋겠지만 그렇지 않는 구조도 있기 때문이다. 아래의 예시를 살펴보자.

<p align="center">
  <img src="https://i.imgur.com/jaNA5VT.png" alt="image"/>
</p>

특정 DVR의 디렉터리 구조를 살펴보면 00 ~ 07 폴더는 채널을 의미한다. 즉, 00이 CCTV 카메라 1번 채널을 시작으로 07이 CCTV 카메라 8번 채널을 의미한다. 그리고 각 채널 폴더 내부에는 채널별 영상 파일이 담겨있다.

첫번째 채널의 하위 트리구조를 조금 더 살펴보자.

<p align="center">
  <img src="https://i.imgur.com/nR9FHUt.png" alt="image"/>
</p>

눈썰미가 좋다면 바로 알아봤겠지만, 해당 폴더 구조는 **채널\년\월\일\시각**순으로 폴더가 구성되어 있다.
그리고 ##.audio, ##.index, ##.video 3개의 파일이 쌍으로 저장된 구조를 지니고 있다. 여기서 ##.video가 녹화된 영상 파일이다.

DVR은 차량용 블랙박스 영상과는 다르게 avi, mp4와 같은 일반적인 동영상 컨테이너로 저장되지 않는다. 대부분 제조사 자체 포맷을 사용하고 각 제조사 별로 그를 재생할 수 있는 플레이어가 따로 존재한다.

제조사 플레이어를 사용하지 않으려면 영상 파일의 구조를 파악해서 일반적인 avi, mp4와 같은 형태로 변환한 뒤에 재생해야 한다. 실무에선 이런 방식으로 구조를 분석한다.

위 그림처럼 폴더 트리구조를 통해 시각, 채널 정보를 확인할 수 있으나 영상 파일 내부 구조에는 오로지 영상 프레임 데이터만 저장되어 있는 형태이다.

<p align="center">
  <img src="https://i.imgur.com/0oS5mQg.png" alt="image"/>
</p>

이럴 경우에는 비할당 영역에서 특정 날짜/시간을 정하여 영상 형태로 복원이 불가능한 모델이며, 변환 가능한 프레임 시그니처를 검색하여 JPG 이미지 형태로 변환만 가능하다는 구조적인 한계가 있다.
따라서, JPG 이미지 형태로 변환하고, 개별 해시값을 산출하여 전달해야하는 상황이다.

만일 영상으로 복원이 가능한다고 하더라도 언제 촬영된 영상인지 확인 할 수 있는 방안이 없다. 한 프레임만 이미지로 변환해서 살펴보자.

<p align="center">
  <img src="https://i.imgur.com/ceynJu9.png" alt="image"/>
</p>

이미지 파일로 변환해도 채널이 뭔지, 해당 영상이 촬영된 시간이 언제인지 알 수 있는 방법이 없다.

활성화 된 영상파일이라면 폴더 트리 구조별로 채널, 시간 값 등 파악을 할 수 있겠지만, DVR에서의 중요한 부분은 활성화된 영상 파일이 아닌 삭제된 영상의 복원이다.

프레임 복원은 비(미)할당 영역, 슬랙 영역을 대상으로 하는데 복원된 프레임에는 폴더 정보가 없기 때문이다.

### Frame Recovery Procedures & Difficulties
경험상 약 2.8TB의 비할당 영역에서 복원되는 영상 프레임 양은 1600만건 정도이다. 1600만 개의 파일을 추출하려면 다음과 같은 포렌식 절차를 거친다.

1. 비할당 영역을 대상으로 영상 프레임을 복원하는 스크립트(Python, EnScript 등)를 작성
2. 복원할 때 적절한 개수(ex. 10,000개 단위)로 영상 프레임을 폴더에 나누어 저장  
    - 하나의 폴더에 너무 많은 파일을 담을 시 프리징 현상 발생
3. 복원된 영상 프레임 해시값 산출  
    - 1600만개의 파일 해시값을 산출하려면 적절한 도구 필요 (EnScript 등)
    - 해시값을 계산하더라도 적절한 행 단위(ex. 약 100,000행)로 분리하여 CSV 파일로 저장
    - 서드파티 도구를 이용할 경우 대량의 파일이 정상적으로 처리가 된다는 보장이 있어야 하고 분리 저장이 파일의 분리 저장이 가능한 도구가 필요함
4. 영상 프레임 및 해시값 파일을 압축하여 제공

이렇게 EnScript를 사용하여 CSV 파일로 저장한 모습이다. 그런데 엑셀을 이용하여 파일을 열었는데,, 한글이 깨져보이는 문제가 발생했다.

<p align="center">
  <img src="https://i.imgur.com/gxFbeSb.png" alt="image"/>
</p>

EnScript에서 파일 저장할 때 인코딩 방식이 잘못 지정했나 생각했지만 막상 [Notepad++](https://notepad-plus-plus.org/downloads/) 도구를 이용해서 보니까 한글이 또 정상적으로 나왔다.

문제는 EnScript에서 파일 저장 시, CodePage(UTF8) 설정을 하더라도 Excel에서 읽어들일 때 정상적으로 못 보여주는 문제점이 있었다. 어설픈 스크립팅 실력 때문에 그런지도 모르겠다. (EnScript의 CodePage CP949는 결국 못 찾았다.)

아무튼 엑셀에서 데이터를 읽어올 때 인코딩 설정을 **949 : 한국어**로 해주면 한글이 정상적으로 보이나 파일을 불러들일 때 마다 설정을 바꿔줘야 한다.

<p align="center">
  <img src="https://i.imgur.com/G54CX9v.png" alt="image"/>
</p>

그러니까 계산을 해보면 10만 행이 있는 csv 파일이 전체 162개있었으니.. 162번 수동으로 해주면 된다!

여기서 나에게 선택지가 있었다.

1. 스크립트를 어떻게든 수정해서 다시 해시값 산출하고 CSV를 뽑는다.
2. 162개의 CSV를 엑셀에서 일일이 수작업으로 변환한다.

일단 2번은 아닌 것 같고,, 그런데 가만히 생각해보니 깨진 부분이 굳이 필요 없는 거라 문자열을 잘라내도 되는 것이었다. 즉, 수정 후의 경로는 복원된 영상 프레임의 저장된 폴더명만 있으면 되는 것이었다.

[수정 전 경로]  
2019-####\C\2019-####(!@$\#!$^&*)\ExportFrames\\<span style="color:red">**RecoverFrame001\RecoverFrame_0001.jpg**</span>

[수정 후 경로]  
<span style="color:red">**RecoverFrame001\RecoverFrame_0001.jpg**</span>

이제 총 162개의 CSV 파일을 어떻게 다룰 것인가가 문제였다.

당시 이 고민을 할 때가 금요일이었는데 나에게 주말을 보내던 중 갑자기 Data Science가 떠올랐다.

## Data Refinement with Pandas
내가 Pandas를 접한 건 예전에 우연찮게 데이터 분석을 Pandas 패키지로 많이 사용한다는 것을 눈팅으로 알게되었다.

본 케이스를 실무에 적용시키 전 일단 나도 잘 모르니 워밍업이 필요했다. 그래서 테스트 데이터를 만들어서 실험한 다음 케이스에 적용을 하려고 생각했다.

### What is Pandas?
우리가 가장 많이 접하는 데이터는 대부분 엑셀의 스프레드시트 형태일 것이다. 이는 통상적으로 행과 열로 되어 있는 자료구조이다.

Pandas는 파이썬(Python)에서 사용하는 데이터분석 라이브러리로, 행과 열로 이루어진 데이터 객체(Dataframe)를 만들어 다룰 수 있게 되며 보다 안정적으로 대용량의 데이터들을 처리하는데 매우 편리한 도구이다.

Pandas는 웨스 맥키니가 투자 운용 회사인 AQR에 재직중이던 2008년 초에 개발하였고 현재 Python으로 데이터 과학을 하기 위해서 없어서는 반드시 필요한 도구가 되었다.

### Pandas Basics

일단 테스트 데이터를 만들었다. 전화번호 컬럼에서 앞 010번호를 자르고 뒷자리 번호만 남길 예정이다.

<p align="center">
  <img src="https://i.imgur.com/g4BvuJu.png" alt="image"/>
</p>

참고로 Jupyter Notebook 환경에서 테스트를 하여 일반 VSCode, PyCharm 등과 같은 IDE에서 보려면 print문을 넣어줘야 한다.

패키지를 임포트 하고 데이터를 로드하는 방법은 다음과 같다. 

<p align="center">
  <img src="https://i.imgur.com/c8sTY0G.png" alt="image"/>
</p>

전체 행, 열 수를 확인하려면 보통 shape로 확인한다. 총 7행 3열로 구성된 DataFrame이다.

<p align="center">
  <img src="https://i.imgur.com/B8pQHZ6.png" alt="image"/>
</p>

전체 행을 <span style="color:red">**for loop로 순회**</span>하면서 전화번호 뒷자리만 잘라내는 작업을 한다. 그런데, 이게 pandas를 쓰는 <span style="color:red">**가장 바보같은 방법**</span>이다. 이유는 실제 데이터에 적용할 때 다시 언급하도록 하겠다.

<p align="center">
  <img src="https://i.imgur.com/SHqu1aR.png" alt="image"/>
</p>

Dataframe에서 수정된 데이터가 저장됐는지 살펴보면 정상적으로 적용된 것이 확인된다.  
df.head() 메소드는 Dataframe의 상위 5개의 행만 보여주는데 숫자를 넣으면 그 숫자 행만큼 보여준다.

<p align="center">
  <img src="https://i.imgur.com/knKZ3bM.png" alt="image"/>
</p>

이제 데이터가 수정됐으니 파일로 저장이 필요하다. 파일은 csv 형태로도 저장할 수 있고 엑셀로도 저장할 수 있다.

파라미터 중 **index=False**로 한 이유는 맨 앞에 있는 자동으로 부여되는 인덱스 숫자(0~6)를 저장할 데이터에 포함할지 말지를 결정하는 부분이다.
데이터를 로드할 때나 저장할 때 Encoding 방식이 중요한데, 저장할 때는 데이터에 따라 **CP949**나 **UTF-8** 등을 사용해도 되며 여기서는 CP949나 EUC-KR 아무거나 사용해도 상관없다.
<p align="center">
  <img src="https://i.imgur.com/7kKkudU.png" alt="image"/>
</p>

저장된 파일을 열어보면 CP949 인코딩 방식으로 저장된 글자를 잘 보여준다.

<p align="center">
  <img src="https://i.imgur.com/Aco9NUB.png" alt="image"/>
</p>

만일 데이터를 저장할 때 utf-8로 저장하면 어떻게 될까?  
엑셀에서는 기본적으로 인코딩 utf-8로 저장된 문서에 대해 깨진 글자로 표현된다. 물론 엑셀에서도 위에서 언급한 대로 수동으로 변경하면 글자를 정상적으로 읽을 수 있다.

> 참고) Excel 2016으로 테스트를 하였으며 다른 버전에서는 기본 인코딩이 무엇인지 테스트를 해보지 않았다.

<p align="center">
  <img src="https://i.imgur.com/gQkgP7s.png" alt="image"/>
</p>

notepad++에서는 굳이 변환을 안해줘도 정상적으로 한글을 표현해 준다.

<p align="center">
  <img src="https://i.imgur.com/Ip4nbro.png" alt="image"/>
</p>

### Python Full Code
아래 Python 코드를 확인해도 되고 라인별로 실행결과를 보고 싶다면 업로드한 [Jupyter Notebook](https://github.com/kyl3song/Python-Code-Snippet/blob/master/Github_Blog/Blog14/pandas_basic.ipynb)을 참고하면 된다.


``` python
import pandas as pd

df = pd.read_csv('pandas_data.csv', encoding = 'euc-kr')

row_counts, col_counts = df.shape

for i in range(row_counts):
    print(df["전화번호"][i][4:])
    df["전화번호"][i] = df["전화번호"][i][4:]

df.to_csv("output_csv_EUC_KR.csv", encoding='euc-kr', index=False)
df.to_csv("output_csv_CP949.csv", encoding='cp949', index=False)
df.to_csv("output_csv_UTF8.csv", encoding='utf-8', index=False)
df.to_excel("output_excel.xlsx", encoding='euc-kr', index=False)
```

## Wrap-up
DVR Case에서 발생할 수 있는 문제점과 이를 해결하기 위해 데이터를 다루기 위한 Pandas 패키지를 알아봤다. 또한 실무에 적용하기 전 워밍업으로 기본 코드를 살펴보았다.

다음 포스트에서는 실무에 적용시키고 옵션에 따른 시간적인 차이가 얼마나 나는지 확인할 예정이다.


## Reference
- <https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%82%AC%EC%9D%B4%EC%96%B8%EC%8A%A4>
- <https://dandyrilla.github.io/2017-08-12/pandas-10min/>


## Copyright (CC BY-NC 2.0 KR)
본 게시글은 CC BY-NC 2.0 KR Licence를 따릅니다.  
비영리 목록으로만 사용할 수 있고, 저작자와 출처를 표시하면 언제든지 게시글을 자유롭게 사용할 수 있습니다.

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
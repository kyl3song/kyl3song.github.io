---
title : "Blog #15: Data Science in Digital Forensics (DVR Case) part 2"
category :
  - Data Science
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - CCTV
  - DVR
  - Pandas
  - Numpy
  - UTF-8-SIG
  - UTF-8
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
디지털포렌식에서의 데이터 과학 (DVR Case) part 2

## Data Science in Digital Forensics Series
- [Blog #14: Data Science in Digital Forensics (DVR Case) part 1](https://kyl3song.github.io/data%20science/Data-Science-in-Digital-Forensics-(DVR-Case)-part-1/)

## Issue Review
이번 포스트에서는 이슈 사항에 대해 원인을 파악하고 이슈를 해결할 수 있는 여러가지 방법을 알아보도록 하겠다. 그 중에 Data Science를 사용하는 방법도 포함되어 있다.

지난번 EnScript로 CSV 파일을 추출할 때 EnScript에서 제공하는 Codepage UTF-8를 사용하여 저장하였고, 해당 CSV 파일을 엑셀 프로그램에서 바로 열었을 때 엑셀이 바로 읽어 들일 수 있는 인코딩과 맞지 않아 한글이 깨지는 이슈가 발생하였다.

일단 조금 더 깊게 이슈가 발생한 원인에 대해 파악해보자. 대체 UTF-8 인코딩 방식으로 저장했는데 왜 깨질까?

notepad++ 도구로 CSV 파일의 인코딩을 확인해 보면, <span style="color:red">**UTF-8 w/o(without) BOM**</span>이라고 되어있다.

<p align="center">
  <img src="https://i.imgur.com/OwNjMS5.png" alt="image"/>
</p>

UTF-8 뒤에 붙어 있는게 뭔가 좀 수상한데 이럴 땐 메모장을 이용하여 테스트용 CSV 파일을 만들어 보자.

윈도우의 메모장에는 파일을 저장할 때 인코딩을 선택할 수 있는 옵션이 있는데 ANSI, 유니코드, 유니코드(big endian), UTF-8 총 4개의 옵션이 존재한다.

<p align="center">
  <img src="https://i.imgur.com/NQyGsQi.png" alt="image"/>
</p>

그 중에서 인코딩 UTF-8로 csv 파일을 하나 만들어 생성된 CSV 파일을 엑셀로 열어보자. 한글이 깨짐없이 깔끔하게 잘 읽힌다.

<p align="center">
  <img src="https://i.imgur.com/8f98KfL.png" alt="image"/>
</p>

여기서 같은 UTF-8인데 왜 다를까?라는 의문이 든다. 테스트로 만든 CSV 파일을 notepad++에서 인코딩 방식을 확인했더니 UTF-8이다.

<p align="center">
  <img src="https://i.imgur.com/lrQFgnU.png" alt="image"/>
</p>

결국 UTF-8이랑 UTF-8 without BOM의 차이가 이 문제가 발생된 원인으로 요약될 수 있다. UTF-8 without BOM은 BOM이 없다는 것을 의미하니 **UTF-8 BOM**이 무엇인지 확인을 해보자.


## UTF-8 BOM(Byte Order Mark)
유니코드(Unicode) 인코딩에는 UTF-8, UTF-16, UTF-32 등 여러가지 방식이 있으며 이는 우리가 다양한 언어를 표현할 수 있도록 해준다.

BOM은 인코딩된 문서 첫 머리에 사용되어 정확한 인코딩 방식을 알려주는 역할을 하는데 대표적인 인코딩 방식과 그에 따른 BOM 목록은 아래 표와 같다. [(출처)](http://blog.wystan.net/2007/08/18/bom-byte-order-mark-problem)

|인코딩 방식          |Byte Order Mark(BOM)|
|:------------------:|:------------------:|
|UTF-8               |<span style="color:red">**EF BB BF**</span>|
|UTF-16 Big Endian   |FE FF               |
|UTF-16 Little Endian|FF FE               |
|UTF-32 Big Endian   |00 00 FE FF         |
|UTF-32 Little Endian|FF FE 00 00         |

UTF-8의 BOM은 데이터 가장 첫 3byte를 차지하고 있으며 **EF BB BF**로 정해져 있다. 그런데 UTF-8은 인코딩 형식이 고정되어 있기 때문에 굳이 BOM이 없어도 인코딩 방식을 자동으로 알아낼 수 있다.

따라서 BOM이 없어도 상관없기 때문에 BOM을 UTF-8 Signature라고 부르기도 한다. 즉, 해당 문서가 UTF-8로 인코딩 되었다는 사실을 알리는 시그니처의 역할을 하는 것이다.

데이터를 직접 열어서 비교해보자.

**[엑셀에서 한글이 깨지지 않는 CSV]**

<p align="center">
  <img src="https://i.imgur.com/yDiBblO.png" alt="image"/>
</p>

**[엑셀에서 한글이 깨지는 CSV]**

<p align="center">
  <img src="https://i.imgur.com/eWl60eN.png" alt="image"/>
</p>

메모장의 UTF-8 인코딩 방식으로 생성했던 UTF8.csv 파일은 최초 3바이트에 BOM이 기재되어 있다. 하지만 한글이 깨지는 즉, 내가 EnScript로 생성한 CSV 파일은 BOM이 없다. (UTF-8 w/o BOM)

그럼 엑셀 프로그램에서는 BOM이 있어야지 정상적으로 한글을 인식한다는 의미를 갖는다.

좀 더 구글링을 해보면 많은 소프트웨어가 BOM이 있어야 정상적으로 인식할 수 있으며, 또 다르게 BOM이 있으면 정상적으로 인식이 안되는 소프트웨어가 있다고 하니 참고하면 좋을 것 같다. [(출처)](https://topic.alibabacloud.com/a/the-difference-between-font-colorredutffont-font-colorred8font-and-font-colorredutffont-font-colorred8font-without-bom_4_86_30950132.html)


<p align="center">
  <img src="https://i.imgur.com/8WzNk9a.png" alt="image"/>
</p>

다시 본론으로 돌아와서 엑셀은 BOM이 있어야만 정상적으로 한글을 인식할 수 있도록 되어있는 것으로 판단할 수 있다.

그리고 소프트웨어에 따라서 BOM이 있을경우 이를 자동으로 제거하고 문자열을 인식할 수 있는 프로그램도 있는 반면에 자동으로 제거하는 로직이 없어서 그냥 깨진 문자열 상태로 보여주는 프로그램도 있다.

그럼 BOM에 해당하는 헥사값(0xEFBBBF)을 깨져보이는 문서에 추가하고 엑셀에서 정상적으로 읽힐 수 있을지 확인해보자.

<p align="center">
  <img src="https://i.imgur.com/bS8tw1t.png" alt="image"/>
</p>

한글이 깨지지 않고 잘 읽힌 것으로 보아 문제의 원인 파악이 되었으며 해결 포인트를 얻었다.

<p align="center">
  <img src="https://i.imgur.com/f3q3Mls.png" alt="image"/>
</p>


## How to workaround
정리하면 한글이 깨져보이는 이슈를 해결할 수 있는 방법은 여러가지가 있다.

1. CSV 파일의 맨 앞에 BOM 헥사값을 추가한다.
2. 인코딩 방식을 다시 지정한다.
3. 깨진 한글의 스트링 부분을 제거한다.

모든 부분을 수동으로 변경할 수 있겠지만 많은 파일을 대상으로 생각한다면 코딩으로 자동화가 필요한 부분이다. 하나씩 살펴보도록 하자.

## Add BOM to CSV
첫 번째는 현재 BOM이 없는 상태에서 엑셀에서 읽을 수 있게 BOM 헥사값을 추가해 주는 방식이다.

``` python
# -*- coding: utf-8 -*-

file = 'Export File List_001.csv'
output_file = 'addBOM_result.csv'

with open(file, mode='rb') as fh_read:
	data = fh_read.read()
	
	bom = b'\xEF\xBB\xBF'
	with open(output_file, mode='wb') as fh_write:
		fh_write.write(bom)
		fh_write.write(data)
```

## Convert File Encoding
두 번째는 UTF-8로 텍스트를 읽은 후 인코딩 방식을 **utf-8-sig**(UTF-8 with BOM)으로 변경해서 저장하면 간단하게 해결할 수 있다.
다른 인코딩 방식인 ANSI, EUC-KR, CP949를 지정하여 저장해도 정상적으로 깨지지 않는 것을 확인하였다.

``` python
# -*- coding: utf-8 -*-

file = 'Export File List_001.csv'
output_file = 'convert_enconding.csv'

with open(file, mode='r', encoding='utf-8') as fh_read:
	data = fh_read.read()
	
	with open(output_file, mode='w', encoding='utf-8-sig') as fh_write:
		fh_write.write(data)
```

## Refine Data with Pandas
마지막 세 번째로는 인코딩 방식을 아예 신경 쓰지 않고 Data Science를 이용하여 이슈를 해결해보자. 이전 포스트에서 수정이 필요한 경로를 다음과 같이 정리했었다.

**[수정 전 경로]**  
2019-####\C\2019-####(!@$\#!$^cjd)\ExportFrames\\<span style="color:red">**RecoverFrame001\RecoverFrame_0001.jpg**</span>

**[수정 후 경로]**  
<span style="color:red">**RecoverFrame001\RecoverFrame_0001.jpg**</span>

즉 어차피 깨진 문자열을 잘라낼거면 아예 경로 중 필요 없는 앞 부분을 다 잘라내고 프레임이 들어있는 필요한 경로만 남길 필요가 있는 경우에 해당한다. 나 역시 이 방법을 선택하였다.

CSV 파일을 읽을 때 해당 파일이 UTF-8로 명시적으로 인코딩을 지정해 준다. 정확하게는 UTF-8-SIG로 줘야 하지만 Pandas에서는 UTF-8로 해도 오류 없이 불러온다.

<p align="center">
  <img src="https://i.imgur.com/u83a2gp.png" alt="image"/>
</p>

Dataframe의 행, 열을 확인하고 ItemPath 컬럼에서 잘라내고 싶은 스트링 모양을 만든다.

<p align="center">
  <img src="https://i.imgur.com/PFkKBCn.png" alt="image"/>
</p>

Python의 반복문을 돌려 CSV 파일을 처리하는데 시간을 계산하고 처리된 결과를 확인해보자.

<p align="center">
  <img src="https://i.imgur.com/ntb8TXy.png" alt="image"/>
</p>

<p align="center">
  <img src="https://i.imgur.com/tTO2YP7.png" alt="image"/>
</p>

경로 변경이 완료되었다. 여기서 주의 깊게 봐야 할 부분은 시간인데 1개의 파일을 처리하는데 약 1분이라는 시간이 걸렸다.

그럼 총 162개의 파일이 있으니 대략 계산하면 **약 162분(2시간 42분)** 걸린다고 생각해도 될 것 같다.

위와 같은 전통적인 Python의 For 문법을 사용하는 것보다, Pandas 자체에서 제공하는 메서드(method)들을 사용하면 더욱 빠르게 처리할 수 있는데, 그중 하나인 열(colum)을 처리하는 apply 메서드를 사용하는 것이다.

조금 더 찾아보니 데이터 프레임의 특정 축(행 또는 열)을 따라 함수를 적용하여 사용하는 방법으로, apply()는 본질적으로 행을 반복하지만 Cython에서 이터레이터를 사용하는 것 같이 내부 최적화를 다양하게 활용하므로 훨씬 효율적이라고 한다. [(출처)](https://aldente0630.github.io/data-science/2018/08/05/a-beginners-guide-to-optimizing-pandas-code-for-speed.html)

그럼 aplply 메서드를 사용해서 적용한 다음 시간이 얼마나 차이나는지 확인해보겠다.

<p align="center">
  <img src="https://i.imgur.com/NCmsuyL.png" alt="image"/>
</p>

이번 코드에서는 수정된 Dataframe을 CSV로 저장하는 부분을 집어넣었는데도 불구하고 CSV 파일 1개를 처리하는데 1초도 안 걸렸다.

파일로 저장할 때 수정된 부분을 통해 한글이 없기 때문에 인코딩을 UTF-8, UTF-8-SIG, ANSI, CP949, EUC-KR로 해도 상관없지만, 만일 한글이 있다면 **UTF-8-SIG, ANSI, CP949, EUC-KR** 인코딩으로 추출하면 엑셀에서 잘 읽힌다.

이번에는 162개의 파일을 처리하고 다른 이름으로 CSV 파일을 저장하는 시간을 확인해봐야겠다.

<p align="center">
  <img src="https://i.imgur.com/0DnkEeo.png" alt="image"/>
</p>

수행하는 데 걸리는 시간 총 **2분 16초**로 162개의 파일을 다룰 수 있었다.

for loop(2시간 42분)와 apply(2분 16초) 메서드를 이용한 시간을 비교하자면 약 **72배** 속도가 차이난다.

사실 워크스테이션에서 테스트 한 부분이라 일반 PC에서 for loop을 사용할 경우 속도가 현저히 더 느려져서 속도 차이가 더 날 수 있다. 그리고 본 테스트는 Jupyter Notebook을 이용하여 속도가 일반 IDE에서 실행하는 것보다 좀 더 느리다.

내가 예전에 실험한 PC에서는 100배 이상 차이가 났었다.


### Python Full Code

```python
# -*- coding: utf-8 -*-

import pandas as pd
import glob
import os
import datetime

start_time = datetime.datetime.now()

target_file_list = glob.glob('csv_files\\*.csv')

for target in target_file_list:
    
    df = pd.read_csv(target, encoding='utf-8')
    df['ItemPath'] = df['ItemPath'].apply(lambda x: x[42:])

    output_file = os.path.splitext(target)[0] + '_HASH.csv'
    df.to_csv(output_file, encoding='cp949', index=False)    

end_time = datetime.datetime.now()

print(end_time - start_time)
```

## Wrap-up
지금까지 EnScript(Codepage UTF-8)로 생성한 CSV 파일이 엑셀에서 한글이 깨지는 이슈를 확인하고, 원인 파악 후 다양한 해결 방법에 대해 알아보았다.

특히 해결 방법 중 Data Science를 활용하는 방법은 디지털포렌식 분야뿐 아니라 다양한 분야에서 적용될 수 있을 것이다.

Pandas에서 단순 반복은 절대 하지 말아야 한다고 많은 사람들이 이야기하고 있다.
이전 포스트에서 for loop로 순회하면서 Dataframe을 처리하는 것은 매우 바보 같은 방법이라고 이야기했던 이유이기도 하다.

만일 apply 메서드를 사용하지 못하는 상황이고 무조건 반복을 돌려야 할 상황이라면 Pandas에 최적화된 iterrows 메서드도 있으니 참고하자.

사실 속도는 apply 메서드보다 Pandas Series 벡터화를 하거나 Numpy Array를 사용한 벡터화를 하는 것이 더 빠르다고 하니 참고해서 나중에 적용할 부분이 있으면 사용할 예정이다.


## Reference
- <https://topic.alibabacloud.com/a/the-difference-between-font-colorredutffont-font-colorred8font-and-font-colorredutffont-font-colorred8font-without-bom_4_86_30950132.html>
- <http://blog.wystan.net/2007/08/18/bom-byte-order-mark-problem>
- <https://blog.naver.com/stonefly2001/221289234878>
- <https://docs.python.org/3/library/codecs.html#standard-encodings>
- <https://aldente0630.github.io/data-science/2018/08/05/a-beginners-guide-to-optimizing-pandas-code-for-speed.html>


## Copyright (CC BY-NC 2.0 KR)
본 게시글은 CC BY-NC 2.0 KR Licence를 따릅니다.  
비영리 목록으로만 사용할 수 있고, 저작자와 출처를 표시하면 언제든지 게시글을 자유롭게 사용할 수 있습니다.

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
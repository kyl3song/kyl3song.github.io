---
title : "Blog #17: IoT Forensics (Log Refinement w/ Data Science) part 2"
category :
  - IoT Forensics
  - Data Science
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - IoT
  - Embedded Systems
  - Data Science
  - pandas
  - numpy
  - UART Log
  - Log Refinement
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
IoT Forensics (데이터 사이언스를 통한 로그 정제) part 2

## IoT Forensics Series
- [Blog #16: IoT Forensics (Acquisition) part 1](https://kyl3song.github.io/iot%20forensics/IoT-Forensics-(Acquisition)-part-1/)

지난 포스트 글 말미에서 UART + 터미널프로그램을 통한 메모리 획득 방법의 단점에 대해 언급을 하였다.

그중 시리얼 통신은 말 그대로 데이터를 직렬화 하여 한 비트씩 전송하는 방식이므로 데이터 전송 속도가 느린 부분은 개선 가능한 여지가 없는 어쩔 수 없는 부분이다.

그럼 putty/Xshell과 같은 터미널 프로그램으로 시리얼 통신 로그를 화면에 현출하는 경우 정확이 어떤 문제가 발생할 수 있는지에 대해 초점을 맞춰 해결 방안을 살펴보도록 한다.


## 용량별 사본 획득 및 로그 변환 시간 (Total Required Time of Acquisition)
시리얼 통신을 이용하여 메모리 덤프 로그를 획득하고 데이터 영역을 가공하여 바이너리로 변환하는 시간을 수차례 측정해본 결과 다음과 같이 확인할 수 있었다.

|플래시 메모리 용량       |로그 획득 시간           |바이너리 변환 시간        |
|:----------------------:|:----------------------:|:----------------------:|
|4MBytes                 |약 2시간                 |약 2분                  |
|16MBytes                |약 7 ~ 8시간             |약 5분                  |
|32MBytes                |약 14 ~ 15시간           |약 10분 ~ 11분          |


## 발생 가능한 이슈 (Possible Issues)
시리얼 통신을 기반으로 dump, md, xxd, hexdump 등 명령어를 이용하여 터미널 프로그램에 메모리의 정보를 현출하는 방법을 통해 획득한 바이너리는 정말 정확한 데이터일까? 정확할 수도 있고 그렇지 않을 수도 있다.

현재 시리얼 통신을 기반으로 한 사본 획득 과정에는 정작 가장 중요한 부분인 덤프에 대해 정합성 검증을 하는 과정이 빠져있다.

정합성 검증은 반드시 필요한 과정으로 분석관이 획득한 파일이 정상적으로 맞는 것인지 확인하여 신뢰성을 높이기 위한 필수 작업이다. 획득한 파일이 정상적이지 않으면 정확한 분석을 할 수 없기 때문이다.

덤프 로그를 생성할 때 경험한 부분을 바탕으로 발생 가능한 이슈 2가지를 살펴보자.

### 1. 데이터 역전 현상 (Data Reversed)
기기와 터미널로 연결된 상태에서 메모리 덤프를 화면상에 출력하다 보면 가끔 주소가 역전되는 현상이 발생하는 경우가 있다.

아래 그림에서는 데이터 1줄(16 bytes)이 전체 뒤집힌 경우인데 여러 줄이 통째로 뒤집힌 경우도 본 적이 있다.
주소가 역전된다는 의미는 데이터도 신뢰할 수 없다는 의미를 가지고 있다.

<p align="center">
  <img src="https://i.imgur.com/G2N9g58.png" alt="image"/>
<br>그림 1.  주소가 역전된 예시</p>

주소/데이터가 역전된 현상은 방대한 로그 데이터 속에서 알아차리기가 매우 어렵고 사람이 수작업으로 찾는 것은 거의 불가능하다.


### 2. 데이터 누락 (Data Missing)
두 번째로 발생 가능한 이슈는 덤프 진행 중 일부 덤프 데이터가 누락되는 경우와 첫 번째와 마찬가지로 한 줄 또는 여러 줄의 데이터가 누락되는 현상이 종종 발생되기도 한다.

<p align="center">
  <img src="https://i.imgur.com/erOQXjZ.png" alt="image"/>
<br>그림 2.  데이터가 누락된 예시</p>


## 정합성 검증 로직 (Consistency Checking)
위 두 가지 문제점을 대응하기 위해 로그의 정합성을 확인할 수 있는 방법이 있다면 신뢰할 수 있는 데이터를 얻을 수 있을 것이다.

첫 번째로 덤프 상 주소가 순서대로 증가하는지 검증하는 로직으로 주소 정합성을 체크할 수 있다.
주소가 순서대로 증가해야 되나 만일 주소값이 누락되거나 주소 역전이 발생하는 경우 정상적으로 로그가 쌓이지 않았다고 볼 수 있다.

<p align="center">
  <img src="https://i.imgur.com/vK2J4Ms.png" alt="image"/>
<br>그림 3. 주소 정합성 검증로직</p>

두 번째로 라인별 길이 검증 로직도 필요한데 다른 덤프 로그의 길이와 다르다면 획득된 로그의 신뢰성이 낮다고 판단할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/OfyDGlA.png" alt="image"/>
<br>그림 4. 라인별 길이 검증로직</p>

정리하면 덤프 로그를 바이너리 파일로 변환하기 전 라인별 길이를 판단하는 <span style="color:red">**가로 정합성**</span>과, 메모리의 주소가 순차적인지를 판단하는 <span style="color:red">**세로 정합성**</span>을 먼저 판단해서 정합성이 모두 맞을 때 바이너리 파일로 변환을 해야 할 것이다.

만일 로그의 정합성이 맞지 않을 경우 다시 시리얼 로그를 획득 후 정합성 검증을 또 해야 하는데 이럴 경우 '획득 + 검증 + 바이너리 변환' 단계에 따라 시간이 더 늘어날 가능성이 있다.

일반적인 프로그래밍은 텍스트 형태의 로그 데이터를 한 줄씩 읽어 처리한 뒤 가공된 데이터를 차례로 저장하는 방식이다. 대용량 로그를 한 줄씩 반복하여 처리할 경우 반복횟수가 증가할수록 처리 시간이 길어진다는 단점이 있다.

실제로 32MB 플래시 메모리의 덤프 로그를 바이너리로 변환하면 약 10~11분 정도 소요된다. 이는 시리얼로그 획득 시간에 비하면 긴 시간이 아니나 가로, 세로 정합성까지 판단한다면 바이너리 변환까지 최소 3배 이상의 시간이 소요된다.

만일 정합성 이슈로 덤프 로그를 처음부터 다시 생성해야 한다면 시리얼 통신을 통해 또 14시간이라는 로그 획득 시간을 기다려야 한다.

따라서 기존의 프로그래밍 방식 대비 좀 더 빠른 방법으로 정합성을 검증할 필요가 있다.


## 데이터사이언스를 이용한 해결 방안 (Workaround using Data Science)
32MB 플래시 메모리 기준, 메모리 내용을 로그형태로 저장한 파일은 133MB의 파일 크기를 가지고 있고 내부 데이터는 약 2,100,000(210만)행으로 구성되며 약 140,000,000(1억 4천만)개의 문자로 이루어진 파일이다.

<p align="center">
  <img src="https://i.imgur.com/A62RFe0.png" alt="image"/>
<br>그림 5. 32MB 플래시 메모리 로그</p>

결과적으로 사본 파일로 변환하기 위해서는 이런 대용량의 텍스트를 빠르게 검증하고 정확하게 처리하는 작업이 필요하다. 대용량의 데이터를 효과적으로 다룰 수 있는 데이터사이언스 방식을 적용하여 이를 해결해보자.

데이터사이언스(Data Science)는 비정형적인 데이터에서 과학적인 방법, 프로세스, 알고리즘 등을 통해 실제 현상을 이해하고 분석하여 새로운 인사이트를 얻거나, 가치 있는 데이터를 얻는 방법론이다.

획득 및 분석 시간이 중요한 실무에서 대용량 데이터를 다루는 데이터 사이언스를 가능하게 해주는 도구 또는 프로그래밍 패키지(pandas, numpy, scipy 등)를 사용한다면 로그 데이터를 1차원인 한 줄씩 처리하여 가공하는 것이 아닌 2차원 데이터 즉, 행과 열로 구분된 Dataframe(데이터프레임)으로 처리하면 가공 속도를 향상시킬 수 있다.

결론적으로 정합성 검사를 빠르게 수행하여 데이터에 문제가 있다면, 문제가 있는 일부분만 재획득하는 방법을 사용할 수 있어 많은 시간을 절약할 수 있다.

본 포스팅에서는 정합성 이슈 해결을 위해 데이터 분석 및 정제용으로 Python의 numpy, pandas 패키지를 사용하였다.

참고로 지금 사용하는 방법보다 더 빠른 속도로 최적화를 구현하려면 Pandas Series 또는 Numpy Array를 이용한 벡터화(Vectorization)를 통해 행의 반복 없이 최소의 연산으로 구현하면 된다.

<p align="center">
  <img src="https://i.imgur.com/H36jJTb.png" alt="image"/>
<br>그림 6. 데이터 벡터화
<br>(Source: PyCon Korea 2019 - 오성우) </p>

먼저 이해를 돕기 위해 간단한 샘플 로그를 가지고 설명하려고 한다. 아래 샘플 덤프 로그는 일부러 주소 및 라인 길이도 정합성이 모두 맞지 않게 수정을 한 상태이다.

<p align="center">
  <img src="https://i.imgur.com/vsImcD5.png" alt="image"/>
<br>그림 7. 샘플 로그 (Flash_Dump_MD2.txt)</p>


### 1. 패키지 임포트
데이터 분석에 필요한 패키지들을 임포트 한다.

<p align="center">
  <img src="https://i.imgur.com/jqOLGR0.png" alt="image"/>
</p>


### 2. 로그 파일 읽기
샘플 로그 파일을 읽는다. 파일을 읽을 수 있는 방법은 다양하게 존재하지만, 여기서는 'read_csv()' 함수를 사용한다.

<p align="center">
  <img src="https://i.imgur.com/PxbCpLM.png" alt="image"/>
</p>

예전엔 간단히 'pd.read_table(target, names=['data'])' 함수를 이용하여 읽기도 하였으나 앞으로 없어질 함수(Deprecate Warning)로 read_csv를 사용하였다.

파일을 읽어서 df(dataframe) 변수에 저장한다.

<p align="center">
  <img src="https://i.imgur.com/9uJfonA.png" alt="image"/>
</p>

### 3. 특정 행 선택
원하는 Index 영역만 선택하는 게 필요할지 모르므로 덤프 영역만 선택하여 원하는 데이터프레임(df)을 다시 구성한다.

<p align="center">
  <img src="https://i.imgur.com/YS53s1x.png" alt="image"/>
</p>


### 4. 각 행에 대한 길이 계산
각 행에 대한 길이 값을 계산할 때 'apply()'를 사용한다. 'iloc'을 이용한 반복문 또는 'iterrows()' 반복을 사용하는 것 보다 훨씬 효율적이다.

길이를 계산하여 dataframe의 'len' 컬럼을 만들어 계산된 길이를 저장한다.

<p align="center">
  <img src="https://i.imgur.com/bl6fsHB.png" alt="image"/>
</p>


### 5. 길이 정합성 판단 (가로 정합성)
전체 데이터 중 길이가 65가 아닌 것만 따로 한번에 추출할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/PokVRjr.png" alt="image"/>
</p>

만일 길이가 64 이상인 데이터만 추출하려고 하면 간단히 부등호를 넣어주면 된다.

<p align="center">
  <img src="https://i.imgur.com/LTIcLhb.png" alt="image"/>
</p>


전체 데이터의 가로 정합성을 위해 Bool Type 형태의 반환이 필요하다.

<p align="center">
  <img src="https://i.imgur.com/7GUFW2K.png" alt="image"/>
</p>

조건을 통해 길이가 65가 아닌 모든 행에 대해 출력할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/ett7s1R.png" alt="image"/>
</p>

'all()' 메서드를 통해 모든 데이터의 길이가 65에 대한 True/False 여부를 확인할 수 있다.
샘플에서는 길이가 63인 데이터가 존재하므로 False를 출력한다.

즉, 하나라도 조건에 맞지 맞는 데이터가 있다면 False를 출력해준다. 이 구문을 통해 **데이터의 가로 정합성을 판단**할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/QzqTyJU.png" alt="image"/>
</p>


### 6. 주소 정합성 판단 (세로 정합성)
주소 부분을 데이터 프레임의 address 컬럼을 만들어 분리시킨다.

<p align="center">
  <img src="https://i.imgur.com/B9l89fv.png" alt="image"/>
</p>

'address' 컬럼은 개별 데이터가 문자열(str type)이기 때문에 문자열 비교를 위해 '0x'를 붙여준다.

<p align="center">
  <img src="https://i.imgur.com/AI4jbVE.png" alt="image"/>
</p>

덤프의 시작 주소, 끝 부분 주소를 이용하여 검증할 주소 데이터를 만든다. numpy 배열 이용하는 이유는 검증용 주소를 만들 때도 반복문을 사용하지 않기 위해서이다.

<p align="center">
  <img src="https://i.imgur.com/dNyc1EQ.png" alt="image"/>
</p>

로그에서 분리시킨 address_hex와 검증 용도의 address_compare의 문자열 비교를 통해 다른 경우 다른 부분만 필터링이 가능하며 이 부분을 **데이터의 세로 정합성을 판단**하는데 사용할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/OKug1jR.png" alt="image"/>
</p>


### 6. 데이터 정제 및 바이너리 파일 변환
덤프 로그는 그림과 같이 Offset(주소), 덤프 데이터, ASCII Text 영역으로 이뤄져있다.

<p align="center">
  <img src="https://i.imgur.com/VIZfrl5.png" alt="image"/>
</p>

바이너리 데이터로 만들기 위해서 Offset, ASCII Text 부분을 제외하고 필요한 덤프 영역만 추출 해야 한다. 'data' 컬럼을 대상으로 문자열을 나누고 붙이는 작업을 통해 그림처럼 한 줄 단위의 hex 데이터를 만들 수 있다.

<p align="center">
  <img src="https://i.imgur.com/YHVTl4t.png" alt="image"/>
</p>

정합성이 맞지 않는 샘플 로그를 대상으로 적용하면 9행이 문자열이 1바이트 만큼 모자란다. 물론 프로그래밍으로 이미 가로 정합성 부분에서 필터링 되어 여기까지 오지 않게 만들어야 할 것이다.

이제 각 행으로 되어 있는 데이터를 모두 하나의 줄로 합치는 작업을 한다.

<p align="center">
  <img src="https://i.imgur.com/pxWNZQm.png" alt="image"/>
</p>

스트링 타입으로 되어있는 문자열을 바이트열로 변환을 하고,

<p align="center">
  <img src="https://i.imgur.com/2eX1kex.png" alt="image"/>
</p>

마지막으로 저장하면 완료된 모습을 확인할 수 있다.
<p align="center">
  <img src="https://i.imgur.com/5lkuzdk.png" alt="image"/>
</p>


그럼 133MB의 실제 데이터를 위와 같은 작업을 통해 로그 파싱, 정제, 변환 작업까지 얼마나 걸리는지 확인해보자.
전체 Python 코드는 맨 아래에 첨부하였다.

<p align="center">
  <img src="https://i.imgur.com/u9QfImr.png" alt="image"/>
<br>그림 7. Python Script 실행 결과</p>

전체 프로세스를 수행하는데 걸린 시간은 **약 8.4초** 정도 소요되었다.

지금은 32MB의 플래시 메모리 덤프 로그지만 128MB과 같이 대용량의 메모리 로그를 변환하면 이보다 더욱 큰 차이를 보일 수 있고, 무엇보다 이 방법을 사용함으로써 장점은 가로/세로 정합성의 판단은 거의 몇 초 이내에 이뤄진다.

따라서 빠르게 판단하고 로그가 부정확 하다면 명령어를 통해 부정확한 곳의 주소에서 명령어를 통해 원하는 부분만 다시 덤프를 뜰 수 있을 것이다.

정합성이 맞지 않는다면 과연 어떻게 처리할 수 있을까?

나의 경우는 어디가 맞지 않는지 보여주도록 코드를 작성하였다. 그리고 정합성이 맞지 않을 경우 어차피 데이터는 증거로써 가치가 없기 때문에 바로 프로그램이 종료되도록 하였다.

코딩은 사람의 생각에 따라 작성하는게 다르기 때문에 더 좋은 아이디어가 있다면 그대로 구현해보길 추천한다.

<p align="center">
  <img src="https://i.imgur.com/p4fKxQh.png" alt="image"/>
<br>그림 8. 가로 정합성이 맞지 않을 경우
</p>


``` python
# -*- coding: utf-8 -*-
# @Author: Kyle Song (fkilla8210@gmail.com)
# @Last Modified by:   KyleSong

import datetime
import pandas as pd
import numpy as np
import os
import logging
import sys


DATA_START_ROW_INDEX = 1
DATA_END_ROW_INDEX = 999999999999999999
ONE_LINE_LENGTH_OF_DUMP = 65

class Config:
    # Target File
    CASE_NUMBER = "2020-0001-1"
    TARGET_FILE = "md.l_log.log"
    BASE_PATH = f"{os.path.dirname(TARGET_FILE)}"

    # Log, Save Path
    LOG_PATH = BASE_PATH
    LOG_NAME = f"{CASE_NUMBER}_IxTractor.log"

    # Binary File Save Path
    SAVE_PATH = BASE_PATH
    SAVE_FILE_NAME = f"{CASE_NUMBER}_flashdump.bin"
    SAVE_FILE = os.path.join(SAVE_PATH, SAVE_FILE_NAME)

def main():
    global DATA_START_ROW_INDEX, DATA_END_ROW_INDEX
    target_file = Config.TARGET_FILE
    dest_file = Config.SAVE_FILE
    log_path = Config.LOG_PATH
    log_name = Config.LOG_NAME

    logger = get_logger(log_path, log_name)

    str_start_time = datetime.datetime.now()
    logger.info(f"==== JOB START ==== Start Time : {str_start_time}")

    # Configure very first row as 'data' column name
    logger.info(f"Reading Raw Files({os.path.basename(target_file)})...")
    df = pd.read_table(target_file, names=['data'], encoding='utf-8')
    logger.info("Loading Done !")

    # Get Specific Row and Columns
    df = df.iloc[DATA_START_ROW_INDEX-1: DATA_END_ROW_INDEX]
    row_counts, column_counts = df.shape
    logger.info(f"Total Row Counts: {row_counts}, Column Counts : {column_counts}")

    # Length Verify
    length_verify_flag, df = verify_length(df, logger)
    if length_verify_flag == False:
        logger.critical('ERROR - Each Row Length of FlashDump is not Verified !!!')
        sys.exit()

    # Address Verify
    str_start_address = df['data'][DATA_START_ROW_INDEX-1].split(':')[0]
    str_end_address = df['data'][DATA_START_ROW_INDEX+row_counts-2].split(':')[0]
    int_start_address = int(str_start_address, 16)
    int_end_address = int(str_end_address, 16)

    address_verify_flag, df = verify_address(df, int_start_address, int_end_address, logger)
    if address_verify_flag == False:
        logger.critical('ERROR - Address offset of FlashDump is not Verified !!!')
        sys.exit()

    # Dump Logs shown like as follows:
    # c0000000: ffdd0022 10000000 040800bc 00000060    ..."...........'
    # c0000010: 010800bc 00000060 ffdd0022 80000000    .......'..."....
    logger.info("Preprocess raw data is in progress...")
    df['data'] = df['data'].apply(lambda x: (x.split(':')[1].strip().split('  ')[0].replace(" ", "")))
    logger.info(f"The First Five Row Data look like:\n{df['data'][:5]}")
    hex_string = "".join(df['data'].tolist())

    byte_encoded = bytes.fromhex(hex_string)

    savefile(byte_encoded, dest_file)

    str_end_time = datetime.datetime.now()
    logger.info("Job Finished !!!")
    logger.info(f"==== JOB START ==== Start Time : {str_start_time}")
    logger.info(f"==== JOB END ==== End Time : {str_end_time}")
    logger.info(f"Elasped Time : {str_end_time - str_start_time}")

def verify_length(df, logger):
    global ONE_LINE_LENGTH_OF_DUMP

    # Calculate DataFrame Length and make Series type value
    df['len'] = df['data'].apply(len)
    logger.info(f"The First Five Row Data Frame:\n{df[:5]}")

    # verify_bool will be True if all values are True
    verify_bool = (df['len'] == ONE_LINE_LENGTH_OF_DUMP).all()
    logger.info(f"All Rows' Length Verified (True/False): {verify_bool}")

    if verify_bool == False:
        # 'isin' only takes List Type args (~isin = NOT isin)
        unmatch_df = df[~df['len'].isin([ONE_LINE_LENGTH_OF_DUMP])]
        logger.critical(f"Unmatched LENGTH DataFrame List:\n{unmatch_df}")

    return verify_bool, df

def verify_address(df, int_start_address, int_end_address, logger):
    # Remove 'len' Column as it is no longer to use
    df.drop("len", axis=1)

    # Split address part of 'data' and add them to 'address' column (0xc0000000,...)
    df['address'] = df['data'].apply(lambda x: (x.split(':')[0]))
    df['address'] = df['address'].apply(lambda x: hex(int(x, 16)))

    # Make 'address_compare' column for verification
    df['address_compare'] = np.array(range(int_start_address, int_end_address + 0x10, 0x10))
    df['address_compare'] = df['address_compare'].apply(lambda x: hex(x))

    logger.info(f"The First Five Row Data Frame:\n{df[:5]}")

    # verify_bool will be True if all values are True
    verify_bool = (df['address'] == df['address_compare']).all()
    logger.info(f"All Address Verified (True/False): {verify_bool}")

    if verify_bool == False:
        # Prints only unmatched data
        unmatch_df = df[df['address'] != df['address_compare']]
        logger.critical(f"Unmatched ADDRESS DataFrame List:\n{unmatch_df}")

    return verify_bool, df

def savefile(byte_encoded, dest_file):
    with open(dest_file, 'wb') as fh:
        fh.write(byte_encoded)

def get_logger(log_path, log_name):
    """ Configure Logger and Returns Handler """
    logger = logging.getLogger("logger")
    logger.setLevel(logging.INFO)

    formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

    stream_handler = logging.StreamHandler()
    stream_handler.setFormatter(formatter)
    logger.addHandler(stream_handler)

    file_handler = logging.FileHandler(os.path.join(log_path, log_name))
    file_handler.setFormatter(formatter)
    logger.addHandler(file_handler)

    logger.info("Application Start!!!")

    return logger

if __name__ == '__main__':
    main()
```

## Wrap-up
지금까지 데이터사이언스 방법을 적용하여 신속하고 신뢰성 있는 덤프 파일로 변환할 수 있는 방법을 알아보았다.

서두에서 32MByte 플래시 메모리를 기준으로 시리얼 통신을 통해 로그를 획득한 시간은 약 14시간이 소요되고, 이를 덤프 파일로 변환하는 시간은 **약 10~11분** 정도 소요됨을 확인하였다.

가로 정합성, 세로 정합성을 판별하여 검증하고 문제가 없는 경우에 바이너리로 변환하는 시간은 정합성을 판별하지 않을 때에 비해 약 3~4배의 시간인 **30분~40분**이 걸린다.

이에 반해 데이터사이언스를 이용하면 로그 파싱, 정합성 검증, 정제, 바이너리 변환을 수행하는데 걸린 시간은 **약 8.4초**가 소요되었으며 기존과 비교하면 최대 <span style="color:red">**300배 속도 향상**</span>의 결과를 확인할 수 있었다. 그리고 가장 중요한 부분인 <span style="color:red">**신뢰할 수 있는 데이터로 변환**</span>했다는 점이다.

마지막으로, 아무리 데이터사이언스를 사용했다고 하더라도 32MB 이상 되는 플래시 메모리는 로그 획득 시간만 14시간이 넘어가기 때문에 실무적으로는 왠만하면 시리얼 통신으로 획득하는 방법을 사용하지 않고 다른 방법을 통해 획득하려고 한다.

기회가 된다면 획득 시간을 더 빨리 줄일 수 있는 방법에 대해서도 포스팅 하도록 하겠다.


## Reference
- <https://sacko.tistory.com/62>
- <https://www.johncanessa.com/2018/12/24/numpy-vectorization>


## Copyright (CC BY-NC)
본 게시글은 CC BY-NC Licence를 따릅니다.  
비영리 목록으로만 사용할 수 있고, 저작자와 출처를 표시하면 언제든지 게시글을 자유롭게 사용할 수 있습니다.

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
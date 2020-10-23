---
title : "Blog #16: IoT Forensics (Acquisition) part 1"
category :
  - IoT Forensics
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - IoT
  - Embedded Systems
  - Serial Communication
  - UART
  - ipTIME
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
IoT Forensics (획득) part 1


## IoT Forensics Series
- [Blog #16: IoT Forensics (Acquisition) part 1](https://kyl3song.github.io/iot%20forensics/IoT-Forensics-(Acquisition)-part-1/)
- [Blog #17: IoT Forensics (Log Refinement w/ Data Science) part 2](https://kyl3song.github.io/iot%20forensics/data%20science/IoT-Forensics-(Log-Refinement-with-Data-Science)-part-2/)


## Preface
IoT(Internet of Things)는 사물간 인터넷으로 통신을 하며 서로간 정보를 주고 받는 기술을 의미한다. 출근길 도로가 막히면 스마트폰이 알람을 평소보다 30분 더 일찍 울리고, 집안 전등이 일제히 켜지고 커피포트가 때맞춰 물을 끓여주는 것 처럼 우리 생활을 더욱 편리하게 해주는 기술이다.

그런데 요즘에는 공유기 등과 같이 소형 기기에 인터넷만 연결되어 있으면 무조건 IoT라고 불릴정도로 단어의 쓰임이 약간 변질된 것 같다. 이런 기기들을 더 정확하게 표현하자면 임베디드 시스템(Embedded System)이 맞을 듯 하다.

임베디드시스템은 마이크로 프로세서를 기기에 장착하여 설계함으로써 효과적인 제어를 할 수 있도록 하는 시스템으로서 기기를 동작하는 소프트웨어(운영체제)를 컴퓨터처럼 디스크에서 읽어들이는 게 아니라 칩에 담아 기기에 내장시킨(embedded) 형태의 장치를 말한다.

즉, 하드웨어적으로 임베디드시스템은 IoT 기기의 의미를 포함하고 있다.

이번 포스팅 제목을 IoT Forensics라고 붙인 이유는 포스팅을 통해 커버할 내용이 IoT 기기에도 기초가 되는 부분이기 때문이다.

IoT 기기를 획득하는 방식은 다양하다. 기본적으로 시리얼 통신에서 부터 SPI 통신, 그리고 Chip-off 방식까지 다양한 방식으로 획득할 수 있다.

그 중에 이번 포스팅 시리즈에서는 시리얼 통신(UART)을 이용하여 획득하는 방법과 획득된 기기 로그를 정제하는 방법에 대해서 소개하고자 한다.

로그의 정제 과정은 지난 번 [Data Science Series](https://kyl3song.github.io/categories/#data-science) 포스팅 보다 한 단계 업그레이드 된 부분이다.


## RS-232 & UART Serial Communication
흔히 말하는 공유기, 몰래카메라, Drone, AI스피커 등의 디지털포렌식은 대부분 PCB기판 위의 Flash Memory를 획득하여 분석하는 것이다.

물론 기기에 인터넷을 연결하여 클라우드에 저장된 데이터를 다운하여 분석하는 경우도 있겠지만 특수한 경우가 아니라면 인터넷에 연결하는 것은 증거물의 변형을 가한다는 이유로 금기시 되고 있다.

플래시 메모리의 데이터 획득은 가장 기초적인 부분이면서 어려운 부분이기도 하다. 그리고 분석보다 획득에 더 많은 노력이 들어가기도 하는 부분이다.

시리얼 통신을 통해 획득을 하는 경우 비동기 직렬 통신(Asynchronous Serial Communication) 규격 두 가지에 대한 이해가 필요하다.  

비동기 직렬 통신은 동기화를 위한 별도의 신호선이 필요 없으며, 정보의 비트를 직렬화(Serialize) 하여 차례로 송수신하기 때문에 해당 이름이 붙여진 것이다. 기본 개념만 정리하고 넘어가도록 한다.

### RS-232C(Recommended Stanard 232 Revision C)
RS-232 통신 규격은 EIA(Electric Industries Alliance, 미국 전자산업 연합)가 표준화한 양방향 비동기 직렬통신 규격이며 현재 가장 많이 사용하는 버전은 RS-232C이다.

RS-232C 규격은 15m 내외의 근거리에서 사용되는 직렬통신으로 표준에는 9개의 신호선이 정의되어 있으나 기본적으로 3가지 라인만 연결되어 있어도 통신은 가능하다.
- TD(Transmit Data line) : 자신의 입장에서 정보를 전송하는데 사용
- RD(Receive Data line) : 자신의 입장에서 정보를 수신하는데 사용
- SG(Signal Ground) : 전기적 신호의 높낮이를 표현하며 기기의 충전 전류로 인한 기기의 손상 등 방지

<p align="center">
  <img src="https://i.imgur.com/4bwKDhB.png" alt="image"/>
<br>그림 1. RS-232C 통신 포트 핀 맵</p>

비동기 직렬 통신이기 때문에 통신의 시작과 끝을 알리는 동기(Sync)신호가 필요하지 않으며 송수신기간 설정된 전송 속도에 따라 전송되는 데이터프레임(Data Frame) 내부에 Start/Stop bit로 타이밍 신호가 포함되어있다.

0과 1을 표시하는 전압은 최고 ±25V를 이용하여 저전압 저전력 구조로 제품을 설계하는 최근 추세에는 어울리지 않아 많이 사용되지 않는다.


### UART(Universal Asynchronous Receiver & Transmitter)
범용 비동기 송수신기(UART)는 비동기 직렬통신을 위한 송수신 장치를 하나의 칩에 붙여놓은 것으로, RS232 통신과 기본적인 동작 방식은 같으나 0과 1을 표현하기 위하여 TTL(Transistor-Transisitor Logic) 레벨의 전압인 3~5V 전압을 이용한다는 특징이 있다.

UART IC에는 프로세서에게 이벤트를 알릴 수 있는 인터럽트 발생 기능이 내장되어 있어 송수신 완료/패리티에러발생 등의 상태를 알려 오류처리를 할 수 있게 한다.

UART는 PC간 통신, 서로 다른 하드웨어간의 통신, 하드웨어와 센서간 통신, 하드웨어 디버깅 등 다양한 분야에서 사용되며 일반적으로 RS-232, RS-422, RS-485와 같은 통신 표준과 함께 사용한다.


## PCB에서의 UART 단자확인 (How UART Interface placed in PCB)
UART를 통해 획득하기 위해서는 PCB기판에 UART 단자가 있어야 한다. 만일 없는 경우 다른 방법을 통해 획득을 해야한다. UART 단자가 있다면 4개의 단자가 일렬로 붙어있는게 특징이다.

<p align="center">
  <img src="https://i.imgur.com/VjsmKFG.png" alt="image"/>
<br>그림 2. KT 공유기 UART 단자(핀 없음)</p>

UART 단자에 pin이 달려있을 수 있고 달려있지 않을 수도 있다. 단자는 있는데 핀이 달려있지 않다면 그 자리에 납땜을 해야한다.

<p align="center">
  <img src="https://i.imgur.com/dNgOvIb.png" alt="image"/>
<br>그림 3. ipTIME 공유기 UART 단자(핀 있음)</p>

그리고 UART 단자 근처 기판에 <span style="color:red">**TX, RX, 3.3V, GND**</span>라고 쓰여있다면 각 핀이 명확히 식별 가능하지만 그렇지 않은 경우 디지털멀티미터기를 이용하여 각 단자가 무엇인지 확인할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/sQIIqIj.png" alt="image"/>
<br>그림 4. UART 핀 유무</p>


## UART 통신을 이용한 데이터 획득 절차 (Collecting Evidence in a forensically sound manner through UART)
최근 출시되는 노트북 및 PC는 시리얼 통신을 위한 단자가 존재하지 않으므로 USB-to-Serial convertor 케이블을 이용해야 하며 구글에서 검색하면 다양한 케이블이 저렴한 가격에 판매되고 있는 것을 확인할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/GmpPWSP.png" alt="image"/>
<br>그림 5. USB-to-Serial 케이블 종류 (Google 검색)</p>

### 1. UART 단자 연결 (UART Interface Cabling)
케이블을 연결할 때는 PCB의 TX, RX 표기는 핀을 가지고 있는 장치 기준으로 표기하는 경우가 대부분이므로 USB-to-Serial 케이블의 TX, RX는 PC입장에서의 핀이므로 TX, RX를 엇갈리게 연결해야 한다.

<p align="center">
  <img src="https://i.imgur.com/oWdmat1.png" alt="image"/>
<br>그림 6. UART 단자와 케이블 연결</p>

UART 포트와 케이블의 VCC 연결을 해주지 않은 이유는 일반적으로 장치에 어댑터를 통해 전원을 공급해주는 것이 기기동작에 원할하기 때문에 따로 연결을 해주지 않는다.

### 2. 터미널 프로그램 설정 (Terminal Program Configuration)
터미널 프로그램을 선택하는 기준은 시리얼 통신 터미널을 지원하고 대용량 통신 로그를 파일로 저장할 수 있는 기능을 가진 터미널 프로그램을 이용하면 된다.
가장 많이 사용하는 putty, XShell 등 프로그램에서는 모두 다 지원하므로 자신이 편한 것을 선택하여 사용하면 된다.

특히 세션 연결을 하기 전, 자동 로그 생성 기능을 이용하여 터미널에 현출되는 모든 문자열이 로깅되게끔 설정한다.

<p align="center">
  <img src="https://i.imgur.com/0UVS0CP.png" alt="image"/>
<br>그림 7. Xshell의 로깅 설정</p>

시리얼 통신을 연결할 때 보레이트(Baud Rate)를 지정해야 하는데 9600, 38400, 57600, 115200 등 (Data bits-8, Stop Bits-1, Parity-None, Flow Control-None)사이에서 선택하고 연결한 후, 장치에서 전송하는 문자가 정상적으로 출력되는지 확인한다.

기기에 전원을 인가할 때 UART 통신 시 이슈가 발생한다면 다음의 경우를 확인하면 문제를 해결하는데 도움이 될 수 있다.
- 통신 전압 확인(3.3V, 5V 등)
- Tx, Rx 연결 확인
- GND 연결 확인
- Baudrate 설정 값 확인

특히 연결은 되었으나 아래 그림과 같이 문자열이 깨진 것처럼 보인다면 Baudrate 설정이 잘못되었을 확률이 높으므로 Baudrate 설정을 다른 것으로 변경한다.

<p align="center">
  <img src="https://i.imgur.com/39YFHud.png" alt="image"/>
<br>그림 8. UART 통신 시 문자열 깨짐 현상</p>

<p align="center">
  <img src="https://i.imgur.com/4gAzJsp.png" alt="image"/>
<br>그림 9. 문자열이 정상 출력된 예시</p>

### 3. 데이터 획득 (Collecting Flash Memory Dump Log))
기기의 부팅 과정 중 현출되는 부팅 로그를 보면 플래시 메모리 종류와 메모리 주소별 저장 정보를 확인할 수 있는 경우가 많으므로 부팅로그를 유심히 확인하는 것이 좋다.

<p align="center">
  <img src="https://i.imgur.com/rM3VpCz.png" alt="image"/>
<br>그림 10. 부팅 로그 중 메모리의 정보 및 파일시스템 구조 정보</p>

최근에는 보안 이슈가 중요한 문제로 인식되어 커맨드 셸(shell)을 바로 현출하지 않으나 저가형 인터넷 공유기나 오래전 개발된 IoT 기기의 경우 부팅 후 바로 OS의 커맨드 셸을 바로 현출하는 경우가 많다.

> 국내에서 많이 사용되는 ipTIME 공유기의 경우 펌웨어 v9.12까지는 #notenoughmineral^을 입력하고, v9.14 ~ 9.72는 !@dnjsrurelqjrm*&을 입력하면 리눅스 커맨드 셸을 사용할 수 있다.

로그인 프롬프트가 현출되는 기기의 경우 root/root를 입력하거나 기기의 매뉴얼에 표시되어 있는 경우가 종종 있으므로 구글링을 할 필요가 있다.

부팅이 완료된 후 터미널 프로그램에서 엔터 키를 누르면 로그인 프롬프트가 나오거나 셸이 바로 표시된다.

<p align="center">
  <img src="https://i.imgur.com/xygCAWr.png" alt="image"/>
<br>그림 11. Linux Shell</p>

이 상태에서 데이터 전체를 획득을 하는 방법은 다양하다. 네트워크를 이용한다면 netcat을 이용하여 dd로 덤프를 이용해도 되고 tftp를 이용해도 된다. 하지만 실무상 네트워크를 이용하는 방법은 왠만하면 지양하는 편이고 가능한 연결의 수를 최소화 하여 획득한다.

즉, 현재 UART 시리얼 통신이 연결되어 있으니 이 시리얼 통신으로 획득할 수 있는 방법이 있다면 그걸로 획득을 권장한다는 의미이다.

임베디드 리눅스 기기에서는 플래시 메모리를 MTD (Memory Technology Device)로 관리하고 있고 그 정보는 /proc/mtd 파일을 읽으면 부팅로그에서 현출된 정보와 동일한 정보를 확인할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/TkAGWdW.png" alt="image"/>
<br>그림 12. /proc/ 경로 내부</p>


<p align="center">
  <img src="https://i.imgur.com/AuWTchc.png" alt="image"/>
<br>그림 13. /proc/mtd 파일 내용</p>


위 /proc/mtd 파일 내용의 의미는 다음과 같다.

**mtd0 영역**
- 0x400000(4,194,304)bytes의 크기를 가지고 있음
- 0x1000(4,096) bytes 단위로 삭제가 가능함 (플래시메모리)
- Boot Loader, 설정값(cfg), 리눅스 커널이미지(linux), 루트 파일시스템(rootfs)가 저장되어 있음

**mtd1 영역**
- 0x300000(3,145,728)bytes 크기를 가지고 있음
- 0x1000(4,096) bytes 단위로 삭제가 가능함 (플래시메모리)
- rootfs만 저장되어 있음

리눅스 시스템 개발자마다 rootfs를 생성할 때 포함시키는 명령어가 다를 수 있으나 주로 hexdump, xxd, dump 등과 같은 명령어가 포함된다.

본 포스팅에서는 ipTIME 공유기를 이용하였고 dump 명령어가 포함되어 있어 dump를 사용하였다.

dump명령은 플래시 메모리의 내용을 읽어 터미널에 출력해주는 명령어로 플래시 메모리에 해당하는 device 파일을 대상으로 한다. 리눅스는 모든 장치를 파일처럼 취급하므로 /proc/mtd 파일에 해당하는 /dev/mtd 파일이 있는지 확인한 후 dump를 진행하면 된다.

<p align="center">
  <img src="https://i.imgur.com/DsJ6uaj.png" alt="image"/>
<br>그림 14. /dev/ 경로 내부</p>

<p align="center">
  <img src="https://i.imgur.com/PSUEJMx.png" alt="image"/>
<br>그림 15. dump 명령어 사용</p>

4Mbyts(0x400000)의 플래시 메모리를 획득할 경우 아래와 같이 명령어를 입력하면 된다.

``` shell
dump /dev/mtd 0x0 0x400000 
```

플래시 메모리 사이즈 값은 다음 표와 같다.

|MByte 표기 용량          |Byte 표기 용량           |16진수 표기 용량        |
|:----------------------:|:----------------------:|:----------------------:|
|1MBytes             |1,048,576 Bytes     |0x00100000          |
|2MBytes             |2,097,152 Bytes     |0x00200000          |
|4MBytes             |4,194,304 Bytes     |0x00400000          |
|8MBytes             |8,388,608 Bytes     |0x00800000          |
|16MBytes            |16,777,216 Bytes    |0x01000000          |
|32MBytes            |33,554,432 Bytes    |0x02000000          |
|64MBytes            |67,108,864 Bytes    |0x04000000          |
|128MBytes           |134,217,728 Bytes   |0x08000000          |
|256MBytes           |268,435,456 Bytes   |0x10000000          |


명령어를 입력하면 플래시 메모리 4Mbytes 크기의 16진수 정보가 ASCII 형태로 약 2~3시간에 걸쳐 시리얼 터미널을 통해 PC로 전송되어 화면에 현출하게 된다.

현출이 종료되면 터미널 프로그램(Xshell)에서 자동 로깅을 설정한 부분 때문에 현출된 로그가 자동으로 파일로 저장된다.

<p align="center">
  <img src="https://i.imgur.com/FSFHcTv.png" alt="image"/>
<br>그림 16. 획득된 ipTIME 공유기 로그</p>

저장된 로그에서 기기 덤프 시작부분과 끝부분을 잘라내도 되고 아니면 아예 애초부터 덤프 명령어를 치기 전 로깅을 잠시 끊었다가 덤프 명령어의 시작과 함께 로깅을 해도 상관없다.

### 4. 덤프 로그 정제 작업 (Refine & Convert Dump Log to Binary File)
현재 저장된 로그 파일을 이용하여 ASCII로 전송된 16진수 정보를 Binary 형태로 변환하면 사본 획득이 마무리된다.

변환은 로그 파일에서 프로그래밍을 통해 현재 주소값을 제외하고 순수 원본의 정보에 해당하는 값만 정제하고 바이너리 파일로 변환해야 한다.

``` python
# -*- coding: utf-8 -*-

import datetime
import os
from tqdm import tqdm

fname = 'iptime_N8004R_dump.log'
dst_name = 'iptime_N8004R_dump.bin'
 
start_time = datetime.datetime.now()

with open(fname, mode='r', encoding='utf-8') as fh:
	data_list = fh.readlines()
	for data in tqdm(data_list):
		data = data.split(' ')[1:]
		data = ''.join(data).replace('\n', '')
		bdata = bytes.fromhex(data)

		with open(dst_name, mode='ab') as fh_write:
			fh_write.write(bdata)

end_time = datetime.datetime.now()

print(f"Dump Log File Size: {os.path.getsize(fname)} Bytes")
print("Start Time:", start_time)
print("End Time:", end_time)
print("Elapsed Time:", end_time - start_time)
```

<p align="center">
  <img src="https://i.imgur.com/JTlT4xB.png" alt="image"/>
<br>그림 17. Python Script 실행 결과</p>

플래시 메모리의 용량이 4MB가 밖에 되지 않아 코드의 실행은 약 2분 이내로 끝나고 정상적으로 바이너리 파일로 변환이 완료되었다.

<p align="center">
  <img src="https://i.imgur.com/Zg9mEPD.png" alt="image"/>
<br>그림 18. 변환된 Binary 파일</p>


## UART 통신을 통한 메모리 획득 방법의 단점 (Disadvantages of Data Collecting through UART)
UART 통신으로 화면에 현출된 로그를 이용하여 획득하는 방법은 2가지의 단점이 있다.

1. 시리얼 통신에 따른 데이터 전송 속도가 느리다.  
\- 시리얼 통신의 근본적 문제점
2. 화면에 현출되는 데이터가 부정확할 수 있다.  
\- dump 등 명령어로 모니터에 플래시메모리의 정보를 현출할 때의 문제점

1번은 SPI 통신이나 Chip-off 방법으로 해결할 수 있고, 2번은 다음 포스팅에 구체적으로 데이터의 부정확성(inconsistency)에 대해 살펴보고 해결방안에 대해 알아볼 예정이다.


## Wrap-up
지금까지 UART 시리얼 통신을 통해 소형 기기의 데이터를 획득하는 방법과 이를 실제 분석에 필요한 바이너리 파일로 변환하는 방법에 대해 살펴봤다.

사실 시리얼 통신이라도 획득할 수 있는 방법은 다양하다. 예전 증거 분석을 한 것 중 VoIP Gateway는 Serial Console 포트가 아예 장비 밖으로 나와있고 부트로더(U-boot)의 md 명령어를 사용하여 획득이 바로 가능한 경우도 있었다. 

또한 아두이노 등 장비를 이용하여 플래시메모리칩을 납땜한 뒤 프로그래밍으로 획득할 수 있는 방법도 있다. 따라서 그때 그때 상황에 맞게 획득을 진행하면 된다.

만일 기기를 파손해서는 안되는 상황이고 네트워크 연결이 크게 문제되지 않는 경우에는 포트 스캔, 취약점 검색을 통해 취약한 버전의 어플리케이션을 매개로 삼아 권한상승(Previlege Escalation)을 통해 루트셸을 획득하는 방법도 있으니 이럴때를 대비하여 포렌식 지식뿐 아니라 모의해킹, 프로그래밍, 데이터분석 등 다양한 분야에 관심을 넓게 두는 것이 필요하다.


## Reference
- <https://www.link-labs.com/35-top-iot-terms-you-need-to-know>
- <https://terms.naver.com/entry.nhn?docId=3577301&cid=59088&categoryId=59096>
- <https://m.blog.naver.com/PostView.nhn?blogId=seo0511&logNo=10132695322&referrerCode=0&searchKeyword=%EC%8B%9C%EB%A6%AC%EC%96%BC%20%ED%86%B5%EC%8B%A0>
- <https://bigonemoon.tistory.com/325>
- <https://en.wikipedia.org/wiki/Netcat>
- <https://ko.wikipedia.org/wiki/UART>
- <https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter>
- <https://www.hackerschool.org/HardwareHacking/UART%20%ed%95%b4%ed%82%b9%20-%20Essential.pdf>
- <https://whatis.techtarget.com/definition/UART-Universal-Asynchronous-Receiver-Transmitter>
- <http://www.ktword.co.kr/abbr_view.php?m_temp1=2304&m_search=uart>
- <https://m.blog.naver.com/PostView.nhn?blogId=msechung2k&logNo=220902639639&proxyReferer=https:%2F%2Fwww.google.com%2F>

## Copyright (CC BY-NC)
본 게시글은 CC BY-NC Licence를 따릅니다.  
비영리 목록으로만 사용할 수 있고, 저작자와 출처를 표시하면 언제든지 게시글을 자유롭게 사용할 수 있습니다.

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
---
title : "Blog #13: Leveraging WinFE(Forensic Environment) part 2"
category :
  - Portable OS
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - WinPE
  - WinFE
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
WinFE 활용 방법 part 2 - The Final

## Leveraging WinFE(Forensic Environment) Series
- [Blog #12: Leveraging WinFE(Forensic Environment) part 1](https://kyl3song.github.io/portable%20os/Leveraging-WinFE(Forensic-Environment)-part-1/)
- [Blog #13: Leveraging WinFE(Forensic Environment) part 2](https://kyl3song.github.io/portable%20os/Leveraging-WinFE(Forensic-Environment)-part-2/)

지난 포스트에서는 WinPE를 이용하여 증거물을 획득할 때 발생할 수 있는 이슈사항을 확인하였고, 실제 테스트를 진행하면서 분석 대상 원본 매체에 의도치 않게 쓰기 작업이 발생한 것을 확인하였다.

이번 포스트에는 WinFE에 대해 알아본다.

## WinFE (Windows Forensic Environment)
WinFE는 원래 Microsoft 수석 포렌식 매니저인 Troy Larson이 Windows Vista 사전 설치 환경 2.0 (WinPE 2.0)에 두 개의 레지스트리 키를 추가하여 개발하였는데, 이 레지스트리 키는 부팅 시 볼륨의 자동 마운트를 방지하고 있다.

<p align="center">
  <img src="https://i.imgur.com/gf1ZlvD.png" alt="image"/>
</p>

현재는 BIOS, UEFI 환경에서도 동작할 수 있게 프레임 워크로 개발되었다. Windows, macOS, Linux 운영체제에서 동작이 가능하나 운영체제에 대한 테스트는 계속 진행되고 있다.

<p align="center">
  <img src="https://i.imgur.com/yFcSu0P.png" alt="image"/>
</p>

## 포렌식 관점에서의 WinFE
WinFE는 기본적으로 자동으로 볼륨이 마운트 되는 것을 막아주어 쓰기방지 역할을 한다. 프레임 워크 내 기본적으로 이미징 도구가 탑재되어 있고, 파일 탐색할 수 있는 탐색기 등 여러 도구가 탑재되어 있다.

활용해본 입장에서 포렌식 용도에 맞게 프레임워크가 개발된 것 같은 느낌을 준다. 그리고 WinPE와 같이 자신이 원하는 도구들은 개별적으로 넣을 수 있고, 운영제체의 배경화면도 변경이 가능하도록 빌드가 가능하다.

## How to build WinFE
빌드하는 방법은 [WinFE 홈페이지](https://www.winfe.net/build)에서 확인할 수 있는데 가이드대로 그대로 따라가기만 하면 된다. 
홈페이지에의 스테이지 순서대로만 하면 되고 짧은 설명과 함께 스크린 샷만 남기도록 하겠다.

### Build Stage 1 (사전 준비 및 WinFE Framework 다운로드)
> 아래 기재한 내용에서 경로가 변경되었을 수 있으니 반드시 공식 홈페이지를 참고 바란다.

1. [7-Zip](https://www.7-zip.org) 다운로드 (반디집도 가능함)
2. [Intel x86/x64 framework](https://www.winfe.net/files/IntelWinFE.7z) 다운로드 후 압축 해제

<p align="center">
  <img src="https://i.imgur.com/cPHCcLi.png" alt="image"/>
</p>

3. (Optional) WinFE의 배경화면을 변경하려면 원하는 배경 파일을 **IntelWinFE\x64\wallpaper.jpg** 그리고 **IntelWinFE\x86\wallpaper.jpg**로 넣는다.

<p align="center">
  <img src="https://i.imgur.com/3KDjaab.png" alt="image"/>
</p>

### Build Stage 2 (ADK 다운로드)
다른 ADK(Windows Assessment and Deployment Kit) 버전에서도 동작할 수 있으나 공식 홈페이지에서는 WinFE 테스트는 Windows 10 ADK, version 1803에서 되었다고 명시되어 있다. ADK를 다운로드하여 설치 한다.

ADK 1803 Direct Link: https://go.microsoft.com/fwlink/?linkid=873065

### Build Stage 3 (FTK Imager 도구 다운로드)
FTK Imager를 다운로드하는 것은 선택사항이다. FTK Imager 도구를 넣어서 빌드 하려면 필요하지만 굳이 그렇게 하지 않아도 된다. 이유는 도구 획득 테스트(Acquisition Test with FTK Imager, X-Ways Forensics) 부분에서 다시 설명할 예정이다.

홈페이지의 설명의 요지는 이렇다.

1. 빌드에 포함하기 위한 x86용 도구 준비 (ex. FTK Imager 3.4.0.1)  
32비트용 [FTK Imager 3.4.0.1](https://accessdata.com/product-download#past-versions) 도구를 다운로드해 자신의 PC에 설치한 뒤, 설치된 경로(통상적으로 C:\Program Files(x86)\AccessData\)에 가서 FTK Imager 폴더를 통째로 WinFE 빌드 전 도구 경로(IntelWinFE\USB\x86-x64\tools\x86)에다 붙여넣기 수행

2. 빌드에 포함하기 위한 x64용 도구 준비 (ex. FTK Imager 4.2.0)  
64비트용 [FTK Imager 4.2.0](https://accessdata.com/product-download#past-versions) 역시 마찬가지로 설치 > FTK Imager 폴더 복사 > WinFE 빌드 전 도구 경로(IntelWinFE\USB\x86-x64\tools\x64)에 붙여넣기 수행

### Build Stage 4 (WinFE 빌드)
명령 프롬프트(CMD)를 통해 Stage 1의 압축이 풀린 IntelWinFE 폴더로 이동하여 MakeWinFEx64-x86.bat 명령어를 입력하여 WinFE를 빌드 한다.

<p align="center">
  <img src="https://i.imgur.com/AGBDiRZ.png" alt="image"/>
</p>

그리고 어느 정도 기다리면 배치파일에 의해 자동으로 명령어도 입력하고 복사도 하는 빌드 과정을 거치게 된다.
<p align="center">
  <img src="https://i.imgur.com/3ESwS26.png" alt="image"/>
</p>

### Build Stage 5 (WinFE iso 만들기)
부팅 USB를 만들려면 WinFE bootable iso 파일이 필요하다. Stage 4에서 열어놨던 명령 프롬프트를 그대로 둔 상태에서 이번에는 Makex64-x86-CD.bat 명령어를 입력한다.
그럼 이번에는 iso 파일을 만드는 작업이 자동으로 진행된다.

<p align="center">
  <img src="https://i.imgur.com/zH9OpjX.png" alt="image"/>
</p>

작업이 완료되면 IntelWinFE\ISO\WINFE_10x86-x64.iso로 WinFE ISO 파일이 생성된다.

<p align="center">
  <img src="https://i.imgur.com/5tB0Jjd.png" alt="image"/>
</p>


### Build Stage 6 (Bootable USB Flash Drive 만들기)
여기부터는 해당 iso 파일을 가지고 부팅 USB를 만드는 부분이기 때문에 WinFE 공식 홈페이지 대로 하지 않고, [Rufus](https://rufus.ie)라는 프로그램을 이용하여 부팅 USB를 만들었다.

<p align="center">
  <img src="https://i.imgur.com/yUphN1q.png" alt="image"/>
</p>

## Getting into WinFE
### 부팅 및 초기 화면
만들어진 WinFE USB를 삽입 후 컴퓨터를 부팅을 하면 x86, x64 아키텍처의 환경을 골라서 부팅할 수 있다.

분석 대상물의 환경이 다를 수 있다는 상황을 미리 고민하여 이를 프레임워크에 반영한 WinFE의 초기 부팅 모습은 인상적이었다. 골라서 선택할 수 있는 것은 분명 장점이다.

<p align="center">
  <img src="https://i.imgur.com/ZWTeIWa.png" alt="image"/>
</p>

최초 부팅 시 언어 선택을 할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/tKP5XNI.png" alt="image"/>
</p>

부팅이 완료되기 전 경고 메시지가 현출되는데 내용의 의미는 Diskpart, Device Manager, Disk Manager 도구에 의해 쓰기방지 역할을 못할 수 있다는 의미이다.

<p align="center">
  <img src="https://i.imgur.com/HF4r34c.png" alt="image"/>
</p>

진정한 쓰기방지는 커널 모드 필터 드라이버를 이용하여 쓰기방기 조치가 되야 하나, WinFE의 쓰기방지는 레지스트리를 수정한 방식이므로 현재 경고 메시지에 기재된 도구를 이용하여 디스크를 수정하거나 변조하는 작업을 하지 않으면 큰 문제는 없다.

이 페이지가 넘어가면 Write Protect Tool이 바로 실행되는데 아래서 다시 다루기로 하고 해당 도구를 종료하면 WinFE의 바탕화면이 확인된다.
만일 빌드 과정에서 배경화면을 변경했다면 아래 화면과 같이 배경이 변경된다.

<p align="center">
  <img src="https://i.imgur.com/spO2Puj.png" alt="image"/>
</p>

## WinFE Built-in Tools
### Disk Tools > Write Protect Tool

<p align="center">
  <img src="https://i.imgur.com/5Igli0v.png" alt="image"/>
</p>

WinFE의 여러 도구 중 쓰기방지 도구이다. 기본적으로 자신의 운영체제를 제외한 나머지 디스크 드라이브는 마운트가 되지 않는다. USB를 삽입해도 최초는 마운트가 되지 않은 상태에서 목록에만 보여준다.

부팅을 한 상태에서 가장 처음 나오는 툴이기도 하다. 옵션에는 Read/Write, Mount 등을 선택할 수 있는 옵션이 있다.

<p align="center">
  <img src="https://i.imgur.com/8rLdq2T.png" alt="image"/>
</p>

### Disk Tools > Basic Disk Imager

WinFE에 내장되어 있는 도구 중 이미징을 할 수 있는 도구가 있다. 해당 도구로 획득 시 .img 파일로 획득이 되며 포맷은 따로 변경할 수 있는 옵션이 없다.

<p align="center">
  <img src="https://i.imgur.com/yyoq8Pm.png" alt="image"/>
</p>

이미징 테스트를 수행했을 시 정상적으로 획득이 되었다.(해시값 검증)


### Password Tools

<p align="center">
  <img src="https://i.imgur.com/MUz7ylI.png" alt="image"/>
</p>

이 도구는 패스워드를 없애주는 역할을 하는 도구이다. 상황에 따라 라이브 상태로 분석을 해야 되거나, 사본의 디스크로 부팅해서 분석을 해야 될 경우 유용할 것으로 생각된다.  
기본적으로 이 도구를 사용하려면 Read/Write 모드로 설정되어 있어야 한다.

**Standard Reset**은 윈도우 10, Server까지 로컬 계정 로그인 암호를 제거할 수 있는 도구이다. 직접 사용해 보지는 않았으나 나중에 암호를 복원할 수 있는 기능도 제공한다고 되어 있다.

**Advanced Reset**은 AD(Active Directory) 도메인 환경에서 사용 중인 시스템의 암호를 제거할 수 있는 도구이다.

Password Tool는 모두 별도 구매를 해야만 사용이 가능한데 정부(Governement) 및 법 집행기관(LE(Law Enforcement)은 무료로 사용할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/qpb4yj4.png" alt="image"/>
</p>

### Other Tools
기타 도구로는 사용자가 필요할 만한 다양한 도구들이 있다.

<p align="center">
  <img src="https://i.imgur.com/MuSHzGa.png" alt="image"/>
</p>

1. Command Prompt: 커맨드 창  

<p align="center">
  <img src="https://i.imgur.com/JC6dNcd.png" alt="image"/>
</p>

2. Configure Network: 네트워크 설정, 공유 폴더, Ping, Debug Mode 등 다양한 설정이 가능하다.  

<p align="center">
  <img src="https://i.imgur.com/4QeyF8Z.png" alt="image"/>
</p>

3. File Explorer: 폴더/파일 탐색기  

<p align="center">
  <img src="https://i.imgur.com/qrjOUya.png" alt="image"/>
</p>

4. Install Driver: 드라이버(inf 파일) 설치 가능 도구  

5. Notepad: 메모장  

6. Registry Editor: 레지스트리 편집기  

<p align="center">
  <img src="https://i.imgur.com/cHksAC9.png" alt="image"/>
</p>


## Acquisition Test (With FTK Imager, X-Ways Forensics)
이번에는 Built-in 획득 도구 말고, FTK Imager나 X-Ways Forensics 도구를 통해 획득을 진행해보자. 프로그램이 정상적으로 실행이 되는지도 보면서 말이다.

Write Protect Tool 목록에서 **Disk 0**은 노트북의 HDD이고, Disk 1은 WinFE Booting USB이다.
테스트에서는 HDD(source)를 증거분석 대상으로 가정하고 획득해야 하기 때문에 복제 이미지를 담을 외장 HDD(destination)가 필요하다.

250GB의 외장 HDD(**Disk 2**)를 연결한 뒤, Read/Write 및 Mount 버튼을 누르면 자동 마운트 된다. 그럼 복제 이미지를 담을 준비가 된 것이다.

<p align="center">
  <img src="https://i.imgur.com/g3Vk9U3.png" alt="image"/>
</p>


### Acquisition with non-prebuilt Tools
사본 이미지를 담을 외장 하드에 획득 도구를 넣어두면 바로 사용이 가능하다. 즉, 아까 위에서 설명했던 것과 같이 굳이 WinFE 빌드할 때 도구를 포함해서 넣지 않아도 된다는 의미이다.

X-Ways Forensics, FTK Imager 등 도구는 평소에 사용하는 PC에 설치한 뒤, 설치된 경로로 이동하여 그 폴더를 통째로 복사해서 외장 하드에 넣어두면 사용할 수 있다.

다음은 도구 2개를 실행한 결과이다. 모두 정상적으로 실행이 되며 노트북 HDD를 대상으로 획득을 테스트 한 결과, 노트북 HDD를 정상적으로 획득할 수 있었다.

<p align="center">
  <img src="https://i.imgur.com/s7Ar0uj.png" alt="image"/>
</p>

<p align="center"><b>[ FKT Imager 4.3.0.18 ]</b></p>

<p align="center">
  <img src="https://i.imgur.com/xAtnhGQ.png" alt="image"/>
</p>

<p align="center"><b>[ X-Ways Forensics 19.8 ]</b></p>


## How to Unlock Bitlocker Encrypted Volume
WinFE에는 BitLocker로 암호화된 디스크도 복호화 할 수 있는 기능이 있다. 리커버리 패스워드와 사용자 패스워드를 이용한 2가지 방법으로 복호화 할 수 있다. 
사용하기 전 암호화된 디스크를 Read 모드로 마운트를 해야 하며, 도구 사용법은 다음과 같다.

**Recovery Password 사용 시**
``` shell
> manage-bde.exe -unlock <Volume Letter>: -recoverypassword 123456-123456-123456-123456-123456-123456-123456-123456 <Enter>
```

**User Password 사용 시**
```shell
> manage-bde.exe -unlock <Volume Letter>: -password <Enter>
```

실제로 도구를 이용하여 비트락커로 암호화 된 또 다른 외장하드를 정상적으로 해제 및 마운트 할 수 있었다.

<p align="center">
  <img src="https://i.imgur.com/ZX79Ac7.png" alt="image"/>
</p>

테스트하면서 한 가지 헷갈렸던 점은 **-password** 파라미터 뒤에 비밀번호를 넣는 줄 알고 넣었더니 계속 안되고 실패했던 점이다.

알고 보니 엔터를 눌러야 그제서야 비밀번호를 입력하는 부분이 나와서 당황을 했다.


## Wrap-up
이번 2개의 포스트로 WinPE와 WinFE의 차이점에 대해 알아보았다.

WinFE 운영체제는 WinPE에 대비해서 조금 더 손쉽게 라이브 환경을 탐색할 수 있고 업무상 매체 획득을 할 때 조금 더 안전한 방법을 제공한다.

포렌식 도구는 굳이 WinFE 내에 포함하여 빌드 하지 않아도 된다. 획득할 이미지를 담을 수 있는 외장 하드에 도구들을 저장하고 WinFE에서 해당 도구를 실행하는데 전혀 지장이 없어 도구의 버전 업데이트에 대해 유연하게 대처할 수 있다.

WinPE를 사용했을 때 원본 매체에 데이터를 자동으로 쓰는 부분은 사실 미미하다. 하지만 분명한 점은 원본에 변형을 가한다는 점이고 해당 부분 때문에 불필요한 법적 논란이 발생할 수 있어 이를 미연에 방지하고, 쓰기방지 장치의 중요성 인식 제고를 위해 본 포스트를 작성하였다.


## Reference
- <https://www.winfe.net/>
- <https://www.7-zip.org>
- <https://www.winfe.net/files/IntelWinFE.7z>
- <https://go.microsoft.com/fwlink/?linkid=873065>
- <https://accessdata.com/product-download#past-versions>
- <https://rufus.ie/>


## Copyright (CC BY-NC 2.0 KR)
본 게시글은 CC BY-NC 2.0 KR Licence를 따릅니다.
비영리 목록으로만 사용할 수 있고, 저작자와 출처를 표시하면 언제든지 게시글을 자유롭게 사용할 수 있습니다.

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
---
title : "Blog #32: Building a Forensic Environment with WSL & Chocolatey part 2. [KR]"
category :
  - Software & Tools
tag : 
  - Chocolatey
  - WSL
  - Windows Terminal
  - VM Environment

sidebar_main : true
author_profile : true
use_math : true
toc: true
toc_sticky: true
toc_label: "Table of Contents"
header:
  overlay_image : /assets/images/post.jpg
  overlay_filter: 0.5
published : true
---
생산성 향상 및 호환성 이슈 해결 팁


## This Post Covers
WSL을 사용하다 보면 시간이 지날수록 폰트나 화면 분할 등 세세한 기능을 원하게 된다. 보통 우분투의 기본 터미널에서 단축키로 화면을 분할하여 사용하는 것보다 Tmux, Terminator 등 터미널 프로그램을 사용할 경우 생산성을 더 높일 수 있다.
WSL에 있어 베스트 단짝은 **Windows Terminal**일 것이다. 마이크로소프트가 어쩐 일로 이렇게 좋은 프로그램을 만든 건지 의아해할 수 있다. 그리고 WSL보다 일단 Windows Terminal에 더 관심이 가는 비극적인(?) 상황이 벌어지기도 한다.

이번 글에서는 WSL와 Windows Terminal의 조합 팁, 그리고 터미널을 사용하다 보면 문자열이 겹치는 버그를 경험할 수도 있는데 그것을 간단히 해결하는 팁을 소개하고자 한다. 마지막으로 Chocolatey를 통해 테스트 환경을 좀 더 손쉽게 구축하는 방법에 대해 알아보고자 한다.

## Windows Terminal
Windows Terminal 프로그램은 Microsoft Store에서 다운로드할 수 있다. 현재 오픈소스로 공개되어 있으며 [GitHub](https://github.com/microsoft/terminal)에서 코드를 확인할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/Fz8fIG3.png" alt="image"/>
</p>

해당 프로그램의 리뷰를 보면 우리 마소가 달라졌다는 등 대부분 좋은 평가를 하고 있다. 그런데 그도 그럴 것이 UI도 그렇고 탭 형태로 PowerShell, CMD를 포함한 멀티 터미널을 지원하다 보니 사용성 측면에서 훨씬 편하고 테마를 따로 설정하지 않아도 폰트, 색상도 나름 쓸만하기 때문이다. 아래 그림은 터미널을 사용하기 전후를 비교한 것이다.

<p align="center">
  <img src="https://i.imgur.com/uELmlMc.png" alt="image"/>
</p>

예전 Preview 버전일 때는 터미널이 열리는 단축키, 터미널을 처음 실행했을 때 실행되는 Shell이 무엇인지 json 설정 파일을 직접 수정해야만 했다. 그러나 지금은 설정의 옵션을 통해 손쉽게 설정할 수 있다.

### Tip 1. Change Default Profile and Pin WT to Taskbar

초기 설정으로 기본 프로필은 PowerShell로 되어있다. 즉 터미널이 열리면서 파워셸이 바로 실행되는데 이를 설치된 Ubuntu로 변경하면 Ubuntu 환경으로 바로 실행할 수 있다.

> 현재 기준 WSL을 설치하면 자동 설치되는 우분투 버전은 Ubuntu 20.04 LTS이다.

<p align="center">
  <img src="https://i.imgur.com/YljuByO.png" alt="image"/>
</p>

추가로 윈도우의 작업 표시줄에 고정된 프로그램을 실행하기 위해서는 단축키를 사용할 수 있다. 각 아이콘의 순서대로 ***<span style="color:red">Windows Key + 1</span>*** 등 순서로 실행할 수 있는데 가장 앞으로 배치하여 단축키 하나만으로 우분투를 실행할 수 있다.

<p align="center">
  <img src="https://i.imgur.com/xc7Biub.png" alt="image"/>
</p>


### Tip 2. Keyboard Shortcuts in WT

Windows Terminal에서 다양한 단축키가 있다. 그중 많이 사용되는 단축키는 다음과 같다.

- <kbd>Ctrl</kbd> + <kbd>+</kbd> : 폰트 크기 크게하기
- <kbd>Ctrl</kbd> + <kbd>-</kbd> : 폰트 크기 작게하기
- <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>T</kbd> : 새로운 터미널 세션 열기
- <kbd>Alt</kbd> + <kbd>Shift</kbd> + <kbd>D</kbd> : 화면 분할
- <kbd>Alt</kbd> + <kbd>Shift</kbd> + <kbd>↑ or ↓ or → or ←</kbd> : 화면 분할 시 현재 터미널 창 크기 조절
- <kbd>Alt</kbd> + <kbd>↑ or ↓ or → or ←</kbd> : 화면 분할 시 터미널 창 포커스 이동


<p align="center">
  <img src="https://i.imgur.com/Pj35cIA.png" alt="image"/>
</p>

추가로 단축키를 커스터마이징 하거나 단축키 변경을 하기 위해서는 여기 [blog](https://allthings.how/how-to-use-windows-terminal-keyboard-shortcuts/)를 참고하길 바란다.


### Tip 3. Character Cluttered Alignment Bug (aka. Korean IME Bug)

WSL을 사용하다 보면 글자가 겹치는 현상이 자주 발생할 수 있다. 앱 리뷰에서도 누군가 작성해놓은 동일한 현상이다.

<p align="center">
  <img src="https://i.imgur.com/h62hdCn.png" alt="image"/>
</p>

이 현상은 리눅스를 구동하여 사용하거나 파워셸, CMD 등 Shell 사용 시 발생된다. Windows Terminal을 사용하면서 발생되는 문제로 맨 앞에 커서가 있는 채로 영문이든 한글이든 글자가 작성되는 현상이다. 마치 커서는 그대로 있고 글자가 하나씩 순차적으로 밀리는 것 같은 현상이 발생된다. 겉으로 좋지 않게 보이는 것뿐만 아니라 특히 sudo 비밀번호를 입력할 때, ssh 연결할 때와 같이 비밀번호를 입력할 때 비밀번호가 그대로 노출되는 현상이 현재 가장 큰 이슈이다.

<p align="center">
  <img src="https://i.imgur.com/01kqZjY.png" alt="image"/>
</p>

원인은 우리가 일반적으로 사용하는 한컴오피스에 있다. 한컴오피스가 설치되면서 한컴 입력기(IME)가 추가로 설치되는 데, 한컴 입력기가 선택된 상태로 터미널을 사용하면 이와 같은 현상이 발생되니 Microsoft IME를 선택해서 사용하거나, 한컴 입력기를 삭제할 것을 추천한다. 아직까지 한컴 입력기를 삭제한 뒤 한글 앱을 사용하거나 컴퓨터를 사용하는데 이슈가 발생되진 않았다. 삭제는 아래와 같이 가능하다.

- 언어 및 키보드 옵션 편집 ➜ 기본 설정 언어(한국어:옵션) ➜ 키보드: 한컴 입력기 제거


## Chocolatey
Linux의 apt(Advanced Packaging Tool), macOS의 Homebrew와 같이 명령어 한 줄로 패키지를 설치할 수 있다. [Chocolatey](https://chocolatey.org/)는 이와 비슷한 형태로 윈도우의 패키지 매니저이다. PowerShell 또는 CMD를 이용하여 프로그램을 설치할 수 있다. 패키지 검색은 [Community Packages](https://community.chocolatey.org/packages)에서 검색할 수 있고 GUI 버전의 Chocolatey에서는 패키지 검색, 설치, 업데이트 등 작업을 수행할 수 있으니 사용해보는 것도 좋을 것 같다.

개인적으로 Chocolatey는 윈도우에서 리눅스 도구들을 사용할 때, 귀찮은 테스트 환경 구성 시 자동으로 필요한 도구가 설치된 환경 구성을 하고 싶을 때 사용한다.


## How to use
Chocolatey 설치는 관리자 권한의 PowerShell 명령어로 설치할 수 있다. 스크립트의 가장 첫 구문은 PowerShell의 악성 스크립트의 실행을 방지하기 위해 설정된 기본 실행 정책(Restricted)을 Bypass로 변경하는 구문이다.

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

패키지 설치는 파라미터 -y를 추가하면 설치 여부를 한 번 더 묻지 않고 진행하게 된다. 그리고 리눅스처럼 패키지를 체인처럼 엮어서 여러개를 한 번에 설치할 수도 있다.

```powershell
Usage: choco install -y [package_name_1] [package_name_2]...
```

### Recommended Package
패키지를 몇 개 검색하면 알겠지만 생각보다 엄청나게 많은 프로그램들이 레포에 업로드 되어있다. 그중 리눅스용 도구 모음 패키지명은 **UnxUtils**이다. 환경 구성에 필요한 패키지들과 함께 설치하면 편하다.

- sysinsternals
- unxutils
- 7zip
- hashmyfiles
- vscode

```powershell
choco install -y sysinternals 7zip hashmyfiles unxutils vscode
```

<p align="center">
  <img src="https://i.imgur.com/87NJPLG.png" alt="image"/>
</p>


### Binary Location & Debug Logs

패키지는 모두 **C:\ProgramData\chocolatey** 경로에 설치되고, exe 파일로 포팅 된 파일들은 **C:\ProgramData\chocolatey\bin** 폴더에 모여진다. 시스템의 어느 경로에서도 실행이 가능하기 때문에 환경 변수는 자동으로 잡혀져 있다.

<p align="center">
  <img src="https://i.imgur.com/6gXuONq.png" alt="image"/>
</p>

패키지를 설치하면서 생성된 디버그 로그를 통해 설치 당시 풀 커맨드와 어떤 패키지를 설치했는지도 확인할 수 있으므로 **Command line:** 키워드로 일치되는 것만 찾을 수 있다.

- **Log Path: C:\ProgramData\chocolatey\logs\chocolatey.log**

<p align="center">
  <img src="https://i.imgur.com/4O9O8Nw.png" alt="image"/>
</p>



## Wrap-up
지금까지 WSL(+Windows Termianl)과 Chocolatey를 활용하여 메인 분석 환경이나 테스트 환경을 적절히 구성할 수 있는 팁을 살펴보았다. 특히 초기 환경 구성 시 사이트마다 돌아다니면서 다운로드할 필요가 줄어들 것이다. 필요한 프로그램을 리스트업 하고, 이를 설치하는 명령어 한 줄로 불필요한 작업을 줄여보고 생산성을 높여보는 것도 좋을 것 같다.


## Reference
- <https://github.com/microsoft/terminal>
- <https://chocolatey.org>
- <https://community.chocolatey.org/packages>


## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

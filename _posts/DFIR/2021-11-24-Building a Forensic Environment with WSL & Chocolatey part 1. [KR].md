---
title : "Blog #31: Building a Forensic Environment with WSL & Chocolatey part 1. [KR]"
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
손쉬운 포렌식 환경 구축 팁


## This Post Covers
악성코드, 파일시스템 구조 분석 등 다양한 분석을 하거나 테스트가 필요할 때 보통 가상 환경을 이용한다. 가상 머신을 이용하는 주 이유는 스냅샷 기능을 통해 몇 번이고 다시 되돌릴 수 있고 수많은 다운로드 된 파일 실행 중 혹시나 모를 악성코드 감염에서 보호해 주기 때문이다.

가상 환경을 처음 구성할 때 여러 가지 자신이 필요한 도구를 설치하게 되는데 이 과정에서 각 프로그램별 홈페이지를 접속하여 다운받아 설치를 진행해야 한다. 그리고 프로그램이 어느정도 설치가 되면 나의 경우 **init**이라는 문자열로 초기 스냅샷을 만들어 놓는다. 리눅스 운영체제의 경우 패키지 매니저를 통해 설치를 하여 그나마 낫지만 스냅샷을 만들어 놓는 것은 비슷하다.

이런 작업들은 반드시 필요한 작업이지만 귀찮은 작업임에는 틀림없다. 특히 윈도우가 설치된 분석 시스템에서 리눅스 도구들을 사용해야 될 때가 많은데 만일 VM을 사용하는 경우 각 VM마다 리소스도 신경 쓰이기 마련이다. 이번 포스트 시리즈에서는 윈도우 PC에서 WSL과 Chocolatey를 이용하여 분석 환경을 구성하고 손쉽게 리눅스 명령어를 사용하는 방법과 활용법에 대해 알아보도록 한다.

## WSL(Windows Subsystem for Linux)
WSL은 Windows에서 동작하는 Linux용 하위 시스템으로 VMWare에 설치된 Linux처럼 별도의 OS 부팅을 하지 않고 커맨드 명령줄로 리눅스 명령어, 유틸리티 등을 윈도우에서 실행할 수 있다. 한 번 써보면 VMWare 사용량이 확 줄어들 정도로 잘 구현해놨다.

예전엔 WSL을 설치하려면 Windows 기능 켜기/끄기에서 **Linux용 Windows 하위 시스템**, **가상머신 플랫폼**을 선택하여 설치하고 Microsoft Store에서 Linux Distro를 다운로드해 사용할 수 있었는데, 현재 WSL을 사용하기 위해서는 Powershell 또는 CMD에서 간단히 명령줄 하나로 설치가 가능하다. 아예 Ubuntu로 OS를 설치하여 사용할 수 있게까지 해준다.

```powershell
wsl --install
```

<p align="center">
  <img src="https://i.imgur.com/bZnmtT5.png" alt="image"/>
</p>


### WSL1 Architecture
두 개의 버전을 이해하기 위해 각 버전별 개념, 아키텍처와 마이크로소프트가 WSL2로 업데이트한 이유에 대해 살펴볼 것이다. 

WSL1은 기본적으로 드라이버 형태로 구현된 WSL이라는 Translation Layer 베이스로 동작한다. Linux 유저 모드에서 메모리의 액세스나 파일 접근 또는 네트워크 사용 등과 같은 System call 발생 시 이는 WSL에 전달되고, 윈도우가 이해할 수 있는 System call로 변환되어 NT Kernel로 전달된다. 그럼 이를 처리하고 응답을 줄 때는 다시 WSL을 거쳐 Linux가 이해할 수 있는 형태로 변환되어 Linux 명령어, 도구 실행 등 OS가 동작되는 방식이다.


<p align="center">
  <img src="https://i.imgur.com/ZS1Lxey.png" alt="image"/>
<br>WSL1 Architecture
<br>(https://www.youtube.com/watch?v=lwhMThePdIo)</p>


즉 리눅스 커널 자체 가상화 기술을 사용하지 않고 파일시스템, 파일 처리 등 유저 모드 동작에 필요한 System call을 에뮬레이션 하여 처리하는 방식이다. 실제 Linux 커널이 없기 때문에 장치 드라이버와 같은 커널 모듈은 실행할 수 없다는 단점이 있었다.

그래서 WSL1을 사용할 경우에 tcpdump 도구로 네트워크 패킷 덤프를 수집하려고 해도 지원하지 않는 현상이 있었다. 이처럼 완전한 Linux 커널을 지원하지 않았기 때문에 나처럼 다시 VMWare로 눈을 다시 돌린 경험도 있었을 것이다. 

<p align="center">
  <img src="https://i.imgur.com/nFk1nxu.png" alt="image"/>
</p>

윈도우에서 리눅스를 사용하는데 반쪽짜리 OS가 되고 싶지 않았을 것이다. 그리고 마이크로소프트가 WSL2로 빠르게 전환할 수 밖에 없는 이유가 몇 가지가 더 있다.

**1. 지속적인 WSL Translation Layer 개발 필요**

새로운 리눅스 커널 버전이 나왔을 때마다 개발하고 배포하는데 시간도 오래 걸릴 뿐 아니라 이런 딜레이로 인해 새로운 리눅스 커널을 바로바로 사용하지 못한다는 단점이 있다.

**2. OS의 근본적인 차이**

리눅스와 윈도우는 근본적으로 차이가 있다. 대표적으로 사용하는 파일시스템 구조 자체가 다르고, 메모리 매니징, 권한 등 어느 것이 우월하고 좋다고 말할 수 없는 본질적인 차이가 있다. 이 두 개의 OS를 하나의 시스템에서 잘 구현하려면 베스트 케이스의 경우 호환성을 맞추는 작업이 손쉽게 구현이 가능하나, 그렇지 않은 경우가 상당히 많고 구현하기도 어렵다.

예를 들면 어떤 폴더 내 파일을 Open한 상태에서 파일명을 변경하려고 한다. 윈도우의 경우에는 Win32 API(MoveFile)를 호출하여 변경을 시도하는데 이때 파일이 열려있어 에러가 발생된다. 리눅스의 경우 파일 디스크립터(File Descriptor)를 이용해서 파일을 다루지만 리네임이 가능한 구조로 되어있다. 이처럼 OS 철학이 근본적으로 다르기 때문에 그 사이를 중재해 줄 수 있는 Translation Layer 개발은 앞으로 더 부담스러울 수밖에 없었을 것이다.


### WSL2 Architecture
이런 이유로 WSL2에서는 가상화 기술 베이스로 동작하도록 개발하였다. 실제 리눅스 커널을 가상화하여 동작하게 만들었고, 리눅스 커널이 있기 때문에 MS에서는 100% System call 호환성을 가지고 있다고 얘기하고 있다. 탑재된 리눅스 커널은 마이크로소프트가 커스터마이징 했고 오픈 소스로 Github에도 공개되어 있다.


<p align="center">
  <img src="https://i.imgur.com/jweRRA4.png" alt="image"/>
<br>WSL2 Architecture
<br>(https://www.youtube.com/watch?v=lwhMThePdIo)</p>


WSL2에서 사용하는 가상 머신은 일반적으로 사용하는 VM이 아닌 Lightweight utility VM을 사용하고 있어 실제로 사용해보면 부팅하는데 1~2초 정도밖에 걸리지 않는다. 시스템 리소스의 사용량도 적을뿐더러 필요할 때마다 손쉽게 부팅하여 사용할 수 있게 되어있다.

유저 영역에서 발생된 System call은 중간에 변환되는 절차 없이 바로 Linux Kernel로 전달되기 때문에 I/O 속도가 빠르다고 할 수 있다. 예를 들면 우분투에서 패키지 설치 완료 시간이 5~6배나 빠를 정도이다. 공개된 [WSL Docs](https://docs.microsoft.com/ko-kr/windows/wsl/compare-versions)에서 WSL1과 WSL2의 차이를 아래와 같이 비교하고 있다.

<p align="center">
  <img src="https://i.imgur.com/RX2iChK.png" alt="image"/>
</p>

WSL2가 전체 Linux 커널을 사용할 수 있다고 해서 꼭 좋은 점만 있는 것은 아니다. 위의 표와 같이 OS 파일 시스템 간 성능은 WSL1이 우세하다. WSL1의 경우에는 리눅스 자체 커널의 가상화를 사용하지 않아 Windows NT Kernel에서 직접적으로 모든 파일 시스템을 제어함에 따라 리눅스에 마운트 된 NTFS 파일 시스템이 빠르게 처리가 가능했지만, WSL2에서는 Hypervisor를 통해 윈도우 파일 시스템을 공유하며 처리하는 방식으로 NTFS를 처리하는데 시간이 좀 더 오래 걸린다.

나의 경우에는 리눅스를 통해 분석 대상 파일 시스템의 압축을 풀고 다수의 파일을 정제하거나 처리하는 작업이 많아서 파일 시스템 간 I/O가 많고, 이에 따라 성능의 영향을 많이 받는다. 이럴 경우엔 WSL1을 사용하는게 훨씬 더 빠른 퍼포먼스를 낼 수 있다.


## Wrap-up
기본적으로 활발히 사용중인 WSL2를 추천한다. 특히 도커(Docker) 사용 등 리눅스를 메인 개발 환경으로 설정하고 윈도우에서 IDE로 개발을 하는 경우나 리눅스 OS내 자체 I/O가 많은 경우, 전체 커널 호환성에 초점을 맞춘다면 WSL2 사용을 추천한다. 그렇지 않고 나의 경우와 같이 대부분의 작업이 리눅스와 NTFS로 포맷된 디스크와의 압축 해제, 파일 복사/이동 등 전환 작업이 많은 경우라면 WSL1으로 사용하는 것을 추천한다.

다음 포스트에서는 WSL 활용과 버그(?) 해결 팁, 그리고 윈도우의 패키지 매니저인 Chocolatey에 대해 설명할 예정이다.


## Reference
- <https://docs.microsoft.com/ko-kr/windows/wsl/about>
- <https://docs.microsoft.com/en-us/windows/wsl/compare-versions>
- <https://ksjm0720.tistory.com/10>
- <https://xeppetto.github.io/%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4/WSL-and-Docker/03-What-is-WSL/>
- <https://www.youtube.com/watch?v=MrZolfGm8Zk>
- <https://www.youtube.com/watch?v=lwhMThePdIo>


## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

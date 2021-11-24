---
title : "Blog #31: Building a Forensic Environment with WSL & Chocolatey part 1. [EN]"
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
Architecture of WSL & Tips for building forensic Env.


## This Post Covers
A virtual environment is generally used when performing various analysis such as dissect binaries, or even when simple examine is needed. The main reason for using virtual machine is that we can revert the snapshot, regarding the system was intact at the time of create, over and over again. VM also protects against malicious code infection during the execution of potentially harmful files downloaded from the internet website.

For the first time when configuring the virtual environment, we need whatever tools we need to have for analysis. So, wander around the website in order to download them, then make the **init** snapshot afterwards. That's the tedious job, no-one would argue with that.

As for the Linux OS, however, we use package manager to simplify the job but running Linux on Windows was time & resource-consuming job as VMWare or VirtualBox has to be installed. Some Linux tools are crucial for analysis, but computer resource is always on our minds. In this post series, we're going to cover how to use Linux binaries by setting up a forensic environment with WSL and Chocolatey, and how we can leverage them.

## WSL(Windows Subsystem for Linux)
Windows Subsystem for Linux is a compatibility layer for running Linux binary executables natively on Windows OS without booting a separate Linux like traditional virtual machine software does. If you haven't tried it out, I recommand you should use it or at least try out.

In order to use WSL in the past, we did turn on the Windows features: **Windows Subsystem for Linux**, **Virtual Machine Platform**, and more bits & pieces to get it done. However now all we need to use is single command with administrative privileged PowerShell or Windows Command Prompt and then restarting the system. By default, the installed Linux distribution is Ubuntu.

```powershell
wsl --install
```

<p align="center">
  <img src="https://i.imgur.com/bZnmtT5.png" alt="image"/>
</p>

### WSL1 Architecture
Let's dive a little deep into WSL, the architecture between WSL1 and WSL2 followed by reasons why Microsoft moved WSL2 from the predecessor.

WSL1 basically operates translation based approach. The Translation Layer called WSL is implemented in the form of a driver. When a user space system call like accessing memory, handling file or network occurs from Linux, it is transmitted down to WSL, converted into a system call that Windows can understand, and transmitted to the Windows NT Kernel. When NT Kernel gives a response back it goes through WSL again traslate back to something in the form of Linux would understand.


<p align="center">
  <img src="https://i.imgur.com/ZS1Lxey.png" alt="image"/>
<br>WSL1 Architecture
<br>(https://www.youtube.com/watch?v=lwhMThePdIo)</p>


The system calls are dealt with by emulation, providing an application binary interface for Linux. There's no real Linux Kernel in WSL1, so the device drivers cannot be run because they actually run on Linux Kernel. That's why it was not able to use the tools like tcpdump to capture the network packet in WSL1 and because of this, many people including me turned their eyes back to VMWare once in the past.

<p align="center">
  <img src="https://i.imgur.com/nFk1nxu.png" alt="image"/>
</p>

It's like half-Linux running on Windows, and Microsoft definitely didn't want that too. This leads to move to WSL2 from its predecessor, and there are more reasons as follows:

**1. Constant WSL Translation Layer Development**

Linux distros are active, new kernel updates frequently out of nowhere and it persues new, state-of-the-art challenging features. This makes MS constant development of WSL, QA, publish, and more. Due to this delay, users wouldn't be able to use user space binaries that run on new featured kernel in a timely manner.

**2. Different Semantics**

Windows and Linux have very different semantics, they are fundamentally different. Filesystem structure, managing memory, permissions, it's not which-one-is-better-and-best matter. Best case scenario, development of WSL goes smoothe in terms of compatibility, but there are a bunch of cases that it's not easy and challenging for MS.

As an example, when we try renaming a file while it's opened, Windows calls Win32 API(MoveFile) to rename the file but we get the error because of the file's state. Having said that, in the case of Linux, a file is handled using FD(File Descriptor), but it has a structure that can be renamed. As the OS philosophy is fundamentally different, the development of a Translation Layer that can mediate between them would have to be more burdensome in the future.


### WSL2 Architecture
In WSL2, it operates virtualization based approach rather than translation based. Microsoft puts real Linux Kernel inside, so MS says that leads 100% system call compatibility. Linux Kernel is developed by MS, maintained by MS and it's open-source availble on Github.


<p align="center">
  <img src="https://i.imgur.com/jweRRA4.png" alt="image"/>
<br>WSL2 Architecture
<br>(https://www.youtube.com/watch?v=lwhMThePdIo)</p>


Unlike traditional VM, WSL2 uses Lightweight utility VM that is fast, and it boots up under one or two seconds. Furthermore, it does not take much resources of your system if you've tried out.

System call from your linux user space binaries it calls down to the kernel so the speed of I/O is fast, the package installation time is 5-6 times fast compared to WSL1.

[WSL Docs](https://docs.microsoft.com/en-us/windows/wsl/compare-versions) compares features between WSL1 and WSL2 below.

<p align="center">
  <img src="https://i.imgur.com/PXKNyzIl.png" alt="image"/>
</p>


Just because WSL2 can use the full Linux kernel doesn't necessarily mean it's perfect. In the case of WSL1, because Linux Kernel itself was not virtualized, all file systems were handled directly by one Windows NT Kernel resulting in quick file accessing between two file systems.

In WSL2, the Windows file system is shared and processed through a hypervisor that takes a longer time to deal with NTFS from Linux. Most of my forensic work engages with extract files for file system analysis, so there are so much I/O between Linux OS and NTFS formatted disk. In this case, using WSL1 can give you much faster performance.


## Wrap-up
Basically WSL2 which is being actively used, is recommended. In particular cases like below:

- Linux configured as the main development environment(Docker, etc), and use an IDE from Windows side.
- I/O mainly in the Linux side, not many accessing NTFS side.
- Focus on overall Linux Kernel compatibility.
 
Otherwise, if most of the work involves conversion between Linux and NTFS, decompression and file copy/move, WSL1 is highly recommended. In the next post, we will cover the use of WSL, tips and Chocolatey, Windows package manager.


## Reference
- <https://docs.microsoft.com/ko-kr/windows/wsl/about>
- <https://docs.microsoft.com/en-us/windows/wsl/compare-versions>
- <https://ksjm0720.tistory.com/10>
- <https://xeppetto.github.io/%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4/WSL-and-Docker/03-What-is-WSL/>
- <https://www.youtube.com/watch?v=MrZolfGm8Zk>
- <https://www.youtube.com/watch?v=lwhMThePdIo>


## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>s
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

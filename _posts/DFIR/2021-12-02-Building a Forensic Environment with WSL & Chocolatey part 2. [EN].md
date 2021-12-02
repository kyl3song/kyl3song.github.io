---
title : "Blog #32: Building a Forensic Environment with WSL & Chocolatey part 2. [EN]"
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
Ways to increase your productivity


## This Post Covers
As getting used to WSL, we may want the detailed features of terminal app like split panes, color schemes, backgrounds, and so on. Terminal apps are great, we can increase our productivity by using terminal programs such as Tmux and Terminator rather than having a hard time but let it go with the default terminal application.

Best buddy for WSL is probably the **Windows Terminal**. You may be stunned how Microsoft made such a good program and sometimes there will be a tragic(?) situation made where you are way more interested in Windows Terminal than WSL itself.

In this post, I would like to introduce tips for combining WSL and Windows Terminal, as well as giving tips to solve the character cluttered issue(IME compatibility bug) while using the terminal. Finally, we'll look at how to build a test environment more easily with Chocolatey.

## Windows Terminal
The app can be downloaded from the Microsoft Store, open source, you can check the code at [GitHub](https://github.com/microsoft/terminal).

<p align="center">
  <img src="https://i.imgur.com/Fz8fIG3.png" alt="image"/>
</p>

WT has lots of great reviews with stars in Microsoft Store, commented "MS has outdone" sort of reviews. WT supports multi-terminals, great color scheme, decent default font to use and has overall developer-loving ambience. The figure below compares before and after using the WT.

<p align="center">
  <img src="https://i.imgur.com/uELmlMc.png" alt="image"/>
</p>

When it first made its debut with preview version, we had to manually edit the json file to change the configuration. The style, shortcut key, first shell that opens when loading WT, etc. But now, we can easily set it up through the GUI style options in Settings.

### Tip 1. Change Default Profile and Pin WT to Taskbar

The default profile is PowerShell. In other words, when the terminal is opened, the first executed shell is the PowerShell. If we change it to the installed Ubuntu and pin WT to taskbar, we can go fast to meet the Ubuntu environment simply by clicking the WT icon.

> At this point(Nov. 2021), Ubuntu 20.04 LTS is the default Linux distro that is automatically installed as part of the installation procedure of WSL.

<p align="center">
  <img src="https://i.imgur.com/YljuByO.png" alt="image"/>
</p>

Additionally, for Windows tips, keyboard shortcuts are used to launch programs which pinned to the Windows taskbar. They can be executed in icon order, so <kbd>Win</kbd> + <kbd>1</kbd> is the first order of each items. The combination default profile as Ubuntu and WT taskbar will save your time.

<p align="center">
  <img src="https://i.imgur.com/xc7Biub.png" alt="image"/>
</p>


### Tip 2. Keyboard Shortcuts in WT
There are tons of shortcut keys we can use in WT. Most used ones are the following:

- <kbd>Ctrl</kbd> + <kbd>+</kbd> : Increase the font size
- <kbd>Ctrl</kbd> + <kbd>-</kbd> : Decrease the font size
- <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>T</kbd> : Open new termianl session
- <kbd>Alt</kbd> + <kbd>Shift</kbd> + <kbd>D</kbd> : Split panes
- <kbd>Alt</kbd> + <kbd>Shift</kbd> + <kbd>↑ or ↓ or → or ←</kbd> : Resize the current pane up/down/left/right
- <kbd>Alt</kbd> + <kbd>↑ or ↓ or → or ←</kbd> : Move pane focus to the arrow direction


<p align="center">
  <img src="https://i.imgur.com/Pj35cIA.png" alt="image"/>
</p>

I recommend this [blog](https://allthings.how/how-to-use-windows-terminal-keyboard-shortcuts/) if you want to customize and change keyboard shortcuts.

### Tip 3. Text Alignment Bug (aka. Korean IME Bug)

When using WT, sometimes you may experience overlapping characters while typing in WSL. The issue occurs when running a Shell, no matter what Shell(PS, CMD, Bash) your're using. This is the same issue that someone wrote in an app review.

<p align="center">
  <img src="https://i.imgur.com/h62hdCn.png" alt="image"/>
</p>

The phenomenon in which characters, whether in English or Korean, are written by order(left to right) but the cursor remains at the front. 

Not only it looks weird, but especially when you enter a password, like when make SSH connection or even sudo password to install packages, the password is exposed in terminal in spite of the fact that it should be masked.

<p align="center">
  <img src="https://i.imgur.com/01kqZjY.png" alt="image"/>
</p>

Reason for this issue is because of the Hancom Korean IME(Input Method Editor). Word Processor named Hancom is generally used in many companies, government offices in Korea. So when Hancom Office is installed, the IME is also installed in your pc.

You will have the string cluttered problem and password will not masked, when using WT with the IME selected. I guess countries that have their own languages will possibly have the same IME compatibility issue.

To resolve the issue, use WT with Microsoft IME(default one) selected, or you may remove the Hancom IME. In my case, after removing it, there hasn't been a single issue using hancom app or other apps so far. Below is how to remove Hancom IME in Windows 10.

- ***Languages ➜ Preferred languages ➜ Korean ➜ Remove***


## Chocolatey
Chocolatey is a command-line package manager and installer for Windows software. It simplifies the process of downloading and installing software. Visit the [official site](https://chocolatey.org) to download it.


Similar to Homebrew(of macOS) or apt(of Ubuntu), you can install the program by using PowerShell or Command Prompt. Also there is GUI version of Chocolatey to search, install, update packages if you're not familiar to CLI. Packages can be searched at [Community Packages](https://community.chocolatey.org/packages). 

## How to use
A package can be installed with an administrator-privileged PowerShell command. The first syntax of the script is to change the default execution policy(Restricted) set to bypass to prevent the execution of malicious scripts in PowerShell.

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

Adding parameter **-y**, package installation proceeds without asking whether to install or not. When you use package names in a row, you could install as many packages as you wish to install with a single command, same as Linux apt installation.

```powershell
Usage: choco install -y [package_name_1] [package_name_2]...
```

## Recommended Packages
Among the many packages, the recommend one in terms of linux binaries would be the **UnxUtils**. It contains linux tools such as grep, dd, wc, so it would be easy to get tem together to set up the analysis environment.

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


## Binary Location & Debug Logs
All packages are installed in the **C:\ProgramData\chocolatey** path, and the binary files ported as exe for Windows are collected in the **C:\ProgramData\chocolatey\bin** folder. Since tools can be executed from any path in the system, the environment variable is automatically set.

<p align="center">
  <img src="https://i.imgur.com/6gXuONq.png" alt="image"/>
</p>

Debug log left while operating Chocolatey, so to check the full command when installing package with what parammeter, use the keyword **Command line:** to only match the result.

- **Log Path: C:\ProgramData\chocolatey\logs\chocolatey.log**

<p align="center">
  <img src="https://i.imgur.com/4O9O8Nw.png" alt="image"/>
</p>



## Wrap-up
So far, We have looked at bits & pieces to configure analysis environment using WSL(+Windows Termianl) and Chocolatey. In particular, when setting up the environment, it would be good to list up the necessary programs without having to go around and download them from site to site. With WSL & Chocolatey, why not reduce unnecessary tasks and increase your productivity.


## Reference
- <https://github.com/microsoft/terminal>
- <https://chocolatey.org>
- <https://community.chocolatey.org/packages>
- <https://allthings.how/how-to-use-windows-terminal-keyboard-shortcuts/>


## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

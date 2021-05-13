---
title : "Blog #26: Importance of Drive Trim in Forensic Imager part 2. [EN]"
category :
  - Acquisition
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - Falcon-NEO
  - Mirror Clone
  - Drive to Drive
  - Drive Trim
  - EnCase 20.4
  - X-Ways Forensics 20.1
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
Why we need to use Drive Trim part 2


## Drive Trim Series
- [Blog #25: Importance of Drive Trim in Forensic Imager part 1. [EN]](https://kyl3song.github.io/acquisition/Importance-of-Drive-Trim-in-Forensic-Imager-part-1.-EN/)
- [Blog #26: Importance of Drive Trim in Forensic Imager part 2. [EN]](https://kyl3song.github.io/acquisition/Importance-of-Drive-Trim-in-Forensic-Imager-part-2.-EN/)

## This Post Covers
Previously, we confirmed that when mirror cloned without drive trim, under the different capacity of Source and Destination condition, the drive hash of Destination can be exactly matched to Source's hash value by configuring LBA options before computing drive hash.

In this post, we're going to think about how to perform forensic analaysis in this case and know the best practice, also let's dive a little deeper into the Drive Trim option, which is **the most important part**.

## How to Do an Analysis with Mirror-cloned disk w/o. Drive Trim
Now we understand if we connect Destination HDD to our workstation it will be recongnized as 1TB HDD by the OS. You would probably hesitate to do an automatic analysis like Process(Encase) or Refine Volume Snapshot(XWF) because all we want is to process 4GB(Source) portion of Destination drive.

It seems that it can be devided into two cases, it's the matter of whole Destination disk except 4GB is wiped.

### Case 1. Destination is not wiped before mirror clone
If not wiped, the data that previously used in the past can be recovered together resulting in forensic examiners confusing.

Issues to think about when assuming data still remains is the possibility of forensic tools are able to compute hash in specific range of LBAs, and perform automatic analysis whthin certain block of sectors (LBAs).


**1. Possibility of hash verification based on LBAs in tools**

EnCase 20.4 supports drive hash with predefined sectors, but it seems X-Ways Forensics 20.1 does not support that. Also the criteria can be different by software, Falcon-NEO has **LBA counts** whereas **Start/Stop Sectors** in EnCase. So it totally depends on the tools that we use in the analysis phase.

EnCase for example, based on the Falcon Log tells us **7,831,552** for LBA count, it's equivalent to '0 ~ 7,831,551 sector' as start/end position. Thus, like Fig 1. below, we need to specify 0 for Start Sector and 7,831,551 **(LBA count-1)** for End Sector.

<p align="center">
  <img src="https://i.imgur.com/sjmTmAi.png" alt="image"/>
<br>[ Fig 1. Drive Hash with Sectors ]</p>

We definitely need to beware of the term 'count' itself when doing this, otherwise we might misjudge & conclude that two hash do not match. Fig 2. represents the result of hash verfied within a range of sectors.

<p align="center">
  <img src="https://i.imgur.com/y1npFv6.png" alt="image"/>
<br>[ Fig 2. Hash Verified by EnCase v20.4]</p>


**2. Possibility of an anlaysis within LBAs in tools**

Forensic tools has splendid automatic analysis feature like Process, RVS that examiners make full use of, however, the majority of tools do not have the option to perform this auto-analysis only within a particular area of disk.

Well some say that we could possibly make a script to scan the range and make that happen, but it's not a piece of cake and most of all, commercial tools have way more options & features that one person's script. Let's think about the other smart way.

**3. Best Practice in analysis phase**

Best practice would be perform re-image only 4GB(Source) of Destination drive.
It would be much eaiser to analyze e01 files imaged off the Destination with tools so that it does not confuse us, no need to have extra care of things in order to make a script for automatic analysis.

One thing we need to keep in mind is to ensure a valid digital evidence CoC(Chain of Custody), so we have to note what we have to do, why we need to do, comment everything that has something to do with in the forensic report.


### Case 2. Destination is wiped before mirror clone
If wiped, different from case 1, it would not be very a big problem except for taking a long time to do automatic process. Forensic tools will consider wiped area(996GB) as slack space, and if we consider carving in byte-level, it could take a while to be finished.

That also colud lead us re-image method the Best Practice.


## Cautions for Drive Trim
Let's get back to where we were when using drive trim with Falcon-NEO. We did test out drive trim function in the previous post. If we take a look at the Fig 3. bottom left, the result log shows us the option is successfully applied with **Pass**, highlighted in green.

<p align="center">
  <img src="https://i.imgur.com/FtIuoBn.png" alt="image"/>
<br>[ Fig 3. Falcon Log (Mirror Clone with Drive Trim option) ]</p>

This time we will use portable USB HDD for the same test. Connect the Source(4GB USB Stick) in the left side, Destination(250GB portable HDD) in the right.

<p align="center">
  <img src="https://i.imgur.com/BQoPGQ6.png" alt="image"/>
<br>[ Fig 4. Falcon Neo with Source(4GB, left) & Destination(250GB, right) ]</p>

Select Drive-to-Drive(Mirror clone) mode, and Trim option changed to enable then start drive cloning.

<p align="center">
  <img src="https://i.imgur.com/yxAFEto.png" alt="image"/>
<br>[ Fig 5. Mirror Clone Option Menu ]</p>

Falcon log describes Bay, Model, Serial, Capacity, all are okay. However if we see the bottom part, **Type: Not Applicable** with **<span style="color:red">Skipped</span>** as the result.

<p align="center">
  <img src="https://i.imgur.com/EMaGvSn.png" alt="image"/>
<br>[ Fig 6. Drive Trim not applicable with USB portable HDD ]</p>

To make sure, I connected the portable HDD to my workstation and Windows recognized it 250GB HDD, not 4GB. Drive Trim is not applied at all.


### Why not applicable & skipped?
Drive Trim option is applicable only though the ATA interface. To change the size of disk logically it should manipulate HPA(Host Protected Area), DCO(Device Configuration Overlay) area which I mentioned in the previous post.

Falcon uses command set interating with disk controller directly to make Destination perfect fit to Source drive.

- HPA: SET MAX ADDRESS
- DCO: DEVICE CONFIGURATION SET / DCO MODIFY
- ACS3: ACCESSIBLE MAX ADDRESS (ATA/ATAPI Command Set)

The bottom line is, using SAS drives or drives that connect with USB, PCIe or any ohter ports except SATA(ATA), Drive Trim will not applied.

In many cases we use usb type external SSD/HDD because it's easy to use, compact, and may reasons, but we should pay extra attention when using these devices especially in forensic work.

Falcon-NEO manual reads as follows:
> Drive Trim only works with ATA drives connected to the 
SAS/SATA Destination ports. Drive trim will not work with 
SAS drives or drives connected to the USB, PCIe, or I/O ports.


## Wrap-up (TL;DR)
I explained the details about the best practice of analysis of mirror-cloned disk without drive trim, and iterated important thing to know when using drive trim up to now.

Here's TL;DR below.

### 1. Method of Hash verification in tools
- Use LBA counts option for drive hash.
- Check the terms in tools, whether it's Start/End sectors or LBA Counts.

### 2. Best Practice in anlysis phase
- Since almost every tools do not have the options to perform automatic analysis within a user-defined range of disk, re-image only the Source area off Destination drive and use the e01 file for anlaysis.
- Make comments how you do, why it needs to be done, everything that has to do with in the forensic report.

### 3. Cautions for Drive Trim
- Manipulate HPA/DCO/ACS3 by interacting disk controller with command sets.
- Drive Trim will not work with SAS drives or drives with USB, PCIe, etc except SATA(ATA) ports.


## Reference
- <https://www.logicube.com/wp-content/uploads/2019/10/MAN-Falcon-NEO-v2.3-FIN-2.pdf>
- <https://www.logicube.com/wp-content/uploads/2017/08/MAN-Falcon-v3.2-FIN.pdf>
- <http://forensic-proof.com/archives/284>
- <http://www.ktword.co.kr/abbr_view.php?m_temp1=2108>
- <https://en.wikipedia.org/wiki/Host_protected_area>


## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

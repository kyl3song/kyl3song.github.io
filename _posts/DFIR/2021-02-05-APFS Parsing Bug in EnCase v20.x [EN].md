---
title : "Blog #24: APFS Parsing Bug in EnCase v20.x [EN]"
category :
  - Software & Tools
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - File System
  - APFS
  - Apple File System
  - macOS
  - EnCase v20.4
  - EnCase v8.11
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
APFS Parsing Bug & comparison between EnCase versions.


## This Post Covers
When BitLocker-encrypted disk image gets loaded in the forensic tools such as EnCase, it requires you to enter the password with a small pop-up window.

Likewise above, after acquiring APFS(Apple File System) with the FileVault password applied, EnCase also asks for the password in order to interpret the file system. That's how it works.

Recently I've confirmed that even if I enter the valid password in EnCase v20.x, it seems the tool does not parse the whole file system. Let's see how it deals with the file system.


## EnCase Versions
There are numerous versions of EnCase which OpenText updates its product release periodically. An update in 2020, I was pretty shocked by the fact that the version was a big leap from v8.11 to v20.x and UI was also changed with blue-ish(or dark navy) color.

That makes me have a little expect something big major change in EnCase. but it looks like not so much of changes have made than I personally expected.

To cut to the chase, from 2020 OpenText releases new version of EnCase every quarter, four times a year. The version is managed under year & quarter basis as follows:

**(YEAR).(QUARTER) : 20.1 ➔ 20.2 ➔ 20.3 ➔ 20.4.**

<p align="center">
  <img src="https://i.imgur.com/Ob2yMYx.png" alt="image"/>
<br>[ Fig 1. Product Release Timeline ]
<br>(Source: OpenText Forensics DataExpert Webinar) </p>


## EnCase v20.4 vs. v8.11
The comparison test is conducted with EnCase v8.11, the final version of v8.x series, and the current latest release of v20.4, loading the same APFS imgae on each products and compare the basic file system parsing feature between them.

Both in v8.11 and v20.4, we are prompted to enter the filevault password to decrypt the APFS as expected. However, we can spot a slight difference between them. If we type the valid password in v20.4, the tool gets us the password prompt window again as if it does not match, and one more time again. so, **three times in total.**


<p align="center">
  <img src="https://i.imgur.com/5omLpTu.png" alt="image"/>
<br>[ Fig 2. Password required window]</p>

As we get the same password required prompt three times, EnCase v20.4 now pretends to interpret file system, parsing Catalog, processing FS Tree, and so on. This is where we need to be very cautious about when examining APFS.

<p align="center">
  <img src="https://i.imgur.com/ihpp8JY.png" alt="image"/>
<br>[ Fig 3. Processing FS Tree in EnCase v20.4 ]</p>

Take a look at the volumes in APFS Container (**disk1 - APFS Continer (synthesized)**) listed by Macquisition before performing acqusition in Fig. 4 below.

We could see and notice five volumes in disk 1, among them **<span style="color:red">Macintosh HD - 데이터</span>** is the right volume that user data is saved.

<p align="center">
  <img src="https://i.imgur.com/kQdPBMW.png" alt="image"/>
<br>[ Fig 4. Disk Drives listed by Macquisition ]</p>

Let's compare the difference by versions under the five categories after tessting.

### 1. Volume List in APFS Container
v8.11 parses APFS Superblock and pinpoints exact five volumes in the tool, whereas v20.4 seems to partially parse the file system which includes vague and undetermined something.

<p align="center">
  <img src="https://i.imgur.com/fdZcHBC.png" alt="image"/>
<br>[ Fig 5. Volume List - EnCase v8.11 ]</p>

<p align="center">
  <img src="https://i.imgur.com/zSdIJ9u.png" alt="image"/>
<br>[ Fig 6. Volume List - EnCase v20.4 ]</p>


### 2. Data validity
Entries are all valid and accessible in EnCase v8.11, we could see the personal photos in the viewer pane bottom left, play videos just as normal.

<p align="center">
  <img src="https://i.imgur.com/hBosCjR.png" alt="image"/>
<br>[ Fig 7. EnCase v8.11 - IMG_0459.JPG (valid picture) ]</p>

In v20.4 however, filenames and folders may look okay but, we can't see the majority of photos. Some photos can be seen unclear like 30% of them are only valid.

<p align="center">
  <img src="https://i.imgur.com/5S48jBF.png" alt="image"/>
<br>[ Fig 8. EnCase v20.4 - IMG_0459.JPG (invalid picture) ]</p>

I guess it's somehow similar to NTFS file system issue in which some sectors are missing out of nowhere.
It's like the tool could access $MFT to parse the filenames, folders, metadata, but data run(cluster run)'s offset or length not exactly points to the real data area.

<p align="center">
  <img src="https://i.imgur.com/f8LUeYW.png" alt="image"/>
<br>[ Fig 9. EnCase v20.4 (partially invalid) ]</p>


### 3. Export Files & Compare them
We need to verify that the start offset of the file is really the start point.

After checked out we could tell the start offset is correct because the header of JPG laid highlighted in blue.

<p align="center">
  <img src="https://i.imgur.com/oeqSLkf.png" alt="image"/>
<br>[ Fig 10. Header of the sample picture in EnCase v20.4]</p>

Getting deeper into the issue, we extract a sample photo in each version and check out the differences.

Same data is filled with until the offset 0x0FFF, but from the **offset 0x1000(4096 bytes)** EnCase v20.4 reveals totally different hexadecimal values than v8.11's. What's more, the very end offset is not even jpeg's footer signature.

Yes, it's totally off the grid.

<p align="center">
  <img src="https://i.imgur.com/ldnpOBU.png" alt="image"/>
<br>[ Fig 11. Data compare between v8.11 & v20.4]</p>

EnCase v20.4 sometimes makes an alert saying that it cannot extract the file, so it should be a product bug that definitely needs to be fixed.


### 4. Entry Counts
Last but not least, entry counts differs by version. Total entry counts in v8.11 is **1,141,823** whereas **622,533** in v20.4.

<p align="center">
  <img src="https://i.imgur.com/5ft217w.png" alt="image"/>
<br>[ Fig 12. Entry counts between v8.11 & v20.4]</p>


## Wrap-up
A few months back, I experienced this APFS parse bug in EnCase v20.2 and thought that it should be fixed in the next release which is v20.4 but still have the same issue, not fixed yet.

You're going to have to take extra care when dealing with APFS forensics in EnCase for the time being.


## Reference
- <https://developer.apple.com/support/downloads/Apple-File-System-Reference.pdf>
- <https://www.dataexpert.nl/media/rczo0203/opentext-forensics_dataexpert_webinar.pdf>



## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

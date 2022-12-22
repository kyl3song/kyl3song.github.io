---
title : "Phone Scam Series: USB Modem with Asterisk Analysis and Visualize artifacts"
category :
  - IoT & Embedded
tag : 
  - USB Modem
  - Phone Scams
  - Asterisk
  - Visualization

sidebar_main : true
author_profile : true
use_math : true
toc: true
toc_sticky: true
toc_label: "Table of Contents"
header:
  overlay_image : /assets/images/post.jpg
  overlay_filter: 0.5
published: true
---
보이스피싱 시리즈: Asterisk 아티팩트 분석 및 시각화

Previously we've covered the artifacts left when using the USB Modem through the program named Huawei Mobile Partner. The program only takes one modem at a time so you call, text one-on-one.

The case that I introduce in this blog post is about to see how multiple usb modems can be used with Asterisk app and check the remain artifacts.

## Case Study: USB Modem with Asterisk

### What is Asterisk?

[Asterisk](https://www.asterisk.org/) is an open source framework for building communications applications sponsored by Sangoma. It turns an ordinary computer into a communications server.
You can use Asterisk to build communications applications, things like business phone systems (also known as IP PBXs), call distributors, **VoIP gateways** and conference bridges.

In this case, it is to replace the role of VoIP Gateway, the physical box. Following network diagram shows the overall network flow when using USB modems through asterisk application.

<p align="center">
  <img src="https://i.imgur.com/Z7OWlB0.png" alt="image"/>
</p>

In the figure above, a number of modems are connected to a PC through a usb hub, and connected to an external IP-PBX. This allows to change the calling number. Since the Asterisk application is linux machine based, it is convienent to use the ubuntu virtual machine to install the package.

Investigating PCs where USB Modems have been used so far, all Ubuntu VMs have been used so as to avoid reinstalling, configuring the environment over and over again.

### Key Artifacts
Before getting into the artifacts, a lot of times I discoverd remote control programs are used beforehand to commit crimes. For example, from many cases, they make small local groups install Teamviewer/SunloginClient applications first, then they take control to install Virtualbox & Asterisk, run prepared script to finish the configuration. So when investigating this case, be sure to examine the bash history.

The artifacts are mainly saved in /var/log/asterisk, and config values ​​remain in the /etc/asterisk path. The key artifacts narrow down as following:

- **/var/log/asterisk/cdr-csv/Master.csv** : Call Logs
- **/var/log/asterisk/messages** : Daemon Logs (package operation, fail logs)
- **/etc/asterisk/dongle.conf** : IMEI of USB Modem(SIM card), Extension number
- **/etc/asterisk/extensions.conf** : Phone number(from SIM card), Voice codecs, Extension number
- **/etc/asterisk/sip.conf** : IP PBX(SIP Server) IP address

---

**1. Master.csv**

<p align="center">
  <img src="https://i.imgur.com/Y2n3YBV.png" alt="image"/>
</p>

Master.csv is a file in which logs of incoming and outgoing calls through a USB modem are recorded, and the file is saved in CSV format. The names of each field were not recorded in the logs but the meaning of each field can be found at [asterisk docs](http://www.asteriskdocs.org/en/3rd_Edition/asterisk-book-html-chunk/asterisk-SysAdmin-SECT-1.html) or [cdr_csv.c](https://github.com/asterisk/asterisk/blob/master/cdr/cdr_csv.c).


|No| Field Name | Description |
|:---:|:---:|:---|
|1|accountcode|account (string, 20 characters)|
|**2**|**src**|**Caller ID number (string, 80 characters)**|
|**3**|**dst**|**Destination extension (string, 80 characters)**|
|4|dcontext|Destination context (string, 80 characters)|
|5|clid|Caller ID with text (80 characters)|
|6|channel|Channel used (80 characters)|
|7|dstchannel|Destination channel if appropriate (80 characters)|
|8|lastapp|Last application if appropriate (80 characters)|
|9|lastdata|Last application data (arguments) (80 characters)|
|**10**|**start**|**Start of call (date/time)**|
|**11**|**answer**|**Answer of call (date/time)**|
|**12**|**end**|**End of call (date/time)**|
|**13**|**duration**|**Total time in system, in seconds (integer)**|
|14|billsec|Total time call is up, in seconds (integer)|
|**15**|**disposition**|**What happened to the call: ANSWERED, NO ANSWER, BUSY, FAILED**|
|16|amaflags|DOCUMENTATION, BILL, IGNORE etc, specified on a per channel basis like accountcode|
|17|Uniqueid|Unique Channel Identifier (32 characters) (In some cases, uniqueid is appended)|


The data in ***dst*** are recorded in the form of src+dst number concatenated.

<p align="center">
  <img src="https://i.imgur.com/sdLGbI0.png" alt="image"/>
</p>

We could also get the caller number from ***lastdata*** field. Asterisk used system timezone by default, so the ***start, answer, end*** timestamps are based on the system time.

<p align="center">
  <img src="https://i.imgur.com/eVKC99b.png" alt="image"/>
</p>


---

**2. messages**

In the messages file, daemon logs, operation/fail logs related to Asterisk are stored.

<p align="center">
  <img src="https://i.imgur.com/fPwOos1.png" alt="image"/>
</p>

---

**3. dongle.conf**

IMEI of each dongle and extension numbers used as the originating number through the USB Modem remain in dongle.conf.

<p align="center">
  <img src="https://i.imgur.com/PuhIIgZ.png" alt="image"/>
</p>

---

**4. extensions.conf**

Inside the extensions.conf file, the phone numbers(from SIM cards) and voice codecs, extension number configured to use USB modem can be discovered.

<p align="center">
  <img src="https://i.imgur.com/bKqWlCu.png" alt="image"/>
</p>

---

**5. sip.conf**

Lastly, USB modem should be facing the IP-PBX(SIP Server) to establish, release, change call sessions. Here in this case, the public IP remains.


## Visualize Artifacts

Visualization is very important in data analysis. So putting the artifacts all together detailed above, let's visualize them with the web application. I have put a little effort to make it interactive just to practice for future.

### Dashboard

**Phone Scam Analysis Dashboard**

<p align="center">
  <img src="https://i.imgur.com/6PiGmMq.png" alt="image"/>
</p>

**Call Count Analysis (Daily, Monthly)**

<p align="center">
  <img src="https://i.imgur.com/MEzuaQ3.png" alt="image"/>
</p>

**Call Heatmap by Call Counts/Duration (Daily, Monthly)**

<p align="center">
  <img src="https://i.imgur.com/8hnx2PN.png" alt="image"/>
</p>

**Call Duration per Recipient (Top10, Top20, Top50)**

<p align="center">
  <img src="https://i.imgur.com/YW3wiv8.png" alt="image"/>
</p>


### RAW Data

<p align="center">
  <img src="https://i.imgur.com/syik6ju.png" alt="image"/>
</p>

<p align="center">
  <img src="https://i.imgur.com/BjxCJwK.png" alt="image"/>
</p>


## Wrap-up

From this phone scam forensics series, we have convered the three major forensic cases:

1. VoIP Gateway
2. USB Modem with windows app(Mobile Partner)
3. USB Modem with Asterisk

And yet many devices are still used as part of phone scam crimes, though we cannot conver all of them.


## Reference
- <https://www.asterisk.org/>
- <http://www.asteriskdocs.org/en/3rd_Edition/asterisk-book-html-chunk/asterisk-SysAdmin-SECT-1.html>
- <https://github.com/asterisk/asterisk/blob/master/cdr/cdr_csv.c>



## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

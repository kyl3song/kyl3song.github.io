---
title : "Phone Scam Series: VoIP Gateway Forensics"
category :
  - IoT & Embedded
tag : 
  - VoIP Gateway
  - Phone Scams

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
보이스피싱 시리즈: VoIP Gateway 분석


## Phone Scam

**Phone scam crime** is one of the top of financial crimes in South Korea. It causes cost damages of over $500M per year. Scammers have figured out countless ways to cheat you out of your money over the phone. They pretend to be law enforcement, banks or any other financial agencies, saying there’s been illegal activities on your account so they need to wire the deposit to safer account. Not to mention some unknown telemarketer advise you to change a loan with lower interest rates.

Unfortunately, VoIP Gateways are part of the crime. The device can be used to manipulate the calling number, so to make higher chance to deceive people to take their money, even life savings.


## Criminal Behavior 

From what I have experienced in Digital Forensics, in 2018-2019 the scammers use VoIP Gateway equipped with illegally obtained SIM Cards. SIMs are directly inserted to spoof the calling number.

<p align="center">
  <img src="https://i.imgur.com/8P8hunt.png" alt="image"/>
</p>

In 2020-2021, they cunningly changed their method to separate SIM Cards from VoIP Gateway. Instead, they use the SIM Pool, device that can hold at least 128 SIM cards, in order to service 3G/LTE without interruption as well as lower the possilbility of getting seized when they are busted onsite.

<p align="center">
  <img src="https://i.imgur.com/hAMOvQE.png" alt="image"/>
</p>

And from late 2021, scammers have started using USB modems with SIM cards because it is harder to supply the gateways after massive crackdown by the police and getting harder to import through the customs as well.

<p align="center">
  <img src="https://i.imgur.com/Z7OWlB0.png" alt="image"/>
</p>


## Case Study: VoIP Gateway

### VoIP Gateway Internals
Let's dissect the device where we can see onsite. VoIP Gateway has an ethernet interface frontside, and multiple PSTN(Public Switched Telephone Network) interfaces backside to inter-connect between individual networks.

<p align="center">
  <img src="https://i.imgur.com/N4Vj1LU.png" alt="image"/>
</p>

The chip which we need to look into is the Flash Memory. The device has 32MB flash memory mounted on the PCB, but with very rare case, 16MB flash can also be found.


### Partitions & Data
There are seven partitions in total, the most important one is USER partition. USER partition uses JFFS2(Journalling Flash File System version 2) filesystem, which makes us have an option to use the [jefferson](https://github.com/sviehb/jefferson) tool to easily extract filesystem.

<p align="center">
  <img src="https://i.imgur.com/bla9diu.png" alt="image"/>
</p>

Forensically rich information is located in the highlighted blocks. ENV partition has also juicy information. Non-volatile data, described in the table below are all in the USER partition. Volatile data are all gone when we power off the device.


<p align="center">
  <img src="https://i.imgur.com/eKfp50K.png" alt="image"/>
</p>


### Received SMS from RAM dump
What we can get from the RAM dump. Received SMS are temporarily saved in the RAM. The GUI of VoIP Gateway shows us the Port(Slot Number), Sender, Receiver, Time, SCTS and Content of SMS. The interesting part is we figured out that the **order of received SMS** and **SMSC** which they are not even seen in the GUI.

<p align="center">
  <img src="https://i.imgur.com/jgDEoaE.png" alt="image"/>
</p>

**[Key Data saved in RAM]**

- SIM slot number: It starts from '0' (zero based index)
- **Order of received SMS**
- Date & Time of received SMS (Based on TimeZone)
- **SMSC(Short Message Service Center)**
- SCTS(Service Center Time Stamp)

> A Short Message Service Center is a network element in the mobile telephone network. Its purpose is to store, forward, convert and deliver Short Message Service messages. The full designation of an SMSC according to 3GPP is Short Message Service.

I tried finding the structure of the sent message from the RAM dump, I could see the message content itself, but it doesn't seem to have a specific structure and disappers so fast. Having said that, the counts of sent SMS and sendfails are saved in the non-volatile configuration file.

<p align="center">
  <img src="https://i.imgur.com/aMeAfpnm.png" alt="image"/>
</p>

### Network Traffic Analysis

If we capture the packets between VoIP Gateway and one of the Remote Management Systems the gateway regularly transmit the heatbeat packet that contains its MAC address, SIM slot status, and more information. Moreover, the gateways sends packets when it's triggered by activities such as sending & receiving calls, sms.

<p align="center">
  <img src="https://i.imgur.com/PFYE65e.png" alt="image"/>
</p>

Because of the special feature it has, we are so lucky to draw the timeline from call begins to the end.


## Reference
- <https://en.wikipedia.org/wiki/Short_Message_service_center>
- <https://www.kci.go.kr/kciportal/ci/sereArticleSearch/ciSereArtiView.kci?sereArticleSearchBean.artiId=ART002801155>



## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

---
title : "Phone Scam Series: Diving into the USB Modem Artifacts"
category :
  - IoT & Embedded
tag : 
  - USB Modem
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
보이스피싱 시리즈: USB Modem Artifacts

Criminal behavior always changes overtime. As stated in the [previous post](https://kyl3song.github.io/iot%20forensics/Phone-Scam-Series-VoIP-Gateway-Forensics/), starting from late 2021, there's been a huge change that the scammers have started using USB modems for the crime.

<p align="center">
  <img src="https://i.imgur.com/Z7OWlB0.png" alt="image"/>
</p>

The reason why they use usb modem is relatively simple. USB modems are a lot cheaper, smaller, and they are more common devices to use compared to VoIP Gateways. On top of that, scammers can buy whatever amount of modems they want which is not very suspicious behavior at all.

That being said, this doesn't mean the VoIP Gateways are no loger use. Some of them are perfect for the crime. They even provide special feature to change the IMEI of each cellular/LTE module, which makes the devices difficult to be identified.



## Case Study: USB Modem

### Specs & Internals
USB modems always go with laptops & usb hubs as scammers make as much call transactions as possible. So we're going to investigate the data left after using the modem. I did purchase the model named Huawei E303 for a little more testing. Here are the frontside and backside of internal photos that I've taken after losing the case below, followed by the specifications.

<p align="center">
  <img src="https://i.imgur.com/4anpVJb.png" alt="image"/>
</p>

<p align="center">
  <img src="https://i.imgur.com/F7LW2I2.png" alt="image"/>
</p>

Since we're going to focus on the forensics part, we won't be playing with serial communication or any other debug connection this time. 


### APP: Mobile Partner
Huawei Mobile Partner is a piece of software to connect to the internet with Huawei modem. It supports sending SMS, MMS, making and receiving calls, and it even logs the call we missed, answered and dialed.

<p align="center">
  <img src="https://i.imgur.com/AUXOnMZ.png" alt="image"/>
</p>


### Key Artifacts

Turns out, the software leaves most artifacts in ***%ProgramData%\Mobile Partner*** directory.

<p align="center">
  <img src="https://i.imgur.com/dxOXUFk.png" alt="image"/>
</p>

The key artifacts are:

- NetInfo.dat: Statistics data
- SmsDBConnection: Sent or received SMS
- CallLogRecordDB: Call Logs
- MMSBox_*.mmb: Partial(subject line) MMS saved

---

**1. NetInfo.dat**

NetInfo is network related information. The dongle uploads & downloads data via 3G(WCDMA) network since connection, so to provide daily, monthly, yearly and total stats for billing. Data also separates the statistics by transfer types, so the whole data can be leveraged to trace of use.

<p align="center">
  <img src="https://i.imgur.com/n4wU9i9.png" alt="image"/>
</p>

---

**2. SmsDBConnection**

SmsDBConnection, a SQLite database, has a wealth of information. Following each field is what I figured out after testing.

<p align="center">
  <img src="https://i.imgur.com/Seqf63x.png" alt="image"/>
</p>

- TIME: String format datetime recorded in local timezone.
- ISREAD: 0(read), 1(not read)
- POSITION: 0(outbox), 1(inbox), 2(draft), 3(important), 4(deleted)
- SENDRESULT: This column is only valid for messages in the inbox
- INITIALPOSITION: POSITION before moved

The interesting part is the software **records its initial position before moved to trash** so we could track down the scammers behavior. So "reply sms from USB Modem" message is currently deleted, but was in the outbox before.

---

**3. CallLogRecordDB**

This database also records call type, number, start/end time information.

<p align="center">
  <img src="https://i.imgur.com/VG4jZT0.png" alt="image"/>
</p>

- Type: 0(answered call), 1(missed call), 2(dialed call)
- Unread: Only valid for missed call records. 0(read), 1(not read yet)
- StartTime, EndTime: Recorded in unixtime(sec)
- CallType: Only valid for dialed call records. 0(audio call), 1(video call)

How do we get the call duration? As the basic math, EndTime - StartTime gives us the result.

---

**4. MMSBox_0.mmb**

MMSBox_0.mmb contains a first few bytes of MMS content which is the subject line that is automatically downloaded when we receive a MMS, if a sender use the subject line for MMS. From the binary file, we manage to spot the various values like unixtime of each message, caller number, its length, content subject, and etc.

<p align="center">
  <img src="https://i.imgur.com/Pbp1sdJ.png" alt="image"/>
</p>

We're not even able to get the first few bytes of MMS if the MMS Server IP, Port, MMS Center URL are misconfigured when using these modems. I tried to manually configured MMS service provided by the mobile carrier's website in order to receive the MMS while using usb modem, it did not work to get the full content of message.

Given the fact that I got the MMS push message alerting new MMS is arrived, but not be able to MMS content itself, suggests that any one of the three config are wrong or it's because of telco's some policy to restrict MMS service from non-mobile-phone devices.

## Wrap-up

The limitation of using the Mobile Partner is we only could use one usb modem at a time. Scammers could've realized they need a better way to use multiple modems, so the artifacts above haven't been discovered in recent few months. That Ubuntu virtual machine with Asterisk comes in. Asterisk is an open source framework for building communications applications sponsored by Sangoma.

As commented in the first part of this post, criminal behavior constantly changes overtime.

## Reference
- <https://www.asterisk.org/>




## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

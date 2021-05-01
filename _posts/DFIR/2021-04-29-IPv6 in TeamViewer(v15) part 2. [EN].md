--- 

title : "Blog #28: IPv6 in TeamViewer(v15) part 2. [EN]"
category : 
  - Artifacts
tag :  
  - DFIR
  - Digital Forensics & Incident Response
  - TeamViewer
  - IPv6
sidebar_main : true 
author_profile : true 
use_math : False
toc: true 
toc_sticky: true 
toc_label: "Table of Contents" 
header: 
  overlay_image : /assets/images/post.jpg 
  overlay_filter: 0.5 
#published : true 

--- 

IPv6 in Logs with Bits and Pieces

## This Post Covers
In the previous post, we covered the basics of Teamviewer log analysis and confirmed that we have a lot more to delibrate from the basic log, 'connections_incoming.txt'. We have more information coming right up, tips & details including IPv6 in part 2.

## TeamViewer##_Logfile.log (LocalTime)
Unlike the 'connections_incoming.txt', 'TeamViewer##_Logfile.log' has more rich information about remote access. The two digits '##' followed by '_Logfile.log' indicates software version. TeamViewer##_Logfile.log is applied in LocalTime, UTC+9 in here South Korea.

Two log files with the exact same name are saved in different directories. Although they both have the same name, contents are distinct.

**1. %USERPROFILE%\AppData\Roaming\TeamViewer\TeamViewer##_Logfile.log**  
➔ Records general information of software operation.

<p align="center"> 
<img src="https://i.imgur.com/zaz6Ypq.png" alt="image"/> 
<br>[ Fig 1. %USERPROFILE%\AppData\Roaming\TeamViewer ]</p>

<p align="center"> 
<img src="https://i.imgur.com/xJj9b7r.png" alt="image"/> 
<br>[ Fig 2. Content of TeamViewer15.log ]</p>

**2. C:\Program Files(x86)\TeamViewer\TeamViewer##_Logfile.log (install path)**  
➔ Records detailed information of remote access & connections.

<p align="center"> 
<img src="https://i.imgur.com/O4bNFHc.png" alt="image"/> 
<br>[ Fig 3. Remote Access Log ]</p>

All right, We have two logs illustrated in Fig 3. above, filename is containing version numbers, 14 or 15. This tells us the program was upgraded from 14 to 15 at some point in the past. That aside, the file is automatically separated with the name of 'TeamViewer##_OLD.log' as soon as the size of original is over 1MB more or less.

## Log Analysis Flow & Tips

Let's assume that remote access starts from Device A all the way to Device B.

<p align="center"> 
<img src="https://i.imgur.com/Spf9pfh.png" alt="image"/> 
</p>


### 1. Session Start & Encryption Negotiation
As a start point, log begins with **Activating Router carrier**, followed by encryption negotiation process where using AES-256 encryption algorithm for a symmetric key and RSA for key exchange.

<p align="center"> 
<img src="https://i.imgur.com/KG0TJDk.png" alt="image"/> 
<br>[ Fig 4. Teamviewer Connection Log on Device B ]</p>


### 2. Fingerprint
If we look at the box highlighted in Fig 4 above, local and remote fingerprints are hashed(SHA256) repectively. Like the TeamViewer ID, each device has a unique fingerprint. The fingerprint is generated on the local TeamViewer client by the machines public key and consists of letters, numbers, and special characters.

<p align="center"> 
<img src="https://i.imgur.com/rW4x6wC.png" alt="image"/> 
<br>[ Fig 5. Local Fingerprint of Device B ]</p>

We know Device A(Remote) and Device B(Local)'s fingerprints from the log that makes us have another chance to match suspect's fingerprint if we find a PC on the crime scene.


### 3. Participants
Easy way to distinguish whether a portion of logs belong to Device A or B, is just use the role type number.
Support PC(Device A) always has **type 6 and role 6** whereas, client PC(Device B) has **type 3 and role 3.** If this doesn't ring a bell in the future, remember attacker gets the upperhand, gets stronger position.

- type 6, role 6: Support PC(Device A)
- type 3, role 3: Client PC(Device B)

<p align="center"> 
<img src="https://i.imgur.com/w0Ycekd.png" alt="image"/> 
</p>

<p align="center"> 
<img src="https://i.imgur.com/wg0LDsn.png" alt="image"/> 
<br>[ Fig 6. Log type & role by Participants ]</p>


### 4. IPv4 (Public vs. Private)
In the network world NAT does always matter, specifically for P2P(peer to Peer) communication. Device A and B does not know each other's public IP, private IP and even so, they cannot directly communicate without NAT Table.

NAT Table in home environment? So we need an intermediate node(Server) that works NAT Translation for each party. We call this technique, UDP hole punching. TeamViewer Server(e.g. KR-SEL-ANX-R019.teamviewer.com in Fig 4.) in the middle, between Deivce A and B works for us. That's why Device A is able to access Device B without any network device in home environment.

> UDP hole punching is a commonly used technique employed in network address translation (NAT) applications for maintaining User Datagram Protocol (UDP) packet streams that traverse the NAT.

- punch received a=***Public IP:Port***
- punch ignored a=***Public IP:Port***


<p align="center"> 
<img src="https://i.imgur.com/9oDIOy5.png" alt="image"/> 
<br>[ Fig 7. Public IP Address of Device A ]</p>

If Device A and Device B are in the same network(private), they don't need to use public IP for communication, **they use private IP instead.** That's why sometimes we spot private IP from the logs in a test environment.

- punch received a=***Private IP:Port***

<p align="center"> 
<img src="https://i.imgur.com/n5Ew8jr.png" alt="image"/> 
<br>[ Fig 8. Private IP Address of Device A ]</p>


### 5. IPv6 (Public)
I haven't seen any IPv6 in Teamviewer log until I used a couple of mobile phones for testing. To check if you have an IPv6 assigned by service provider, go to website at [http://test-ipv6.com](http://test-ipv6.com)

<p align="center"> 
<img src="https://i.imgur.com/8MDN8Ah.png" alt="image"/> 
<br>[ Fig 9. IPv6 log ]</p>


That being said, not every phone will create IPv6 log. As far as I'm concerned from the test, it's matter of your service provider directly gives your device IPv6 so that it gets ready to communicate via IPv6. Just so you know about the service plan, mobile phones used in test all use LTE data plan except one, Galaxy S10 5G.


|No.|Test Mobile|Model|5G Support|Service Provider<br>(carrier)|Data Plan|in Log|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|1    |iPhone 8           |A1906    |No  |LG U+|LTE|IPv4|
|2    |iPhone X           |A1901    |No  |SKT  |LTE      |IPv4|
|3    |LG VEVET           |LM-G910N |No  |KT   |LTE      |IPv4|
|4    |iPhone 12          |A2403    |Yes |LG U+|LTE      |IPv4|
|5    |Galaxy S10 5G      |SM-G977N |Yes |SKT  |5G       |**IPv6**|
|6    |Galaxy A Quantum   |SM-A716S |Yes |SKT  |LTE      |**IPv6**|

Search keyword to filter out IPv6 would be as follows:
- EmergingUdpConnection::AsyncSendTo::Handler Send error system:1231 to ***Public IPv6:port***

<p align="center"> 
<img src="https://i.imgur.com/aG5Oy3c.png" alt="image"/> 
</p>

Galaxy series leave **null** for Device A's display name in the Connections_incomings.txt by default, but we could find **model name** in TeamViewer15_Logfile.log.

<p align="center"> 
<img src="https://i.imgur.com/6ojvzF7.png" alt="image"/> 
</p>


### 6. Update Log
TeamViewer also leaves software update logs including last update time(in unixtime) as well as version, temp directory for a installation binary to save in. In my test environment, username "incheon" could be another evidence factor that should not be overlooked.

<p align="center"> 
<img src="https://i.imgur.com/Fd9NEyO.png" alt="image"/> 
<br>[ Fig 10. Tracking Software Update ]</p>



### 7. Session End
Remote access ends with the keyword 'RemoveParticipants' and RA terminates all related services.

<p align="center"> 
<img src="https://i.imgur.com/ZRy5ntz.png" alt="image"/> 
<br>[ Fig 11. Remote Session End ]</p>


## Wrap-up
Unfortunately in this post series we only focused on the TeamViewer logs. However, we need to walk through other teamviewer artifacts like in %USERPROFILE%\Appdata\local\Teamviewer folder and other artifacts as well, registry, event logs and so on.

The more you know, the greater the chance for a good judgement.

## Reference 
- <https://en.wikipedia.org/wiki/UDP_hole_punching>
- <https://www.netmanias.com/ko/post/blog/6263/nat-network-protocol-p2p/p2p-nat-nat-traversal-technic-rfc-5128-part-2-udp-hole-punching>
- <https://community.teamviewer.com/English/kb/articles/102142-what-is-the-teamviewer-fingerprint>
- <https://cjwoov.tistory.com/6?category=774541>
- <http://test-ipv6.com>


## Copyright (CC BY-NC 2.0)

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%"> 

- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
- <http://ccl.cckorea.org/about/>
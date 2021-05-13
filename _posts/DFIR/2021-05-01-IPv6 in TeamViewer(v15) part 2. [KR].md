--- 

title : "Blog #28: IPv6 in TeamViewer(v15) part 2. [KR]"
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
published : true

--- 

팀뷰어 로그에서의 IPv6

## TeamViewer Forensic Series
- [Blog #27: IPv6 in TeamViewer(v15) part 1. [KR]](https://kyl3song.github.io/artifacts/IPv6-in-TeamViewer(v15)-part-1/)
- [Blog #28: IPv6 in TeamViewer(v15) part 1. [KR]](https://kyl3song.github.io/artifacts/IPv6-in-TeamViewer(v15)-part-2.-KR/)

## This Post Covers
이전 글에서 팀뷰어 기본적인 연결 로그인 'connections_incoming.txt'의 해석과 단순한 로그더라도 나름 생각해 볼 가치가 있는 부분도 확인하였다. 이번 글에서는 실제 상세 로그파일에 대한 분석 팁과 내용에 대해 정리를 할 예정이다.

## TeamViewer##_Logfile.log (LocalTime)
'connections_incoming.txt' 로그와 다르게 'TeamViewer##_Logfile.log' 로그는 원격 연결에 대한 더 상세한 정보를 담고 있다. 파일명에 기재된 두 자리 숫자(##)는 팀뷰어 소프트웨어에 대한 버전을 의미하는데, 조금 더 상세하게 말하면 버전 중 가장 상위인 메이저 숫자를 말한다.

> **Version 정보(X.Y.Z.)**  
> Major Version . Minor Version . Build or Maintenance Version(patch) 형태로 구성된다.

'TeamViewer##_Logfile.log' 로그 역시 시간을 기록하는데 시간은 로컬 타임으로 보정되어 로그가 기록된다. 즉 우리나라의 경우 시스템 시간에 맞게 UTC+9로 기록이 된다. 해당 로그는 동일한 이름으로 2개가 존재하는데 로그의 내용도 다르고 저장되는 경로도 다르다.

**1. %USERPROFILE%\AppData\Roaming\TeamViewer\TeamViewer##_Logfile.log**  
➔ 소프트웨어 동작에 대한 전반적인 로그를 기록

<p align="center"> 
<img src="https://i.imgur.com/zaz6Ypq.png" alt="image"/> 
<br>[ Fig 1. %USERPROFILE%\AppData\Roaming\TeamViewer ]</p>

<p align="center"> 
<img src="https://i.imgur.com/xJj9b7r.png" alt="image"/> 
<br>[ Fig 2. Content of TeamViewer15.log ]</p>

**2. C:\Program Files(x86)\TeamViewer\TeamViewer##_Logfile.log (install path)**  
➔ 원격 접속 및 연결에 대한 상세로그

<p align="center"> 
<img src="https://i.imgur.com/O4bNFHc.png" alt="image"/> 
<br>[ Fig 3. Remote Access Log ]</p>

Fig 3.에 두 가지 메이저 버전에 대한 로그가 보이는데 이는 버전이 14에서 15로 업그레이드된 것을 의미한다. 그리고 로그가 약 1MB 이상 되면 기존 로그 파일이 _OLD 이름으로 자동 아카이빙 되고 새로운 로그를 기록한다.

## Log Analysis Flow & Tips

아래 디바이스 A에서 B로 원격 접속을 가정하고 로그가 어떻게 남는지 살펴보도록 하자.

<p align="center"> 
<img src="https://i.imgur.com/Spf9pfh.png" alt="image"/> 
</p>


### 1. Session Start & Encryption Negotiation
원격 세션이 시작될 때 로그는 **Activating Router carrier** 문구로 시작된다. 그리고 팀뷰어 중개서버를 연결하고 노드 간 암호화 통신을 위한 RSA 키 협상 절차가 진행된다. 실제 A와 B는 AES-256 대칭키로 암호화 통신을 한다. 더 자세한 내용은 [여기](https://www.teamviewer.com/ko/trust-center/security/)에서 확인할 수 있다.


<p align="center"> 
<img src="https://i.imgur.com/KG0TJDk.png" alt="image"/> 
<br>[ Fig 4. Teamviewer Connection Log on Device B ]</p>


### 2. Fingerprint
협상이 끝나면 SHA256으로 해시화한 fingerprint를 교환하는데, 9~10자리 숫자로 구성된 Teamvier ID와 같이 fingerprint 값도 기기마다 모두 다르다. 사용하는 이유는 원격 대상이 진짜 그 상대 기기가 맞는지 확인 및 검증하기 위해서 사용된다.

숫자로 구성된 팀뷰어 아이디와 비밀번호만 있으면 원격 접속할 수 있기 때문에 이런 허점이 보완하고자 Fingerprint를 이용하여 원격 기기와 상대방에 대한 검증 차원에서 필요한 기술이라고 볼 수 있다.

<p align="center"> 
<img src="https://i.imgur.com/rW4x6wC.png" alt="image"/> 
<br>[ Fig 5. Local Fingerprint of Device B ]</p>

Fig 4.처럼 로컬 및 리모트 fingerprint 값이 로그에 남게 되는데 피해자 PC에서 남은 리모트 값과 피의자로 의심되는 사람의 압수된 PC에서 확인된 로컬 값이 연결고리로 작용할 수 있으니 잘 살펴봐야 한다.


### 3. Participants
로그를 보다 보면 수많은 로그 중 어느 것이 Device A와 관련된 정보이고 어느게 Device B에 대한 정보인지 헷갈릴 수 있다. 이럴 때 type 번호와 role 번호를 기억하면 쉽게 식별할 수 있다.

Device A(Support PC)의 경우 항상 **type 6 and role 6**로 로그가 기록되고 Device B(Client PC)의 경우 **type 3 and role 3**로 남는다. 만일 나중에 팀뷰어 로그를 분석할 때 오히려 이 두개의 숫자가 헷갈린다면 쉽게 찾을 수 있는 방법은 ***"공격자는 항상 사건에 있어 우위를 점한다. 따라서 번호가 높다."***라고 기억하면 한방에 확 와닿을 것이다.

- type 6, role 6: Support PC(Device A)
- type 3, role 3: Client PC(Device B)

<p align="center"> 
<img src="https://i.imgur.com/w0Ycekd.png" alt="image"/> 
</p>

<p align="center"> 
<img src="https://i.imgur.com/wg0LDsn.png" alt="image"/> 
<br>[ Fig 6. Log type & role by Participants ]</p>


### 4. IPv4 (Public vs. Private)
네트워크 환경, 특히 P2P(Peer-to-Peer) 통신에서 항상 문제가 되는 부분은 NAT 부분일 것이다. Device A와 B가 통신할 때 처음부터 상대방의 공인 IP, 사설 IP를 아는 것도 아니고, 만일 알지라도 통신을 위한 포트포워딩 설정 또는 NAT 테이블 관리를 해주는 장비가 없이는 상호 간 직접 통신하지 못한다.

개인 PC 환경에서 네트워크 장비나 공유기의 설정을 해야 한다면 그건 고도의 IT 지식을 요구하기 때문에 일반적인 사용자들은 사용하기 어려울 것이다. 그렇기 때문에 이를 대신해 줄 수 있는 무언가가 필요하다.

팀뷰어에서는 중계서버가 A와 B간 통신이 이뤄질 때 NAT 테이블을 관리하는 역할을 대신하여 해주는데 이런 기법을 'UDP hole punching' 이라고 부른다.

> UDP hole punching is a commonly used technique employed in network address translation (NAT) applications for maintaining User Datagram Protocol (UDP) packet streams that traverse the NAT.

로그에서도 보면 **UDPv4: punch** 등과 같은 키워드로 로그를 남기고 있다. 해당 로그를 통해 상대방의 공인 IP 주소를 알 수 있다.

- punch received a=***Public IP:Port***
- punch ignored a=***Public IP:Port***


<p align="center"> 
<img src="https://i.imgur.com/9oDIOy5.png" alt="image"/> 
<br>[ Fig 7. Public IP Address of Device A ]</p>

한 가지 중요한 건 원격할 때마다 상대방의 공인 IP 정보가 100% 남지는 않는다는 것이다. 나도 사실 이 문제에 대해 왜 어떤 경우에는 나오지 않을까라고 고민을 했었고, 생각해 본 결과 아마 오랫동안 서로 간 통신이 없거나 프로그램 종료 등 사유로 NAT 테이블 정보가 서버에서 삭제된 이후 원격 시도를 하면 다시 그 정보가 처음부터 생성되어 로그가 남고, 그렇지 않은 경우 기존에 정보가 있기 때문에 로그를 남기지 않을까 하는 개인적인 추정이다.

테스트 시 만일 A와 B가 동일한 사설 네트워크인 경우에는 중계서버에서 통신을 공인 IP로 통신할 필요없이 서로의 사설 주소로 통신하는 Internal Network Hole-Punching 유형이 적용되어 **상호간 사설 IP로 통신**을 한다. ➜ [Hole Punching 기술 유형](http://blog.skby.net/hole-punching/)

따라서 팀뷰어 로그에서 아래와 같이 사설 아이피 대역이 나오는 이유이다.

- punch received a=***Private IP:Port***

<p align="center"> 
<img src="https://i.imgur.com/n5Ew8jr.png" alt="image"/> 
<br>[ Fig 8. Private IP Address of Device A ]</p>


### 5. IPv6 (Public)
지금까지 로그에서 IPv6 관련된 로그를 본 적이 없었다. 아마 있었어도 자세히 봐야겠다는 인식 자체가 없었을지도 모른다. 최근에 모바일로 연결 실험을 하다 IPv6 정보를 면밀히 관찰하였다.

모바일 기기에서 공인 IPv6를 확인하는 방법은 [http://test-ipv6.com](http://test-ipv6.com) 사이트에 접속하면 할당된 공인 IPv6를 확인할 수 있는데 무조건 다 나오는 것은 아니고 나오지 않은 경우가 좀 더 많은 것 같다.

로그에서 IPv6와 Device A의 할당된 공인 IPv6가 일치함을 확인하였고 앞으로는 해당 정보를 가지고 추적의 단서로 사용하면 될 것이다.

<p align="center"> 
<img src="https://i.imgur.com/8MDN8Ah.png" alt="image"/> 
<br>[ Fig 9. IPv6 log ]</p>

위에서 언급했듯이 모든 휴대폰에서 팀뷰어로 원격 시도를 했을 때 IPv6 로그가 생성되는 것은 아니다. 실험을 통해 확인한 부분은 사업자 측에서 해당 기기를 IPv6로 직접 할당하고 버전 6를 통해 직접 통신을 할 수 있냐라는 부분으로 판단할 수 있을 것 같다.

내 휴대폰 및 테스트 폰, 회사 동료들의 휴대폰을 거의 반강제적(?)으로 동원시켜 총 6종류를 확인했고, 요금제는 Galaxy S10 5G 모델만 5G 요금제를 사용하고 나머지는 모두 LTE(4G) 요금제를 사용 중이었다.

결과는 다음 표와 같았다. 요약하면 기기가 5G를 지원해야 했고, 요금제에 상관없이 현재 SKT 통신사를 이용하는 기기에서만 IPv6가 확인되었다.

|No.|Test Mobile|Model|5G Support|Service Provider<br>(carrier)|Data Plan|in Log|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|1    |iPhone 8           |A1906    |No  |LG U+|LTE|IPv4|
|2    |iPhone X           |A1901    |No  |SKT  |LTE      |IPv4|
|3    |LG VEVET           |LM-G910N |No  |KT   |LTE      |IPv4|
|4    |iPhone 12          |A2403    |Yes |LG U+|LTE      |IPv4|
|5    |Galaxy S10 5G      |SM-G977N |Yes |SKT  |5G       |**IPv6**|
|6    |Galaxy A Quantum   |SM-A716S |Yes |SKT  |LTE      |**IPv6**|

앞으로는 5G를 지원하는 모바일이 확대될 것이고 그에 맞춰 통신사도 기술 및 통신 정책들도 계속 변할 예정이기에 IPv6에 대한 로그는 점점 많이 보일 것으로 예상된다.

현재 IPv6를 필터링을 위한 키워드 문구는 다음과 같다. (정규표현식 사용 무방)
- EmergingUdpConnection::AsyncSendTo::Handler Send error system:1231 to ***Public IPv6:port***

<p align="center"> 
<img src="https://i.imgur.com/aG5Oy3c.png" alt="image"/> 
</p>

아이폰의 경우 휴대폰 기기에 설정된(일반 > 정보 > 이름) 아이폰 명이 기록되는 반면 2개의 갤럭시 휴대폰으로 원격 접속을 했을 때, Connections_imcoming.txt 로그에서 디폴트로 Display Name이 **null**로 표시된다. 만일 팀뷰어 계정으로 로그인을 한 상태에서 원격 접속을 했다면 로그인 계정이 남을 것이다.

또한, Display Name이 null로 나왔어도 상세 로그에서는 갤럭시 기기 모델명을 확인할 수 있어 해외향으로만 나온 모델인지, 국내향인 모델인지도 추가 검색이 가능할 수 있다.

<p align="center"> 
<img src="https://i.imgur.com/6ojvzF7.png" alt="image"/> 
</p>


### 6. Update Log
팀뷰어 로그에서는 소프트웨어 업데이트 로그도 남기는데 업데이트 버전 정보, 시간 정보, 업데이트 파일 저장을 위한 임시 디렉터리 등 정보를 남긴다.
테스트 환경에서 확인된 임시 디렉터리 경로에는 "incheon"과 같이 윈도우 사용자명이 포함되어 있는데 이는 또 다른 증거 요소로 사용될 수 있으니 간과하지 말아야 할 부분이다.

<p align="center"> 
<img src="https://i.imgur.com/Fd9NEyO.png" alt="image"/> 
<br>[ Fig 10. Tracking Software Update ]</p>



### 7. Session End
원격 세션 종료 시 'RemoveParticipants' 키워드의 로그를 시작으로 관련 서비스, 쓰레드를 종료하는 로그가 남는다.

<p align="center"> 
<img src="https://i.imgur.com/ZRy5ntz.png" alt="image"/> 
<br>[ Fig 11. Remote Session End ]</p>


## Wrap-up
솔직히 말하면 팀뷰어 로그를 별거 아닌 것처럼 생각하는 사람이 많다. "그거 뭐 그냥 보면 다 보이는 거잖아"라고 말이다. 하지만 팀뷰어 로그 하나만 제대로 파보면 사용자명, fingerprint, 업데이트 기록, 공인 IPv6 정보 등 사람과 연결 지을 수 있는 요소가 너무나 많이 관찰된다.

특히 UDP Hole Punching 같이 Fundamental 적인 부분을 잘 알지 못하면 로그 해석에 어려움이 있을 수 있기 때문에 모르면 계속 찾아가며 따라가야 앞으로도 잘 이해할 수 있다.

지금까지 Teamviewer 포렌식 포스트 시리즈에서는 안타깝게 팀뷰어 로그에 대해서만 살펴보았다. 하지만 살펴본 로그뿐 아니라 **%USERPROFILE%\Appdata\local\Teamviewer** 폴더 전체를 살펴볼 필요가 있고 레지스트리, 이벤트 로그 등 다른 아티팩트와 연결고리가 있는지 종합적으로 정리하면 분석에 더욱 넓은 뷰를 가질 수 있을 것이다.

**"The more you know, the greater the chance for a good judgement."**


## Reference 
- <http://seorenn.blogspot.com/2012/02/version.html>
- <https://en.wikipedia.org/wiki/UDP_hole_punching>
- <https://www.netmanias.com/ko/post/blog/6263/nat-network-protocol-p2p/p2p-nat-nat-traversal-technic-rfc-5128-part-2-udp-hole-punching>
- <http://blog.skby.net/hole-punching/>
- <https://community.teamviewer.com/English/kb/articles/102142-what-is-the-teamviewer-fingerprint>
- <https://cjwoov.tistory.com/6?category=774541>
- <http://test-ipv6.com>
- <https://www.teamviewer.com/ko/trust-center/security/>


## Copyright (CC BY-NC 2.0)

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%"> 

- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
- <http://ccl.cckorea.org/about/>

---
title : "Blog #21: Tips for Testifying in Court part 1"
category :
  - Law & Legal process
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - Sworn Testimony
  - Witness Testify
  - Subpeona
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
법정 증언 및 증언 시 유의점 part 1


## Tips for Testifying in Court Series
- [Blog #21: Tips for Testifying in Court part 1](https://kyl3song.github.io/law%20and%20legal%20process/Tips-for-Testifying-in-Court-part1/)
- [Blog #22: Tips for Testifying in Court part 2](https://kyl3song.github.io/law%20&%20legal%20process/Tips-for-Testifying-in-Court-part2/)


## Preface
법원으로부터 증인으로 소환되게 되면 특별한 사유가 있지 않는 이상 출석해야 한다.
만일 참석하지 못할 사유가 있다면 미리 이를 법원에 사유서를 제출하여 출석 기일을 연장해야 한다.

이를 어기고 무단으로 불참했을 시 500만원 이하의 과태료가 부가될 수 있고 필요에 따라 강제구인될 수도 있다. 즉, 강제로 끌려간다는 이야기이다.

오늘은 최근에 법정에 증인으로 출석하여 증언을 한 경험을 이야기하려고 한다.


## Witness Subpoena (증인 소환장)
증인으로 소환되게 되면 법원으로부터 증인 소환장을 우편을 통해 받는다. 증인 소환장에는 사건번호, 피고인 이름, 출석 일시, 장소 등이 기재되고 출석해달라는 요청과 함께 판사의 직인이 찍혀있다.

<p align="center">
  <img src="https://i.imgur.com/WicvmNE.png" alt="image"/>
<br>[ Witness Subpoena ]</p>

그리고 뒷장에는 열람복사 제한신청이 안내되어 있었다.

<p align="center">
  <img src="https://i.imgur.com/xu0J2XB.png" alt="image"/>
<br>[ Witness Subpoena ]</p>

원래 동일 사건 관련하여 소환장을 내가 최초로 받은 것은 아니었다. 한 달 전쯤 나와 같이 증거물을 배당 받은 A분석관이 우편을 처음 받았고, A분석관이 내 자리 바로 뒤에 앉아 있기 때문에 당시 상황을 기억할 수 있었다.

A분석관은 갑자기 검사한테 연락이 와서 노트북 분석 여부를 묻더니 소환장은 잘못 보낸 것이니 신경쓰지 말라고 통화를 끝냈다고 한다.

그 얘기를 나한테 해줬는데, 내가 혹시 그거 누구랑 같이 배당받은 사건이냐고 물어봤고 추적을 해보니.. 나와 같이 배당을 받은 사건이었다.

접수단계에서 증거물이 많을 경우 한 사람에게 몰아서 배당하는 것이 아닌 일정 부분을 나눠서 배당하게 된다. 한 사람이 모두 받는 것보다 여러 명이 나눠서 분석하면 분석 시간도 짧아지고, 효율도 높아지기 때문이다.

아무튼 내가 혹시 나한테 우편 보낼걸 잘못 보낸거 아니냐고 농담 식으로 얘기했던 게 진짜였다. 불길한 예감은 언제나 틀림이 없었다. 나였다.

며칠 뒤에 검사실로부터 연락을 받았고, 증거분석한 거 때문에 법정에 증인으로 출석해 주셔야 할 것 같다며 일정을 알려주었다. 그리고 자세한 내용은 검사를 통해서 확인이 가능하다고 하였다.

그 다음날 검사한테 전화를 받았고 확인해보니 변호인 측에서 내가 작성한 **증거분석 결과보고서 자체를 신뢰할 수 없다는 사유**였다.

이게 보고서의 어떤 부분으로 다투고 있다는 게 명확하면 좋을 텐데 그렇지 않았다. 아니면 변호인의 전술일 수도 있다.

담당 검사랑 이런저런 얘기를 한 뒤에 통화를 마치고 사건에 대해 찾아봤다. 분석 보고서, 의뢰 공문 등 기억을 더듬기 위해 관련된 건 다 찾아봤다.


## Phone Scams Case (보이스 피싱 사건)
시간을 거슬러 2020. 7.경 사기 사건으로 증거를 분석한 건이었다.

보이스 피싱 사기로 편취한 범죄 수익금을 비트코인으로 자금 세탁까지 한 혐의 관련한 내용이었다. 규모는 **약 300억대** 정도되는 나름 큰 규모의 범죄였다.

300억대의 범죄수익이라면 고급 변호사를 붙였을 시나리오는 거의 100퍼센트다. 그리고 어떻게든 빠져나갈 구멍을 찾거나 양형을 받을 생각을 하는 놈이라는 생각에 일단 없던 정의감도 불타오른다.

증거물은 맥북 1대, 아이폰 3대가 접수되었고 나는 아이폰 3대(+USIM)를 배당을 받았다. 담당 수사관의 요청에 따라 보고서를 작성하고 결과물을 전달한 특별할 것 없는 일반적인 보고서였다.


## Going to Court as an Investigator
예전에 사이버팀에서 직접 수사를 할 때는 피의자 영장실질심사로 법정에 자주 들락날락했었다.

영장실질심사는 검사로부터 구속영장을 청구 받은 판사가 피의자를 직접 심문해 구속 여부를 결정하는 제도이다.

그때 수사관들은 보통 배심원석에 앉아있는데 보면 가관도 아니다. 일단 많은 피의자 들이 법정에서 운다. 그리고 어떻게든 범죄와 연관된지 몰랐다고 변명하거나 앞으로 성실하게 잘 살겠다고 최후 발언을 한다. 그러다 실질이 끝나면 언제 그랬냐는 듯이 웃고 떠들고.. 그리고 나중에 또 잡혀서 실질심사에 오고 반복이다.

가끔 담당 판사가 사건 관련해서 담당 수사관에게 질문을 하는 경우가 있는데 내 경우에는 1~2번의 경험이 있었고 많지는 않았다.

그래서 그런지 법정에 들어가는 것에 대한 거부감은 크게 없었다. 그렇지만 내가 직접 증인의 신분으로 발언을 하거나 질문을 받아본 적은 지금까지 없었고 무슨 질문이 들어올지는 전혀 알지 못했다. 특히 분석관의 입장에서 말이다.


## Getting Ready for Court as a Witness
법정 증언을 하기 위해 무엇을 준비해야 될까? 아니 내가 궁금했던 건 무슨 질문들이 들어올까가 최대 관심사였다. 주위의 법정 증언 다녀온 여러 사람들에게 물어봤고 들어보니 질문들은 크게 세 부류로 나뉘었다. 

### General Case
먼저 공통적인 기본 질문, 절차 관련된 질문, 그리고 보고서에 기재된 용어 또는 내용 중심의 질문류였다.

- 소속
- 이름
- 증거 분석관으로 근무한 기간(경력)
- 디지털포렌식 관련 학위 등
- 소유하고 있는 자격증
- 관련 학회에 논문 등 투고한 경험
- 수상 이력
- 증거물 봉인에 관한 질문
- 봉인 해제 시 어떻게 진행했는가 (그리고 근거자료가 있는지 여부)
- 획득은 어떻게 진행했는지
- 획득 후 분석 시 해시값 검증을 했는지 여부
- 분석은 어떻게 진행했는지
- 분석 결과에 대한 해시값 생성 관련 질문
- 분석 보고서에 기재된 용어 또는 기술한 내용 관련된 질문
- 보고서에 기재된 용어가 무엇인지?  
 \- WGS84 등 보고서에 작성된 기술 용어  
 \- 보고서에 기재한 도구 및 기능에 대한 설명  
 \- 맥주소와 아이피 주소에 대한 설명과 둘 간의 차이 등  

### Little More Thinking is Required
그리고 조금 더 생각을 요구하는 질문도 있었다. 자신이 당사자라면 어떻게 대응할지 한 번쯤 생각해 보는 것도 도움이 될 것이다.

- 본인의 증거분석으로 추출된 텍스트 파일을 수사관이 엑셀 형태로 정리했는데 두 파일이 동일한 가를 어떻게 입증할 수 있는가?
- 본인이 그것을 검증해 줄 수 있는가?

이 두 가지의 질문에서 당시 당사자는 이렇게 대답을 했다고 한다.

파일 형태 자체가 다르기 때문에 두 개의 파일의 동일한지 지금 알 수 없다. 입증하려면 두 개의 파일의 문자열 단위로 하나하나 동일한지 비교해야 한다.

검증은 결과 분석한 사람이 하는 것이 아닌 신뢰된 사람이 하는 게 맞다고 본다. 내가 할 일이 아니라고 생각한다.

### Malware Analysis Case
이번엔 악성코드 분석 건에 관한 질문이다.

- 악성코드를 분석했는데 악성코드의 행위를 재현해 줄 수 있는가?

이 질문에서 당사자는 이렇게 대답을 했다고 한다.

차로 치면 차 설계도를 보고 그 차가 어떤 기능을 가지고 있다고 분석한 것이지 그 차를 당시 환경과 동일한 상태에서 직접 동작해본 것은 아니다. 범행 당시 환경과 현재의 환경도 다를뿐더러 악성코드의 일부분을 가지고 분석한 부분이어서 재현할 수 없다.

내가 처음 들은 느낌은 뭔가 명쾌하나 저렇게 법정에서 딱 잘라서 말해도 되나 싶을 정도였다.

이처럼 질문은 생각보다 파생될 게 많았다.

### Tips for Success in the Courtroom
질문에 대한 대답을 준비하는 만큼 더 중요한 것들이 있다. 내가 법정 증언에 대한 대비를 하면서 생각해 본 것들이다.

**1. 애매한 증언 금물**

질문에 모르거나 명확하지 않는 부분은 잘 모른다. 기억이 나지 않는 것은 말 그대로 기억이 잘 나지 않는다로 대답하면 된다. 애매한 것인데 넘겨짚지 말라는 의미이다.

**2. 가급적 순화된 용어를 사용**

법정에서 설명할 때 가급적 테크니컬 한 용어를 사용하지 말고 IT 전문가가 아니더라도 잘 알아들을 수 있게 순화된 용어를 사용하고, 가능하다면 위의 악성코드 질문에 대한 대답처럼 비유를 들어서 설명하는 것도 효과적인 방법이다.

**3. 용어 정리**

평소에 용어에 대한 정의를 계속 외우고 있지는 않을 것이다. 몇 개는 개념만 알고 몇몇은 아예 까먹은 것도 있을 수 있다.

이럴 때 법정에서 용어에 대해 설명하다가 버벅댈 수 있는데 반대신문에서는 이런 부분을 이용하여 분석관의 자격을 깎아내리고 증거를 탄핵하려고 노력한다.

한 번씩 용어 정리를 하는 것이 필요하다. 특히 내가 보고서에 작성한 용어들은 충분한 습득이 필요하다.

**4. 사건에 대한 이해**

분석관의 입장에서 관련 사건에 대해 깊은 이해는 필요 없을 수 있으나 당시 담당했던 수사관과 통화하여 상황을 설명하고 이야기를 나누면 힌트가 될 만한 내용을 받을지도 모른다.

나의 경우에는 수사관은 연락을 안 했고 담당 검사와 통화만 했는데 도움이 많이 됐다.

그리고 [대법원사이트](https://scourt.go.kr/portal/information/events/search/search.jsp)에 방문하면 나의 사건을 검색할 수 있는 메뉴가 있다.

여기에서 사건 관련된 정보를 입력하면 사건이 현재 어떻게 진행되어 가고 있는지, 피고인은 총 몇 명인지, 변호인의 법무법인 이름도 확인할 수 있으니 한 번쯤 확인해보는 것도 중요한 것 같다.

<p align="center">
  <img src="https://i.imgur.com/qD0gywh.png" alt="image"/>
<br>[ 대법원사이트 > 나의 사건검색 ]</p>

<p align="center">
  <img src="https://i.imgur.com/YBb0qZL.png" alt="image"/>
<br>[ 대법원사이트 > 나의 사건검색 ]</p>


## Wrap-up
나의 경우는 어떻게 됐을까? 글이 너무 길어지는 것 같아 다음 포스트에 내가 준비했던 부분 및 실제 법정에서 진술한 내용을 포스팅할 예정이다.


## Copyright (CC BY-NC)
본 게시글은 CC BY-NC Licence를 따릅니다.  
비영리 목록으로만 사용할 수 있고, 저작자와 출처를 표시하면 언제든지 게시글을 자유롭게 사용할 수 있습니다.

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
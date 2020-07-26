---
title : "Blog #11: Windows Storage Spaces Forensics (aka. SPACEDB) part 5"
category :
  - Windows Storage Spaces
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - Storage Spaces
  - Storage Direct
  - Software RAID
  - SPACEDB
  - X-Ways Forensics 19.9
  - EnCase 8.11
  - EnCase 20.2
  - Magnet AXIOM 4.01
  - FTK Imager 4.3.0.18
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
윈도우 저장소 공간 포렌식(SPACEDB) part 5 - The Final

## Windows Storage Spaces Series
- [Blog #7: Windows Storage Spaces Forensics (aka. SPACEDB) part 1](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-1/)
- [Blog #8: Windows Storage Spaces Forensics (aka. SPACEDB) part 2](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-2/)
- [Blog #9: Windows Storage Spaces Forensics (aka. SPACEDB) part 3](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-3/)
- [Blog #10: Windows Storage Spaces Forensics (aka. SPACEDB) part 4](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-4/)
- [Blog #11: Windows Storage Spaces Forensics (aka. SPACEDB) part 5](https://kyl3song.github.io/windows%20storage%20spaces/Windows-Storage-Spaces-Forensics-(aka.-SPACEDB)-part-5/)

이번 포스트를 마지막으로 윈도우 저장소 공간 포렌식을 마무리할 예정이다. 지금까지 복원된 아티팩트를 분석하여 혐의 구증에 필요한 정보를 확인해 보도록 하겠다.

## Artifacts Analsysis
이전 포스트에서는 파일시스템을 복원하는 작업을 수행하였다. 파일시스템이 복원됐으니 세부적으로 들어가서 내부에 남아있는 또는 복원된 아티팩트를 분석하는 작업을 거쳐야 한다.

윈도우 저장소 공간 자체는 운영체제의 부팅 용도로 사용될 수 없고 데이터 저장 용도로만 사용이 가능하다. 따라서 NTFS의 로그를 저장하고 있는 메타 파일을 분석하여 사용자의 행위를 확인할 수 있다.

사실 여기서부터는 윈도우 아티팩트에 대한 내용이므로 최대한 간결하게 저장소 공간 관련된 내용만 작성하였다.

### \$UsnJrnl‧$J
복원된 $UsnJrnl 파일을 분석한 결과, 테스트로 '새 폴더'를 만들고 폴더명을 'Tools'로 변경한 사용자 행위 이력이 확인된다.
기본적으로 컴퓨터를 하루 8시간 동안 주기적으로 사용할 경우 4~5일 정도의 로그 확인이 가능하다.

<p align="center">
  <img src="https://i.imgur.com/1mIAdL5.png" alt="image"/>
</p>

<p align="center">
  <img src="https://i.imgur.com/dB1aSbx.png" alt="image"/>
</p>

이번엔 \$UsnJrnl에서 SID 값을 가진 폴더가 생성된 것을 확인할 수 있는데 자세한 설명은 아래 휴지통 폴더($RECYCLE.BIN) 내용에 기재하였다.

<p align="center">
  <img src="https://i.imgur.com/bHLbkQ3.png" alt="image"/>
</p>


### 휴지통 폴더($RECYCLE.BIN)

윈도우 운영체제는 기본적으로 볼륨이 연결되면 자동으로 마운트를 시켜주는 기능이 있는데, 마운트 시 외장저장장치 연결 형태로 기록이 남는다.

특히 볼륨이 운영체제에 연결되면 볼륨 내 휴지통 폴더(\$RECYCLE.BIN)에 SID(Security Identifier) 값의 이름을 가진 폴더가 생성된다. 당연히 폴더가 생성된 기록이니 그 행위는 고스란히 $UsnJrnl 로그로 동일하게 남긴다.

휴지통 폴더는 각 볼륨마다 자동으로 생성되고 운영체제에 의해 보호된다. 이 휴지통 폴더는 각 볼륨마다 생성되는데 볼륨에 있는 휴지통 폴더가 바탕화면의 메인 휴지통에 링크가 되어 있어 C드라이브가 아닌 다른 볼륨에서 파일을 삭제해도 바탕화면의 휴지통에서 볼 수 있는 구조로 되어있다.

따라서 저장소 공간의 논리 볼륨 내 휴지통 폴더를 분석하면 해당 저장소 공간 볼륨이 언제 마운트 되었는가에 대한 행위를 확인할 수 있다. 조금 더 나아가면 저장소 공간은 생성되자마자 윈도우에 자동 마운트 되기 때문에 저장소 공간을 언제 생성했는지로 귀결될 수 있다.

아래 휴지통 폴더를 살펴보자.  
$RECYCLE.BIN 폴더 내 SID 값을 가진 폴더가 생성된다. 특히 SID값 중 가장 마지막 숫자인 RID(Relative ID)값은 사용자 식별자로 윈도우에서 기본값으로 만들지 않은 그룹이나 사용자는 1,000 이상의 값을 가진다.

<p align="center">
  <img src="https://i.imgur.com/8TkR6w5.png" alt="image"/>
</p>

<p align="center">
  <img src="https://i.imgur.com/V2CTRf7.png" alt="image"/>
</p>

\$UsnJrnl 로그에서도 SID 폴더가 생성된 이력을 확인할 수 있다.
단, $UsnJrnl 로그는 시간이 지날수록 전에 쌓였던 로그가 없어질 가능성이 높아 분석 시 휴지통 내 SID 폴더의 생성 시간 기준으로 판단해야 할 가능성이 높다.

<p align="center">
  <img src="https://i.imgur.com/bHLbkQ3.png" alt="image"/>
</p>

정리하면, **UsnJrnl 로그의 시간 값을 활용**하거나 **SID 폴더가 생성된 시간 값**으로 **저장소 공간 볼륨이 마운트 되었다는 행위(저장소 공간 생성 시기)를 판단**할 수 있다.

만일 $UsnJrnl에 SID 관련 로그가 없는 경우, $UsnJrnl 레코드를 카빙하는 방법을 사용할 수 있다. 실제 비할당 영역을 대상으로 $UsnJrnl 레코드 카빙을 시도하면 상당히 많은 양의 레코드들이 복구되는 사례가 있고 논문에서는 심지어 약 4년 전의 기록이 복원된 경우도 확인된 적이 있다.

아래 그림은 필자의 외장하드에 쌓인 SID값이다. 이렇게 많이 쌓이는 이유는 SID 폴더는 자동으로 삭제되지 않고 계속 쌓이는 특성이 있기 때문이다. 따라서 해당 아티팩트만 가지고도 어느 컴퓨터에 연결되었는지를 판단할 수 있는 근거가 될 수 있다.

<p align="center">
  <img src="https://i.imgur.com/d63y3lo.png" alt="image"/>
</p>


### Registry 연계 분석 (with SID)
휴지통 폴더에 남는 SID값은 윈도우 레지스트리에서도 동일하게 남는다. SID값은 레지스트리의 SAM Hive(HKEY_USERS)에 저장되는데 이를 분석하면 사용자의 계정명을 확인할 수 있다.

만일 레지스트리에서 분석된 SID값이 동일하다면 해당 계정명을 가진 윈도우 사용자가 PC와 저장소 공간 볼륨을 연결했다는 것을 의미한다.

이는 피의자가 디스크를 연결하여 사용하지 않았다고 부인하더라도 혐의 구증에 결정적인 역할이 될 수 있는 의미 있는 데이터이다.

테스트에 사용한 저장소 공간 디스크의 휴지통에서 발견된 SID 폴더명은 **S-1-5-21-674465314-1800968367-3774905450-1001**이다.
그리고 SAM Hive를 분석한 결과 유저명은 KyleSong 이라는 것을 알 수 있다.

<p align="center">
  <img src="https://i.imgur.com/VLNctbA.png" alt="image"/>
</p>

<p align="center">
  <img src="https://i.imgur.com/tJxN1VS.png" alt="image"/>
</p>

이외의 추가적인 행위는 $Logfile 등 나머지 메타 파일을 이용하여 분석 및 연계하면 사용자 행위를 재구성할 수 있을 것이라 생각된다.

## Wrap-up

윈도우 저장소 공간은 우리가 알고 있는 NTFS 파일시스템을 사용하더라도 구조가 다르게 저장되기 때문에 일반적인 상용 도구에서 정상적으로 복원이 되지 않는다. 특히 물리적인 디스크 3개로 구성된 저장소 공간 환경에서 고의적인 파손으로 단 1개의 디스크만 분석 대상이 된다면 복원은 더 어렵다.

기본적으로 파일시스템의 메타데이터가 파싱되지 않기 때문에 일반적으로 상용 분석 도구를 통한 복원은 카빙의 방법을 사용한다. 하지만 이전 포스트에서 확인했듯이 카빙은 데이터 자체만 복원할 뿐 메타데이터의 부재 등 많은 한계점이 존재한다.

파일시스템을 이해한 상태에서 수동 복원 방법을 수행하면 더욱 복원율을 향상시킬 수 있는데 이를 실험하면서 의미 있는 결과를 얻어낼 수 있는 가능성을 확인하였다.

아래는 총 145개 파일 중 각 케이스별 복원된 파일의 결과를 히트맵(heatmap)으로 시각화하였다. 히트맵은 Python의 시각화 도구인 [Matplotlib](https://matplotlib.org/gallery/images_contours_and_fields/image_annotated_heatmap.html#sphx-glr-gallery-images-contours-and-fields-image-annotated-heatmap-py)을 이용하였다.
<p align="center">
  <img src="https://i.imgur.com/h2arRww.png" alt="image"/>
</p>

카빙의 기능적인 한계를 파일시스템 수동 복원 방법을 통해 보완할 수 있으며 특히 수동 복원 방법과 카빙 모두 수행하게 될 경우 복원율이 극대화되므로 이를 권장한다.

마지막으로 실험한 결과를 통해 포렌식적 의미가 있는 부분을 다섯 가지 항목으로 정리하며 저장소 공간 포렌식 시리즈를 마치겠다.

1. 동일 파일 복원 가능성
   - 원본 파일과 해시값이 동일한 일부 파일 복원 가능
2. 메타데이터 복원 가능성
   - $MFT로부터 파일 및 폴더 등 전체 엔트리에 대한 메타데이터 복원
   - 복원된 메타데이터를 통해 타임라인 재구성 가능
3. NTFS 주요 메타파일 복원 및 분석
   - $UsnJrnl 분석을 통한 사용자 행위 추적 가능성
   - $UsnJrnl에 남는 SID 분석을 통한 저장소 공간 디스크 연결 여부 확인 가능
   - 카빙 시 일부 조각난 메타파일($UsnJrnl, $LogFile) 확인되며 이를 통해 추가로 사용자 행위 분석 가능
4. 휴지통(RECYCLE.BIN) 분석
   - 저장소 공간을 구성했던 디스크의 휴지통(RECYCLE.BIN) 폴더에 남는 SID의 생성 시간을 이용하여 최초 저장소 공간 볼륨이 연결된 시간을 특정 가능
5. 레지스트리 연계 분석
   - 휴지통 폴더에 남는 SID 값과 SAM Hive 레지스트리 연계 분석을 통한 사용자의 계정명 확인 가능


## Reference
- <http://forensic-proof.com/archives/288>
- <https://ko.wikipedia.org/wiki/보안_식별자>
- <https://www.snoopybox.co.kr/1722>
- <https://matplotlib.org/gallery/images_contours_and_fields/image_annotated_heatmap.html#sphx-glr-gallery-images-contours-and-fields-image-annotated-heatmap-py>


## Copyright (CC BY-NC 2.0 KR)
본 게시글은 CC BY-NC 2.0 KR Licence를 따릅니다.  
비영리 목록으로만 사용할 수 있고, 저작자와 출처를 표시하면 언제든지 게시글을 자유롭게 사용할 수 있습니다.

<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
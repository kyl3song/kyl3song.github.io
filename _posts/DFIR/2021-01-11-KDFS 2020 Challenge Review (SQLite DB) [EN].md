---
title : "Blog #23: KDFS 2020 Challenge Review (SQLite DB Recovery) [EN]"
category :
  - Digital Forensics Challenges
tag : 
  - DFIR
  - Digital Forensics & Incident Response
  - KDFS 2020
  - Digital Forensics Chanlleges
  - Media History
  - SQLite DB
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
Another way of recover SQLite Database


## This Post Covers

Digital Forensics Challenges are great way to study for Dforensic-newbies as well as those who are eager to learn sophisticated tech skills in this field.

By taking part in it, you can even learn forensic tech trends and get a good chance to check how you could deal with the new artifacts that are not very publicly known.

Of course you need to research the problem and dive right in to come up with a breakthrough, making a better-than-other-writeups always results in high mark in the competition.

This post covers KDFS 2020 Forensics Challenge review, specifically a SQLite database and how to recover it.


## KDFS Digital Forensic Challenge
KDFS Challege, hosted by [Korea Digital Forensics Society](https://kdfs.jams.or.kr/), gives you approximately one month to analyze your findings from the image and draw conclusions in order to make your report to submit.

<p align="center">
  <img src="https://i.imgur.com/vvBDOtV.png" alt="image"/>
</p>

The target evidence in 2020 is a MicroSD card and the scenario is easily noticeable if we slightly look into it, Narcotics case.

<p align="center">
  <img src="https://i.imgur.com/7kVQul9.png" alt="image"/>
<br>[ Challenge Scenario ]</p>


## Getting into the Challenge
### com.android.chrome
For starters, lots of android package names like com.android.chrome in the image file tells us that the card was inserted in a mobile device before.

<p align="center">
  <img src="https://i.imgur.com/wRER67C.png" alt="image"/>
</p>

Getting deeper into the Chrome browser’s SQLite databases, all files seem fine just except one, Media History DB. Seemes Media History doesn’t contain any tables intact.

This is where your forensic-hunch needs to be activated, something’s wrong with it.
Sorry KDFS, suspicision is my habits from work.

<p align="center">
  <img src="https://i.imgur.com/gz6V0lf.png" alt="image"/>
</p>

There are numerous string data inside the file so we need to recover it, but HOW?

<p align="center">
  <img src="https://i.imgur.com/hsgn2DM.png" alt="image"/>
</p>

### How to Recover Media History DB
I already covered Media Hostory in [Blog #18](https://kyl3song.github.io/artifacts/NEW-Artifact-of-Chrome-Browser-(Media-History)-part-1/), [Blog #20](https://kyl3song.github.io/artifacts/NEW-Artifact-of-Chrome-Browser-(Media-History)-part-2/) a month ago. To be honest, I also ran into this artifact from the challenge in the first place, googled it and found that this is NEW artifacts of Chrome browser.

There are a lot of ways to extract meaningful information from SQLite DB.

1. Extract Strings  
   \- Using tools such as Strings, Bintext, etc.
2. Recover SQLite DB with Tools (Commercial/Free & opensource)  
3. Analysis its structure and modify it by yourself if it's wrong


Drawbacks of using extract-strings method is that it is possible to identify ASCII strings, and some fixed length time values. However, we cannot exactly identify each row or record if records are saved in variable length type.

Also we need more time to focus on the report in the end as much as evidence analysis. This makes us hard to make time to analyze the structure of new artifacts and modify one-by-one, not possible.

We need another way of recover this. That is **Data Transplant**.

### SQLite Structure Analysis
Since Media History DB is Chrome's newely created artifact, we have to look into the how data is stored and which data sits in the database.

After updating Chrome version, **1) collect the clean db** and **2) visit any web pages in order to play any videos & audios then collect the db** again.

From the second DB, I found 3 tables that have meaningful information, and their Leaf Pages sit as follows:

|Leaf Page Offset  | Table Name          |
|:----------------:|:-------------------:|
|0x7000 ~ 0x7FFF   |Playback             | 
|0x9000 ~ 0x9FFF   |PlaybackSession      |
|0x10000 ~ 0x10FFF |mediaImage           |  


Leaf Pages are where the real data is saved in SQLite DB. And the structure, leaf page offsets are the same as given Media History DB from the challenge.

So, we need to transplant the Leaf Pages portion to newly created DB that chrome made. Before that, let's only take a look at the Leaf Page structure below. Data is saved in Big-endian.

<p align="center">
  <img src="https://i.imgur.com/bCK48e3.png" alt="image"/>
<br>[ Before Playback table's Leaf Page is modified ]</p>

Two sections that have to changed

1. Number of Cells(rows): 24(0x18)  
   \- 12 cell offsets (2bytes pair)
2. Start of Cell Offset: 0x09CA  
   \- Same as the last offset data (09 CA) of hilighted Cell offset area  
   \- cell offsets are saved the other way around of the real data ➜ [Ref.](https://www.ahnlab.com/kr/site/securityinfo/secunews/secuNewsView.do?menu_dist=2&seq=22438)

Screen capture below shown the data is modified.

<p align="center">
  <img src="https://i.imgur.com/5uyACO8.png" alt="image"/>
<br>[ After Playback table Leaf Page's is modified ]</p>

With the same method, we need to change a couple of bytes of PlaybackSession table's leaf page.

<p align="center">
  <img src="https://i.imgur.com/Uoagiy6.png" alt="image"/>
<br>[ Before & After PlaybackSession table's Leaf Page is modified ]</p>

In mediaImage table's leaf page, it shows repetitive 2-byte pattern, 0x0CF3 in Cell offset area except one last pair.

<p align="center">
  <img src="https://i.imgur.com/dPUkjog.png" alt="image"/>
<br>[ mediaImage Leaf Page's 2-byte pattern ]</p>

This pattern is populated when deleting rows in DB as 2-byte-offset is move forward. Now we need to go to the data section to check if the real row data is still left. (Separator in blue)

<p align="center">
  <img src="https://i.imgur.com/Q5F6kLP.png" alt="image"/>
<br>[ mediaImage table's data area ]</p>

Now we understand fig. mediaImage Leaf Page's 2-byte pattern above that six 0x0CF3 values are repetitive because they have been deleted, and we also could presume the last one pair (0x0C09) had been deleted before 6-pair was deleted. Challenge designer might have done that.

Just FYI. the test shows that same patterns made after deleting data with SQL Query like below.

```shell
sqlite> DELETE FROM mediaImage;
```

<p align="center">
  <img src="https://i.imgur.com/2lnBiIL.png" alt="image"/>
<br>[ Cell offsets change before & after mediaImage data is deleted ]</p>

It's better to extract strings from mediaImage table because it doesn't have so much rich data except thumbnail url strings.

```shell
$ strings Media History > mediaImage.txt
```

<p align="center">
  <img src="https://i.imgur.com/YiEK6LM.png" alt="image"/>
<br>[ Strings from mediaImage table ]</p>


### Leaf Page Data Transplant & Results

Ok, we have modified a few bytes from leaf pages so far, and all we have to do is to transplant the Leaf Pages portion to newly created DB. Let's except the mediaImage as it used string-extraction method.

<p align="center">
  <img src="https://i.imgur.com/he3R5IA.png" alt="image"/>
<br>[ Media History leaf page transplant process ]</p>

Overwrite modified leaf page from the left to right within the same offset, and there we have very eaily(?) check the full data in tables.

<p align="center">
  <img src="https://i.imgur.com/fYGdlnc.png" alt="image"/>
<br>[ Recovered Playback Table ]</p>

<p align="center">
  <img src="https://i.imgur.com/5CQ6rON.png" alt="image"/>
<br>[ Recovered PlaybackSession Table ]</p>

Now we have more clear view of suspect's behavior.

## Wrap-up
The data is saved in Leaf Page in SQLite DB, so this recovery concept is to modify the page from broken DB and copy it to DB that has the same structure.

SQLite DB transplant method:
1. Check the real data in Leaf Page to see if it is still left.
2. Check the leaf page's header information and correct if it needs.
3. Copy modified leaf page over to the same structured DB within the same offset.

## Reference
- <https://kdfs.jams.or.kr/>
- <https://m.etnews.com/20190918000193>
- <https://www.sqlite.org/fileformat2.html>
- <http://forensicinsight.org/wp-content/uploads/2013/07/INSIGHT-SQLite-데이터베이스-구조.pdf>
- <https://www.ahnlab.com/kr/site/securityinfo/secunews/secuNewsView.do?menu_dist=2&curPage=1&seq=22324>
- <https://www.ahnlab.com/kr/site/securityinfo/secunews/secuNewsView.do?menu_dist=2&seq=22438>
- <https://www.ahnlab.com/kr/site/securityinfo/secunews/secuNewsView.do?seq=22494>
- <https://www.ahnlab.com/kr/site/securityinfo/secunews/secuNewsView.do?seq=22583>
- <https://www.acquireforensics.com/blog/sqlite-database-structure.html>


## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

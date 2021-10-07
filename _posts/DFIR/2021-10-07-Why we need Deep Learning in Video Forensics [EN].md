---
title : "Blog #30: Why we need Deep Learning in Video Forensics [EN]"
category :
  - Video Forensics
tag : 
  - Machine Learning
  - Deep Learning
  - Video Forensics

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
Limitations of video forensics & how to bridge the gap


## This Post Covers
A video file is comprised of lots of video frames. So, recovering the video also means the process that put these frames together, making scattered pieces into one video file. If the time value is not stored in the frame data(binary), it is extremely difficult to merge frames into a video as there's no standards to put them together in order. In this post, I'd like to bring up the cases that I experienced in 2019 and hope this might lead to a breakthrough to video forensics in the foreseeable future.


## Issues of Frame-based Recovery
Dashcams are more close and are able to easily accessd than DVRs because dashcams are way cheaper and we install it at the time of purchasing the car (at least in here in Korea). We get the Micro SD card out of the dashcam and use PC to play the video which is a piece of cake. Reasons of this is they use video containers like AVI, MP4. We only need a general video player software, or even we don't need it, windows has built-in video player app with a bunch of codecs inside. 

DVRs are little different. We set up DVRs generally for public interest in Store, in Mall for safety. DVR uses HDD(s) to store video files and those file are not be able to play with general applications. DVR manufacturers usually make their proprietary video formats or even their own filesystem in order to control DVR systems in their own way. We cannot view the video unless we install the software provided.

Dashcams have datetime inserted in the single frame whereas DVRs are more likely to read time value from the binary and display it using their software.

<p align="center">
  <img src="https://i.imgur.com/hjimf4Q.png" alt="image"/>
</p>

So, if the time value is not stored in dashcam's video frames, then we only recover the 
hundreds of thousands of frames(pictures) and give it to one who requested forensic analysis. That's not very pretty.


## Try OCR
When I moved to the video forensic squad in early 2019, and started recovering the video, I thought that it might be a bit harsh for the person who requested the analysis of evidence to see hundreds of thousands of photos with their bare eyes. Then OCR struck me all of a sudden, and then I installed Tesseract to test it out right away.

<p align="center">
  <img src="https://i.imgur.com/b9RB6Hm.png" alt="image"/>
</p>

Tesseract generates pretty decent OCR result like above, what I did next was to automate the process as follows.

- Iterate all frames(photos) to perform OCR
- Extract datetime out of OCR result
- Save datetime and filename (Sort is optional) to text file

The result text file provides the role of index to find filename corresponding datetime.

<p align="center">
  <img src="https://i.imgur.com/0FSfyqh.png" alt="image"/>
</p>

However, since dashcams use different fonts and sizes there are cases that misinterpret datetime even though Tesseract is a well-crafted OCR framework. Specifically it can't tell the difference well enough to distinguish 0, 6, 8 and '/' sometimes outputs 1 or vice versa.

## Train Tesseract
Since Tesseract 4.0, LSTM, Deep Learning model, has been included. That means Tesseract understands the context of sentence and it leverages to get the best result of OCR. But all I needed was a number and simple delimiters('/', ':', '-', etc.), I didn't really need the context.

Having said that, there are a bunch of options to increase the recognition rate. Most of text in dashcam are expressed in single line, like **YYYY-MM-DD hh:mm:ss** followed by model, sometimes voltages and so on. I used the option --psm 6 or 7, with Legacy Engine to get the best result, I tried mixing other options but there was no improvement.

<p align="center">
  <img src="https://i.imgur.com/yOJ8aUw.png" alt="image"/>
</p>

I looked up in Google and searched how to train Tesseract, and found [jTessBoxEditor](https://softfamous.com/jtessboxeditor/download/) and many similar tools to do it, but none of them gave me satisfactory results. So I had to finish my first analysis case using OCR by providing index text file. And this method couldn't be used in many cases.


## Meeting OpenCV
After a long web search, I saw an blog post titiled [Detect words out of a picture using Deep Learning and OpenCV](https://d2.naver.com/helloworld/8344782), and it was pretty similar what I was looking for. I was new to image processing technology, but I learned the importance of image preprocessing from this post which led me to learn OpenCV. Pushing me in the Computer Vision field made me well enough to create the improved recovery tool by the time I analyzed the second case that needed OCR to recover.

The clipped figure below represents the frames that are left throughout the unallocated area.

<p align="center">
  <img src="https://i.imgur.com/ulhXQA3.png" alt="image"/>
</p>

The tool runs in the following procedures.

### 1. Perform Frame-based recovery in unallocated area
Recover frames in a video container, AVI. Since the time values ​​are jumbled, it is difficult to see the video in this state. Even if we look at it, it came out on the one day of frames, and it came out on the another day's video frames, so there's no meaning as a video.

<p align="center">
  <img src="https://i.imgur.com/rF0kAZ8.png" alt="image"/>
</p>
   
### 2. Load video file and wait for user interaction
In this step, the tool loads the impaired video from the first step, and reads one frame, displays it on the screen afterwards. Wait for user's action.

### 3. Select ROI and crop it
ROI(Region of Interest) is an area of ​​interest that we want to process. The reason for setting the ROI is that reading the strings & characters only from the designated box is more accurate than reading the whole frame. Furthermore, it is far easier in the image preprocessing, as the target region you want to OCR is designated as coordinates, and only that area can be processed.

<p align="center">
  <img src="https://i.imgur.com/eUyUMHN.png" alt="image"/>
</p>

ROI can be selected by dragging the mouse, and it is displayed in yellow-green color in realtime. The image is cropped based on the coordinates after dragged and shown to the forensic examinators afterwards.

### 4. Image Preprocessing
Preprocessing is performed by reading frames one by one, and this process is crucial because it can increase the accuracy of OCR. Following substeps are basics of preprocessing.

- Resizing: Resize the image from the original high resolution image
- Grayscaling: Convert color to gray image
- Blurring: Blur the image removes small noise of image
- Thresholding: Contrast color pixels with a black pixel if the image intensity is less than some fixed constant. Sharpen the numbers to be read by OCR.

<p align="center">
  <img src="https://i.imgur.com/tXCJIM3.png" alt="image"/>
</p>

The reason for preprocessing is simple. This is a process that prepares for OCR to read well in the frame like the result figure above.

### 5. Perform OCR
Performs OCR and if the time value is in between target range, the tool accumulates frames into a video. Skip it if it's not in the range.

<p align="center">
  <img src="https://i.imgur.com/isXknQ6.png" alt="image"/>
</p>

The restoration was done and OCR was quite accurate than expected. Only the Oct. 10. 2019. video was extracted and restored. That's how I finished the second case using OCR.

<p align="center">
  <img src="https://i.imgur.com/2yvbA3l.png" alt="image"/>
</p>

This is not the end, it also has limitations. No matter how much ROI is specified, the background of the video constantly changes while the car is driving, and it was not easy to pre-process it which impacted the results of OCR.

## Need Deep Learning
I wanted to solve these problems through Deep Learning. My goal was to restore videos using deep learning technique, and I thought that it would be good for a research subject. The logic I was thinking of is the following.

***Automatic detect datetime area ➜ Configure ROI ➜ Crop ➜ Preprocessing(Grayscale, Blurring, Thresholding, etc) ➜ Split numbers ➜ OCR***

The automatic detection domain is deep enough to even have papers in Computer Vision academy, which was a challenging field. But as I moved from the video forensic squad to other squad, it was difficult to collect video dataset, frame samples. I had no choice but to give up after realizing one performs outstanding if it is related to the current work.

Currently, one of the OCR tech recognized by the society is Naver CLOVA OCR. They have published many papers at CVPR and ICCV, and has the top-tier level of accuracy in the world.

Now I find that they provide [CLOVA OCR API](https://guide.ncloud-docs.com/docs/en/ocr-ocr-1-1) with free of charge for limited counts, and some charge for the premium plan. you may try it out if you're interested.

<p align="center">
  <img src="https://i.imgur.com/dixRz9U.png" alt="image"/>
</p>


## Wrap-up
Most of the existing video forensics go through the process of analyzing and restoring data in a binary level. This is very important because it is the basis of video forensics, and most of the work is done this way.

However, I wanted to take a different approach. Someone needs to try that an approach from the upper level of video frames. Plan B may be able to cover Plan A's limitation.

Machine Learning & Deep Learning can be applied not only to the video forensics, but to other forensics fields in various way. It may apply to the audio forensics, audio that contained in the video. We would spark an idea to bride the gap if we more think about it.


## Reference
- <https://softfamous.com/jtessboxeditor/download/>
- <https://d2.naver.com/helloworld/8344782>
- <https://guide.ncloud-docs.com/docs/en/ocr-ocr-1-1>


## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>

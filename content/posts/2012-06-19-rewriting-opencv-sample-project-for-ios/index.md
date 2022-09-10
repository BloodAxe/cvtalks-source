+++
title = "Rewriting OpenCV sample project for iOS"
date = "2012-06-19"
tags =  ["opencv", "tutorials", "xcode"]
+++

A lot of changes been made since I posted a tutorial of using OpenCV library on iPhone/iPad devices. It was the time of iOS 4.x and OpenCV 2.1. The time to rewrite the whole sample project has come. With this post I announce that I going to update my sample project and I ask for your help. My idea is to write a project which will demonstrate use of several common computer vision algorithms. So the question is - which samples should I include? Now I can suggest following demo scenarios: 

  * Keypoint matching
  * Feature tracking
  * Corner detection
  * Color operations
Thanks in advance for your comments!

## Comments

**[Nicholas](#1127 "2012-06-23 13:26:58"):** Hi, I only ask that you provide step by step instructions to get opencv on the machine and into any new project assuming no technical knowledge. I enthusiastically await the release of your up to date tutorial!

**[EKhvedchenya](#1122 "2012-06-22 13:38:36"):** I wish to refactor this application a bit, so each demo case will be represented with a class derived from some abstract class SampleBase. something like that: class SampleBase { bool run(...); SampleInformation getInfo(); std::map >getOptions(); }; Each sample provide meta-infromation (name and its description) and exposes set of options that can be adjusted in run-time to see how they affect the sample routine. The main function run will run the sample either using video or still pictures and show the result to user returning processed image. The main benefit of this approach that we can add new samples easily into existing architecture really easy and fast.

**[Nicholas](#1146 "2012-06-25 12:40:11"):** Thankyou so much that is a Godsend. I see you have put something up already and I am going to have a look at it now.

**[Sansuiso](#1147 "2012-06-25 13:41:49"):** Thanks ! I cloned the new project today :-)

**[bittnt](#1102 "2012-06-19 21:23:19"):** Do you mind if you could provide feature tracking? I am also trying to work on feature tracking stuff. :) I got some initial results.

**[EKhvedchenya](#1111 "2012-06-21 09:31:50"):** I think, for this demonstration the simple KLT optical flow tracking will be used. It's reasonably fast. The second option - a tracking using template matching but i'm afraid, cv::templateMatch is too slow for iPad processor.

**[Sansuiso](#1112 "2012-06-21 09:46:47"):** Hello, KLT is fine, but may I suggest something around FAST+BRIEF maybe as an alternative ? The code is also usable from opencv straight from the box, and I feel like many people are using this combo now. Thanks again for the work!

**[EKhvedchenya](#1113 "2012-06-21 10:11:59"):** I appreciate any help for you! The design of this sample project will support easy add of new types of computer vision algorithm demonstations. The main screen of the application will show you the list of available samples. I'll post this design on next week so anyone can make a contribution to it.

**[bittnt](#1117 "2012-06-21 16:49:49"):** Yes. I tried the Optical flow from OpenCV to track hand with MyAVIController Example Code. It works quite well, most of time, it could reach above 15 frames per second.

**[Sansuiso](#1121 "2012-06-22 09:36:05"):** I cloned your github iOS sample project, so that I can add another feature tracking module to the demo app. That way you can add the KLT, and I can add FAST later. Is it ok that way ?

**[Josh](#1355 "2012-07-08 21:09:16"):** ... since you asked - maybe a tutorial on using KLT as an alternative to PTAM, the idea is to use this with the gyroscope to provide smooth tracking of 'dropped' virtual items. As far as I can see OpenCV gives you displacement on x and y (panning) but nothing for z. Cheers and thanks for the great blog - keep up the good work, Josh

**[Ievgen Khvedchenia](#1361 "2012-07-09 10:00:44"):** iPTAM would be definitely interesting sample! Thanks for this idea! Also i was thinking about video stabilization sample (OpenCV has such module) in connect with iDevices accelerometer.

**[EKhvedchenya](#1126 "2012-06-23 11:57:53"):** Hi, i've created a new repository here https://github.com/BloodAxe/OpenCV-Tutorial to write tutorial app from scratch. Feel free to add new samples dericed from SampleBase class. I continue writing application UI to get visualization as fast as possible.

**[EKhvedchenya](#1135 "2012-06-24 07:52:41"):** Nicholas, i'm going to write a series of tutorias with detailed explanation of every step i make during writing this application. Hope you find it useful.

**[Josh](#1453 "2012-07-13 23:08:45"):** ... a couple more requests; AR shadowing and tweaking luminosity to fit virtual object into the scene... I'm tackling this problem now - to keep things simple, for shadowing I'm thinking of using the detected foreground for depth or change in plane and adjusting the shadow. For lighting - thinking of getting the HSV range and adjusting the ambient light based on this. Both interesting problems that require tricks to work in the real world.


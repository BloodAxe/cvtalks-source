+++
title =  "Connecting OpenCV and Node.js inside Cloud9 IDE"
date = "2013-08-27"
tags =  ["opencv", "nodejs", "cloudcv", "cloud-computing"]
+++


Vacation time is over, and now i'm on my way from Tartu, Estonia where i participated in 48 km. inline speedskating marathon to Odessa. My bus have Wi-Fi onboard, so i decided to write a short success-story how i managed to build a C++ addon module for Node.js and run it on the real server inside the Cloud9 IDE.  You may also want to check the **[first tutorial][1]** since this guid relies on it.  The detailed step-by-step guide will be written in the next few days, but here is a key steps: 

## Prerequisites

  * Cloud9 account
  * Node.js
  * Express framework
  * CMake
  * OpenCV

## Building OpenCV

I built OpenCV manually. This is necessary since i had to configure OpenCV to fit the server configuration: 

  * Used static build instead of shared one
  * Disable all tests, samples and apps
  * Disable use of precompiled headers
  * Disable GPU and CUDA modules

### Disk space limit

You have only 1G of disk quota. When i ran OpenCV build for the first time it exit on 80% with "out of space" message. So it's better to disable all unused modules. **Don't clone OpenCV repository.** Download opencv archive instead! This will require small experience with shell, but nothing complicated: 

## Demonstration

A minimal Node.js demo that does nothing expect printing OpenCV build information can be seen here: [OpenCV Cloud9 demo][2]. It's ok to see "No app running" message from time to time. As i understood Cloud9 shuts down the apps if they are not used for some period.  Tweet me if you have interesting suggestions for the demonstration samples. 

   [1]: http://computer-vision-talks.com/2013/08/cloud-image-processing-using-opencv-and-node-js/
   [2]: http://cloudcv.computer-vision-talks.com/
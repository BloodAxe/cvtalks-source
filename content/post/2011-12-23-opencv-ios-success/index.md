+++
title =  "OpenCV + iOS = Success"
date = "2011-12-23"
tags =  ["opencv", "xcode"]
+++

It’s christmas time and i finally managed to deal with all my affairs before holidays. It’s probably the last post in the 2011 year. And i have a little present for you guys – it’s latest **build of OpenCV library** for the iOS platform, an **updated build scipt** and updated **OpenCV in iPhone sample project** with iOS 5 SDK support! 

<!--more-->

#### OpenCV build script for iOS SDK

Fortunately, OpenCV team has introduced toolchains for device and simulator builds. Now you don’t have to patch sources and make dirty hacks in CMakeLists.txt to build the library. Therefore build script become significatly shorter. You can download it as usual on github's project page: [OpenCV build script][1]. 

#### Prebuilt OpenCV binaries

If you don’t have a time to write few commands in shell, i prepared precompiled OpenCV library for you. It was built from trunk revision (7083) with XCode version. [Download precompiled OpenCV library for iPhone/iPad][2]. 

#### OpenCV Sample project

I updated sample project too, to reflect breaking changes in iOS 5. Now it supports both iPhone and iPad devices. Sample project can be found on github too: [OpenCV iPhone/iPad template project][3]. 

![][4]

   [1]: https://github.com/BloodAxe/OpenCV-iOS-build-script (OpenCV build script)
   [2]: http://computer-vision-talks.com/download/opencv_precompiled_ios_rev7083.zip
   [3]: https://github.com/BloodAxe/opencv-ios-template-project



+++
title =  "OpenCV iOS FAQ"
date = "2011-04-27"
tags =  ["opencv", "xcode"]
+++

# OpenCV iOS FAQ

This FAQ contains answers to numerous similar questions I was asked after my posts about using OpenCV with iOS SDK. 

## Step by step OpenCV Tutorial

OpenCV Tutorial  is a sample demonstration project for iOS devices that main goal is to show how to use OpenCV library in XCOde projects to write image processing code for iPhone and iPad devices. There are several step-by-step guides where i describe in details all development process. 

  * [Part 1][1]
  * [Part 2][2]
  * [Part 3][3]
  * Part 4 - in progress

## 

## Advanced reading

If you looking for more, here are some old posts i wrote about OpenCV and XCode. Some of them may become outdated, but they contains a lot of usefult infromationt too. 

  * <http://computer-vision-talks.com/2010/12/building-opencv-for-ios/>
  * <http://computer-vision-talks.com/2011/01/using-opencv-in-objective-c-code/>
  * <http://computer-vision-talks.com/2011/02/building-opencv-for-iphone-in-one-click/>
  * <http://computer-vision-talks.com/2011/03/opencv-build-script-with-xcode-4-support-is-here/>

## Necessary knowledge

To be able to understand what i'm talking about please ensure you have working experience with things listed below. In my posts i usually do not explain trivial things. 

  * Basics of CMake build syntax
  * A little of bash shell scripting
  * XCode 4 build system
  * C++ and OpenCV experience

## Build Script

Latest possible version is always here: [download id="1"] Issue tracker: <https://github.com/BloodAxe/OpenCV-iOS-build-script/issues>

## Common problems and their solutions

 

#### Building OpenCV itself

###### **Problem**: “Installing OpenCV headers ** BUILD FAILED **”

**Solution**: This can happen if OpenCV build is broken. So build script will not be able to make OpenCV even for MacOS platform. Try to find stable revision. 

###### **Problem**: Some other problems, not listed here

**Solution**: Remember, this build script is still experimental, so no warranties at all. Guys from Willow Garage can change project structure, cmake scripts or whatever and build script will fail… The best thing you can do in this case – contact with me and describe your problem. 

#### Using OpenCV static build in your projects:

###### Problem: How to setup my project?

**Solution**: There is template project on my github page. It already configured. Feel free to modify it as you want. Also you may want to do this manually be specifying additional search path and linker input. Here is detailed tutorial: <http://computer-vision-talks.com/2011/01/using-opencv-in-objective-c-code/>

##### **Problem**: Compile-time error “statement-expressions are allowed only inside functions”

**Solution: **The detailed explanation of the problem and it’s solution can be found here: [Complete iOS + OpenCV sample project][4].

   [1]: http://computer-vision-talks.com/2012/06/opencv-tutorial-a-collection-of-opencv-samples-for-iphoneipad-part-1/ (OpenCV Tutorial – a collection of OpenCV samples for iPhone/iPad – Part 1)
   [2]: http://computer-vision-talks.com/2012/06/opencv-tutorial-part-2/ (OpenCV Tutorial – Part 2)
   [3]: http://computer-vision-talks.com/2012/06/opencv-tutorial-part-3/ (OpenCV Tutorial – Part 3)
   [4]: http://computer-vision-talks.com/2011/08/a-complete-ios-opencv-sample-project/

## Comments

**[Arvind](#2611 "2013-01-14 07:45:07"):** Hi Ievgen, thanks for the great post on computer-vision-talks but when I click download kinks to OpenCV Build Script (8974) or Precompiled OpenCV library (5467), I get a 'Page Not Found' error. I also wanted to ask whether you have tried and succeeded in getting OpenCV functions IMREAD IMWRITE to work with iOS? If yes, how? I couldn't. Thanks, Arvind

**[Ievgen Khvedchenia](#2614 "2013-01-14 21:35:21"):** Hi, sorry for this inconvenience. I recently moved my blog to another hoster. I recommend to download Opencv framework from opencv.org. Functions from highgui module a not supported on iOS. You have to use iOS API to perform image read/write operations.


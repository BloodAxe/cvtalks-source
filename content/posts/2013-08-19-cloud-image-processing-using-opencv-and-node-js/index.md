+++
title =  "Cloud image processing using OpenCV and Node.js"
date = "2013-08-19"
tags =  ["opencv", "nodejs", "cloudcv", "cloud-computing"]
description = "This article describe how to build and use OpenCV in a cloud environment with a front-end in node.js"
+++

A long time ago i was playing with cloud-based image processing. The first reason why i didn't shared a reciple how to compile OpenCV as native app for windows azure cloud was trycky build process. It was too complicated and this tutorial will become outdated very quickly. The second one - Azure hosting wants a lot of money. So i put my research in this area on hold for better times.

And i think the time has come. You probably heard about Node.js - server-side asynchronous Java Script library. I have very small experience with Java-Script, but Node.js attracted me so strong i decided to study it. In this post i will describe how to connect Node.js and OpenCV together. On server-side OpenCV libary can be used for many things - generating CAPTCHA's recognizing scans, counting people in video streams.. So i beleive my tutorial will come in handy to people who is looking how to do image processing in the cloud environment.

Node.js is written in Java-script, and follows asynchronous programming model, where events and callbacks plays dominant role. OpenCV is a C++ library written in good old C and C++ and it don't bother with asynchronous and event-based programming model. Fortunately Node.js can interop with external modules written in C++. It's made via V8 engine which is a core of Node.js.

<!--more-->

## Disclaimer

I will be using Mac OS as host platform. Keep in mind this since some commands may differs for Ubuntu or Debian. **Windows users** \- you aren't lucky ones. I had no luck to build Node and OpenCV on windows so far. Please, post your comments if you succeed to build OpenCV module for Node.

## Prerequisites

In this tutorial we will need the following software:

  1. [XCode][1]
  2. [CMake][2]
  3. [Node.js][3]
  4. [OpenCV][4]

I assume you already installed XCode and CMake. In next two sections we describe installation of Node.js and OpenCV.

## Installing Node.js

To install Node.js i used [macports][5] software package manager. It's very similar to `apt-get` command from Unix.

**This command will install latests stable Node.js:**
    
    
    sudo port install nodejs
    

After running this command your Node.js is ready to use. It's not mandatory to use macports, you can install it using homebrew or build Node from the sources. It's up to you and will not affect further steps.

Node.js can be extended by C++ Modules. This is a way to interop with OpenCV library. To use OpenCV from Node.js we have to write C++ Mobule that Node.js can use. To build this module a special build system is used. We install it via Node Package Manager using following command:

**This command will install a GYP build system:**
    
    
    sudo npm install -g node-gyp
    

Take a note on "-g" flag which means "Install globally". By default npm installs new package to current directory.

On this step we are done with setting up Node.js.

### Installing OpenCV

OpenCV can be build as **static** or **shared**. I describe my experience using both options with Node.js:

#### Shared

By default, macports install shared libraries that are linked at run-time during application load. I was able to build a bare miminum Node.js module, but when i run it i got dyld load error. Maybe it's necessary to tell Node somehow where to search for OpenCV libs. But i decided to put this on hold and build a static OpenCV instead.

> If you know how to fix this easily your comments and suggestions are welcome!

#### Static

Building OpenCV as static libs is very trivial if you used CMake. Here is full stack of commands that clone latests OpenCV snapshot and build the final distribution package:

Upon completition you shoudl have a folder named `opencv-node-bin` with following contents:

### Writing your first C++ Module for Node.js

It's actuall a very easy to interop between JavaScript and C++, since JS uses V8 engine which is written in C++ too. I used the [official documentation][6] and [good examples][7] by Konstantin Käfer.

Node.js uses it's own build system to C++ modules - **node-gyp**.

There is a .gyp file which contains build options of your module. A GYP file is looks like JSON-like file where you specify your build targets.

**_binding.gyp_**
    
    
    {
      "targets": [
        {
          "target_name": "cv",
          "sources": [ "main.cpp" ],
          "include_dirs": [ "/users/BloodAxe/Develop/opencv-node-bin/lib/inlcude/" ],  
          "link_settings": {
                            'libraries':    ['-lopencv_core -lopencv_features2d -lopencv_contrib'],
                            'library_dirs': ['/users/BloodAxe/Develop/opencv-node-bin/lib/'],
                           },
        }
      ]
    }
    

Let's look at it in more details:

  * We define a single target with name `cv`.
  * The target includes a single source file `main.cpp`.
  * We adjust `include_dirs` and `link_settings` options to point our static OpenCV build.

Second mandatory file is package.json - it contains meta-information about our module:

**package.json**
    
    
    {
        "name": "cv",
        "version": "1.0.0",
        "main": "./build/Release/cv"
    }
    

Finally, a source code of our module. For sake of simplicity, let's export a single function that prints OpenCV build information from Node.js:

**main.cpp**

To build your module you can write the following command (assuming your current directory contains binding.gyp):
    
    
    npm build .
    

If you see the following output - your Node.js module that uses OpenCV has been built!

Now it's time to check how it works.

Here is a simple run script:

**_run.js_**
    
    
    var cv = require('./build/Release/cv');
    
    console.warn(cv.buildInformation());
    

To run it:
    
    
    node run.js
    

## Summary

I hope after reading this post you understood how to write a simple OpenCV wrapper for Node.js. In the next tutorial i will show how to perform simple image processing inside our Node module. Your comments for this post are welcome!

You can find an bare minimum Node.js module example here: **[OpenCV module for Node.js][8]**.

   [1]: https://developer.apple.com/xcode/
   [2]: http://www.cmake.org/
   [3]: http://nodejs.org
   [4]: http://opencv.org
   [5]: http://www.macports.org/
   [6]: http://nodejs.org/api/addons.html
   [7]: https://github.com/kkaefer/node-cpp-modules
   [8]: https://github.com/BloodAxe/CloudCV/tree/bare-minimum


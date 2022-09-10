+++
title =  "OpenCV 2.3 is available"
date = "2011-07-03"
tags =  ["opencv", "news"]
+++

New major release of OpenCV library is coming. Release candidate is available for testing right now! **Update =  ****Opencv 2.3 has been released on 5 June.** **Update 2: Added precompiled binaries of iOS!** Download OpenCV 2.3: 

  * Win32: <http://sourceforge.net/projects/opencvlibrary/files/opencv-win/2.3/>
  * Android: <http://sourceforge.net/projects/opencvlibrary/files/opencv-android/2.3/OpenCV-2.3.0alpha1-android-bin.tar.bz2/download>
  * Unix: <http://sourceforge.net/projects/opencvlibrary/files/opencv-unix/2.3/OpenCV-2.3.0.tar.bz2/download>
  * iOS: [download id="4"]

## General Modifications and Improvements

Buildbot-based Continuous Integration system is now continuously testing OpenCV snapshots. The status is available at [http://buildbot.itseez.com][1] OpenCV switched to Google Test (<http://code.google.com/p/googletest/>) engine for regression and correctness tests. Each module now has test subdirectory with the tests. 

## New Functionality, Features

Many functions and methods now take InputArray/OutputArray instead of "cv::Mat" references. It retains compatibility with the existing code and yet brings more natural support for STL vectors and potentially other "foreign" data structures to OpenCV. See <http://opencv.itseez.com/modules/core/doc/intro.html#inputarray-and-outputarray> for details. 

## Core

  * LAPACK is not used by OpenCV anymore. The change decreased the library footprint and the compile time. We now use our own implementation of Jacobi SVD. SVD performance on small matrices (2x2 to 10x10) has been greatly improved; on larger matrices it is still pretty good. SVD accuracy on poorly-conditioned matrices has also been improved.
  * Arithmetic operations now support mixed-type operands and arbitrary number of channels.

## Features2d

  * Completely new patent-free BRIEF and ORB feature descriptors have been added.
  * Very fast LSH matcher for BRIEF and ORB descriptors will be added in 2.3.1.

## Calib3d

  * A new calibration pattern, "circles grid", has been added. See findCirclesGrid() function and the updated calibration.cpp sample. With the new pattern calibration accuracy is usually much higher.

## Highgui

  * [Windows] videoInput is now a part of highgui. If there are any problems with compiling highgui, set "WITH_VIDEOINPUT=OFF" in CMake.

## **Stitching**

  * [opencv_stitching][2] is a beta version of new application that makes a panorama out of a set of photos which were took from the same point.

## Python

  * Now there are 2 extension modules: cv and cv2. cv2 includes wrappers for OpenCV 2.x functionality. opencv/samples/python2 contain a few samples demonstrating cv2 in use.

## Contrib

  * A new experimental variational stereo correspondence algorithm StereoVar has been added.

## Gpu

  * The module now requires CUDA 4.0 or later; Many improvements and bug fixes have been made.

## **Android port**

  * With support from NVidia, OpenCV Android port (which is actually not a separate branch of OpenCV, it's the same code tree with additional build scripts) has been greatly improved, a few demos developed. Camera support has been added as well. See <http://opencv.willowgarage.com/wiki/Android> for details.

## Documentation

OpenCV documentation is now written in ReStructured Text and built using Sphinx ([http://sphinx.pocoo.org][3]). It's not a single reference manual now, it's 4 reference manuals (OpenCV 2.x C++ API, OpenCV 2.x Python API, OpenCV 1.x C API, OpenCV 1.x Python API), the emerging user guide and a set of tutorials for beginners. Style and grammar of the main reference manual (OpenCV 2.x C++ API) have been thoroughly checked and fixed. Online up-to-date version of the manual is available at [http://opencv.itseez.com][4]

## Samples

Several samples using the new Python bindings (cv2 module) have been added: <https://code.ros.org/svn/opencv/branches/2.3/opencv/samples/python2>

## Optimization

Several ML algorithms have been threaded using TBB. 

## **Bug Fixes**

Over 250 issues have been resolved. Most of the issues (closed and still open) are listed at <https://code.ros.org/trac/opencv/report/6>. 

## **Known Problems/Limitations**

Documentation (especially on the new Python bindings) is still being updated. Watch opencv.itseez.com for updates. Android port does not provide Java interface for OpenCV. It is going to be added to [2.3 branch][5] in a few weeks. The list of the other open bugs can be found at <http://code.ros.org/trac/opencv/report/1>.

   [1]: http://buildbot.itseez.com/
   [2]: http://opencv.willowgarage.com/wiki/opencv_stitching
   [3]: http://sphinx.pocoo.org/
   [4]: http://opencv.itseez.com/
   [5]: http://code.ros.org/svn/opencv/branches/2.3/opencv


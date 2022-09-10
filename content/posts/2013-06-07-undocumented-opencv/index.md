+++
title =  "Undocumented OpenCV"
date = "2013-06-07"
tags =  ["opencv"]
+++

OpenCV library is widely used by computer vision engineers across the world. It contains almost all algorithms you may want for R&D or product development. It has production-ready build farm with tests and strong community that give nice feedbacks and discover errors. But nevertheless OpenCV has some strange issues and undocummented behaviour that can surprise you as minimum and crash your app as maximum.

<!--more-->

## How to get diagonal matrix in OpenCV

A typical parameter update computation in non-linear optimization using Levenber-Marquardt algorithm looks like this:
    
```cpp   
cv::Mat JtJ = J.t() * J;
x = (JtJ + lambda * JtJ.diag() ).inv(cv::DECOMP_SVD) * (J.t() * I);
```

So 'JtJ' is a square matrix and i want to get the diagonal matrix from it. Seems correct. But hey, **the member-function diag() does not return diagonal matrix**! It returns a vector of diagonal elements. Really? Fine. There is a static member-function diag(), maybe it can help?
    
```cpp 
x = (JtJ + lambda * cv::Mat::diag(JtJ)).inv(cv::DECOMP_SVD) * (J.t() * I);
```

Unfortunately, this is also wrong because this **diag()** constructs a diagonal matrix from a vector! So the correct solution to create a diagonal matrix in OpenCV is to use both diag's:
    
```cpp 
cv::Mat d = cv::Mat::diag( m.diag() );
```    

And the correct Lev-Mar update will have the following look:
    
```cpp 
x = (JtJ + lambda * cv::Mat::diag(JtJ.diag())).inv(cv::DECOMP_SVD) * (J.t() * I);
```

## How to increase speed of optical flow (KLT) in OpenCV

If you were using OpenCV version 1.x you should remeber there was a cvCalcOpticalFlowPyrLK function that allowed to pass image pyramids. The motivation to use precomputed image pyramids is performance gain. When you continuously tracking video frames you can save time by storing built pyramid from current frame.

The C++ API provide the following function prototype for this:
    
```cpp
//! computes sparse optical flow using multi-scale Lucas-Kanade algorithm
CV_EXPORTS_W void calcOpticalFlowPyrLK( InputArray prevImg, InputArray nextImg,
                                        InputArray prevPts, InputOutputArray nextPts,
                                        OutputArray status, OutputArray err,
                                        Size winSize = Size(21,21), int maxLevel = 3,
                                        TermCriteria criteria,
                                        int flags, double minEigThreshold);
``` 


From the function declaration and [documentation][1] it's visible that _prevImg_ and _nextImg_ are arguments for previous and next image buffers. Does it mean that you can't use prebuilt pyramids? No! This is not documented, but you can precompute image pyramid using **cv::buildOpticalFlowPyramid** like this:
    
```cpp 
std::vector<cv::mat> prevPyr, nextPyr;

// prevPyr assumed to be already initialized 
cv::buildOpticalFlowPyramid(nextFrame, nextPyr, winSize, maxLevel, true);
cv::calcOpticalFlowPyrLK(prevPyr, nextPyr, prevPts, nextPts, status, err, winSize, maxLevel);
prevPyr.swap(nextPyr);
```

On my PC this trick increased performance of KLT almost twice!

## cv::findHomography can return empty result

The documentation of cv::findHomography does not state it, but return value of cv::findHomography was always 3x3 matrix of CV_64FC1 type. Starting from approximately 2.4.5 release **cv::findHomography can return empty matrix in some cases**. This seems happen only when cv::RANSAC flag is passed. This [issue][2] has been reported but i suggest to check the computed homography before using it anywhere:
    
```cpp     
cv::Mat h = cv::findHomography(....)
if (!h.empty())
{
    // Use it
}
```
    

## Summary

That's all for now. I hope you enjoy working with OpenCV like i do. This post will be updated with new "easter-eggs". Feel free to share you own discoveries.

   [1]: http://docs.opencv.org/modules/video/doc/motion_analysis_and_object_tracking.html&usg=AFQjCNGCeanEXmqKq5DU7dpbp1m2xo0lPA
   [2]: http://code.opencv.org/issues/3057
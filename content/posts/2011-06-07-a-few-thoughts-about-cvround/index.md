+++
title =  "A few thoughts about cvRound"
date = "2011-06-07"
tags =  ["opencv"]
+++

During writing commercial iPhone app for one of my customers i noticed really strange slowdown in Surf descriptors. Cutting a long story shorter - while matching two frames usign SURF algorithm more than half of overall time (4, FOUR seconds!) was spent in cvRound function! Read more to know what it was and how did i fixed it.  I noticed that matching takes a while and decided to dug into profiler. When i saw the results i was a bit surprised.
<!--more--> 

![OpenCV Surf descriptor extraction on iPhone][1] 

As you can see from figure 1 - more than 4 seconds was spent in lrint1 function. After walking through a call stack i discovered that this function is called from cvRound: 
    
```cpp
    CV_INLINE  int  cvRound( double value )
    {
    #if (defined _MSC_VER && defined _M_X64) || (defined __GNUC__ && defined __x86_64__ && !defined __APPLE__)
        __m128d t = _mm_set_sd( value );
        return _mm_cvtsd_si32(t);
    #elif defined _MSC_VER && defined _M_IX86
        int t;
        __asm
        {
            fld value;
            fistp t;
        }
        return t;
    #elif defined HAVE_LRINT || defined CV_ICC || defined __GNUC__
        return (int)lrint(value);
    #else
        // while this is not IEEE754-compliant rounding, it's usually a good enough approximation
        return (int)(value + (value >= 0 ? 0.5 : -0.5));
    #endif
    }
```

The main problem with this function - it accepts doubles. Even if inlined, there is implicit conversion to double.  The problem is that SURF descriptor extraction is performed in floats, not doubles! Therefore additional float-to-double conversion occurs hundreds times per feature. I've added overloaded cvRound(float): 
    
```cpp
    CV_INLINE  int  cvRound( float value )
    {
    #if defined HAVE_LRINT || defined CV_ICC || defined __GNUC__
      return (int)lrint(value);
    #else
      // while this is not IEEE754-compliant rounding, it's usually a good enough approximation
      return (int)(value + (value >= 0 ? 0.5f : -0.5f));
    #endif
    }
```

After adding single precision floating point overload for cvRound and rebuilding OpenCV with my script i measured performance again. Descriptor extraction works **three seconds faster** (9 seconds against 12 on iPhone 3Gs). PROFIT! I've submitted a [patch][2] to into willowgarage. Hope they will integrate it faster than my previous patch :) PS: Update of OpenCV for iOS build script is coming - new optimization flags for armv7 release mode will be added. 

### Remarks

  1. **lrint **- round to nearest integer value using current rounding direction. More info: <http://pubs.opengroup.org/onlinepubs/009695399/functions/lrint.html>

[1]: opencv-surf-profile.png (OpenCV Surf descriptor extraction on iPhone)
[2]: https://code.ros.org/trac/opencv/ticket/1123


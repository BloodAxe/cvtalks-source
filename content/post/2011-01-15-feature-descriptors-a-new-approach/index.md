+++
title =  "Feature descriptors: A new approach"
date = "2011-01-15"
tags =  ["opencv"]
+++

Last year I was tightly connected with image processing and feature tracking/matching. For my needs I’ve used SURF and later RIFF descriptors. Both of them have strong advantages and but… SURF descriptor robustness are compensated by it’s computational cost. RIFF descriptor extracts much faster but not robust enough for my needs. My needs are very simple – doing markerless AR on mobile phone.
<!--more-->

So, we (me and two other co-authors) decided to develop our own descriptor. And we succeeded! Our descriptor (We called it LAZY) meets following requirements:

  * Scale invariant
  * Rotation invariant
  * Lighting invariant
  * Invariant to slight affine transformations
  * Works a lot faster than SURF (At least 2-3 times in comparison with OpenCV’s SURF implementation)

Here is video demonstration of how our descriptor works on the real video:

On the next week (Approximately in Wednesday, 26th January) in I’ll publish additional charts with detailed information about speed, robustness and comparison to SURF.

## Comments

**[EKhvedchenya](#36 "2011-01-17 16:06:06"):** For sure! However, it's not 100% production ready, and we will improve/change algorithm. So i will publish several posts, describing different parts of the algorithm. And then - write an article. (Maybe, i'll send it to the ICCV).

**[Stéphane Péchard](#35 "2011-01-17 15:58:39"):** Any plans on letting us know how it works?

**[Dewald](#46 "2011-01-22 17:23:14"):** Very nice. Looking forward to hearing more about LAZY. Nice name btw.

**[Shervin Emami](#101 "2011-02-26 11:04:35"):** Looks impressive! If you do post it online for free, then thanks! I can't wait to try it out :-)

**[Tony Marrero](#104 "2011-02-28 14:50:22"):** Great work guys ;) , will be looking forward to more of your posts.

**[yoshiboarder](#279 "2011-04-21 06:29:25"):** Wow! Very nice i wan't execute lazy algorithms. can you send me lazy demo source code?

**[EKhvedchenya](#280 "2011-04-21 09:55:41"):** No it's closed source yet. if one day it become open-source i'll let you know.

**[JaeSik](#305 "2011-04-27 13:03:43"):** It looks very impressive! Is this the result of LAZY on Iphone?

**[EKhvedchenya](#306 "2011-04-27 15:10:09"):** Descriptor extraction takes 2ms per frame (~100 keypoints on frame) on iPhone 3gs. However keypoint extraction takes a lot (0.5 seconds, now using SURF feature detector from OpenCV). So now i'm working on lightweight and fast enough feature detector especially for LAZY descriptor.

**[Brett](#745 "2011-11-28 23:04:03"):** What's the status of this? Looks like the last post was a while ago. Is the algorithm published anywhere? Is there any way for other developers to try this out?

**[EKhvedchenya](#746 "2011-11-28 23:23:44"):** Hi Brett! The status is "not enough time". To be honest i also want to finish the research of this descriptors, but my current work schedule consumes all my time and energy. It's challenging, interesting and i enjoy it. But from the other side i have no physical possibility to work on my own researches simultaneously. That's why development of LAZY descriptors is suspended for now. Thanks for the interest to it. I promised to myself to finish their research and publish an article about it at least.

**[Rehan](#793 "2011-12-29 09:17:17"):** Impressive, Can I get the code for research purpose? Thanks. Dr. Rehan

**[Rehan](#794 "2011-12-29 09:17:49"):** Impressive, Can I get the code for research purpose? Thanks. Dr. Rehan


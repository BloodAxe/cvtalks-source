+++
title =  "On answering reader questions"
date = "2012-04-10"
tags =  ["news"]
+++

A question from intenet:

> Dear Ievgen, I'm researching the field of computer vision and object recognition in particular. I'm working for SENSUS in Amsterdam and we focus on social computing solutions for educational and health care sector. While researching and exploring I came up with a couple of simple questions that you might be able to help me with. I'm quite new to the concept of computer vision and only had some experience with simple blob tracking in OpenCV with use of OpenFrameworks. With best regards, Ralph Das

<!--more-->

From your Twitter profile it became very clear you have a lot of experience in this field. I hope that you can share some insights on the questions I have: 

> Where is computer vision and in particular object recognition (SURF SIFT etc.) used on a larger scale/commercial basis. 
> Is this only in industrial quality check systems or are there any other examples? 
> To me the technology seems only applied between the walls of research institutes/ universities. 
> Is it ready for a next phase? Is this this the case and why? 

The object recognition using features are widely used in marker-less augmented reality and other areas that require object identification by it’s visual description. Probably, the best example of the large scale object recognition is the Google Image Search. The idea and it’s implementation is awesome. After uploading an image, the system recognizes it. If it’s a well-known object you will get the contextual information about it; otherwise – at least similar images will be displayed. By other words, Google uses pattern recognition, OCR and image database lookup to extract as much information from your image as possible. Other good example is a particular product location among large products database by it’s photo. Just imagine – you make a picture of a product packaging by a phone, and get all information about this item. From the technical side it’s a lookup against large image database (10,000 and more). Obviously, that direct comparison of two bitmaps is useless and too expensive. Fortunately, feature descriptors like SURF or SIFT comes in help and provide good and robust pattern description. And I almost forgot about SLAM-based 3D reconstruction and structure from motion estimation. The SLAM is acronym of simultaneous localization and mapping. By tracking 2D feature points it reconstruct 3D points using triangulation algorithm and then perform optimization of camera extrinsic and a 3D cloud to minimize the reprojection error. SLAM is very sensitive to quality of the input data (tracks of 2D points). Usually the KLT tracking algorithm is used to get frame-to-frame correspondences. Limitations of KLT tracking is well known: it tracks only slight movements, the points can ‘drift’ during frames, large number of outliers (false positive tracks), cannot be used for loop-closing. Using feature descriptors for image tracking gives you not only stable tracking with small number of outliers but also you get a tracking history for each feature for last N frames. With this data you can find lost features on new frame and reconstruct scene. Almost all software for video post-processing use them to reconstruct the scene and them embed the artificial 3D object into it. Also it’s true for panorama stitching and HDR making software – they use feature matching to align the images w.r.t to each other. 

> I know that some of the object recognition algorithms are patented. Does that also count for algorithms like LAZY, BRIEF, ORB etc.?

 The original SIFT and SURF implementations are patented. BRIEF and ORB are not. LAZY algorithm is still in development and not published yet. 

> Mobile devices seem more and more capable of handling tasks like feature detection. What are the primary technical concerns constraints when using a mobile device for these tasks? 
> I can imagine that the processing speed is a major factor but are there more?

The processing power of mobile CPU are getting closer and closer to the desktop one. The quad-core mobile processor is not a surprise anymore. But mobile CPU have so different architecture in comparison to desktop processors. Mobile CPU are battery friendly, so they try to put the CPU to sleep when it suitable. From the other hand, any AR application use CPU for 100%. This drains battery extremely fast. Also, mobile device can become hot, since it’s not designed for long use at 100% workload. Mobile CPU has smaller cache size. That’s why processing of the same amount of data is significantly slower in comparison to desktop analogues. Nevertheless, mobile CPU is optimized to process HD media-content like in real time. This is possible with help of special CPU-features like SIMD (Single Instruction Multiple Data). The SIMD engine allows you to perform image processing up to 8x times faster. To use processor with maximum efficiency you will have to rewrite hotspots in your code using either intrinsic functions or low-level assembly language. It’s not a trivial task, but the result is worth of it. 

> Systems like Google Goggles and Layers new Stiktu try to push the boundaries on the field of AR. 
>Is the object/pattern matching done on the mobile itself or is it (like I expect) server based? Can you share you thoughts on the architecture of these systems?

You are right, these apps do all calculations on the server side. The thin client app contains just a rich UI and simple business logic. Let’s examine Google Goggles. You make a photo and receive some information about it. It think the image itself is not sending over network, only list of descriptors found on query image. Also I suppose they do resizing of input image to something like 640x480, because it’s good tradeoff between image quality and calculation speed. After resizing and converting to grayscale the descriptors are extracted and sent to the server. Probably, the resized grayscale image is also sent to improve search. The server back-end perform visual object search. The path can look like this: 

  * Compute a image hash and find hash matches among the database. This step will reduce the search area.
  * If GPS data is available use it to refine search area to current device position.
  * Compute image histogram and reject images with large histogram difference.
  * Perform features matching using KD-trees to get image candidates.
  * Perform OCR to find text on query image.
  * Intersect textual and image search results to get final answer.
  * Return response
  
I hope Google developers won’t be laughing too loud. I almost sure their system is much more complicated and use a lot of additional metrics to describe image. An, of course, this computations are done on a cluster with lots of servers which involves huge part of all Google services. Update =  Thanks to @Sansulso posted very interesting video presentation of machine learning in Google Googles: <http://techtalks.tv/talks/machine-learning-in-google-goggles/54457/> You can always ask me about whatever any time in [twitter][2], [mail][3] or via Skype (eugenekhvedchenya).

   [2]: https://twitter.com/#!/cvtalks
   [3]: mailto://ekhvedchenya@gmail.com

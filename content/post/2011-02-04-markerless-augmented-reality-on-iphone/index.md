+++
title = "Markerless Augmented Reality on iPhone"
date = "2011-02-04"
tags =  ["opencv", "iphone", "ar"]
description = "First attempts on trying to implement state of the art Augmented Reality using mobile phone and OpenCV"
+++

Hello everyone! Today i want to share my results in research of markerless augmented reality. The  main idea - do fast and quality AR without those damn markers and give the ability to use real object as a target.  Markerless augmented reality is very similar to marker-based systems like ARToolkit with one major difference - such technology use real object as a target for augmentation. It can be almost any kind of objects - photos, logos, beer bottle or Cola can. 

<span class="more"></span>

The common pipeline is very trivial: 

  1. Estimate transformation between real-world target and virtual camera.
  2. Render virtual objects using estimated transformation.

The most complex part - is to find a target on video frame. Actually, algorithm you will use depends on the kind of a target. In my today's showcase i use printed image of the graffity pattern. And i will draw the 3d model on the top of this image. 

![][1] 

For target detection i use feature descriptors. At each frame, algorithm detects keypoints (using FAST feature detector), extracts descriptors (using LAZY descriptor) and match them with reference descriptor set (from the image above). Those matches gives us all necessary information to estimate transformation between camera and target. All necessary functions exists in OpenCV. Here is screenshot from working sample running on the iPhone 3Gs: 

![Markerless augmented reality][2]

## Facts:

  * Runs at ~10 fps on iPhone 3Gs.
  * 3rd party libs used: OpenCV, OGRE, Boost.

## Further work:

There are a lot of things i wish to do: 

  * Achieve real-time (25+fps) on iPhone
  * Introduce two-stage algorithm (expensive initialization <-> lightweight tracking) to get significant performance boost.
  * Public demo in AppStore :)

## Instead of conclusion

I'm sure that the markerless AR has a great future. With this technology we can create impressive presentations and games with seamless fusion of the real and augmented worlds. You will no more require to print some kind of markers. Just imagine - any kind of object can be a door to world of the augmented reality!

   [1]: graffiti-300x240.png (graffiti)
   [2]: Screenshot-2011.02.04-10.22.36.png (Markerless augmented reality)



+++
title =  "KAZE 1.5.1"
date = "2013-06-17"
tags =  ["opencv", "algorithms"]
+++

<div class="alert alert-danger">
    <p class="lead">This post is outdated.</p>
    <p>
        Please, visit updated post: 
        <a href="/articles/kaze-1.6-in-opencv/">Integration of KAZE 1.6 in OpenCV</a>
    </p>
</div>

A new version of KAZE features has been integrated my private fork of OpenCV (You can find it's here: https://github.com/BloodAxe/opencv/tree/kaze-features). We're on the way to make pull-request and integrate KAZE features to official OpenCV repository.

There only few things are left:

  * Include KAZE into features2d unit tests.
  * Rewrite KAZE to support OpenCV threading API.
  * Expose adjustable parameters of KAZE algorithm.
  * Do code cleanup and documentation for pull request.

I think we (Pablo, KAZE author) and me complete these steps in a near future. During our hard work you can enjoy these nice charts that shows how awesome KAZE features are:

![KAZE 1.5.1 Rotation Test][1] ![KAZE 1.5.1 Scale Test][2]

   [1]: chart_1.png
   [2]: chart_1-1.png


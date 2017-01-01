+++
title = "How to detect circles in noisy images"
date = "2014-07-14"
tags = ["opencv", "algorithms", "reddit"]
+++

p
    | This was a request from 
    a(href="http://www.reddit.com/r/computervision/comments/2a1lvi/help_how_to_process_this_image_to_find_the_circles/") /r/computervision. 
    | A reddit member was asking on how to count number of eggs on quite
    | noisy image like you may see below. 
    | I've decided to write a simple algorithm that does the job and explain how it works.

    div.beforeafter
        img(src="source.jpg",alt="before")
        img(src="display.jpg",alt="after")

span.more

h2 Step 1 - Filter image

p
    img(src="source.jpg",alt="Source image")
    | The original image has noticeable color noise and therefore it must be filtered before we pass it to further stages.
    | Ideally you should choose filter algorithm based on your task and noise model. For the sake of simplicity, 
    | I will not use Weiner filter or deconvolution to deal with blur and artifacts that are present on source image.


p
    | Instead, I will use pyramidal mean-shift filter. 
    | This algorithm works quite well on this problem and helps to get rid of artifacts.

    pre.
        cv::Mat inputRgbImage = cv::imread("input.jpg");
        cv::Mat filtered;
        cv::pyrMeanShiftFiltering(inputRgbImage, 
                                  filtered, 
                                  spatialWindowRadius, 
                                  colorWindowRadius, 
                                  2);

strong Result of filtering
div.beforeafter.twentytwenty-container
    img(src="source.jpg",alt="before")
    img(src="filtered.jpg",alt="after")

h2 Step 2 - Increase sharpness

p
    | As you may see - eggs edges are not too sharp. For circle detection we will use Hough transform. OpenCV' Hough algorithm implementation use Canny edge detector to 
    | detect edges. Unfortunately it's not possible to pass manually computed binary image. 
    | Instead we have to pass 8-bit grayscale image for circle detection. 
    | Therefore we may want to increase their sharpness to make the image more friendly for Canny detector.

p
    | Sharpening can be easily done via unsharp mask. 

    pre.
        // Perform in-place unsharp masking operation
        // http://opencv-code.com/quick-tips/sharpen-image-with-unsharp-mask/
        void unsharpMask(cv::Mat& im) 
        {
            cv::Mat tmp;
            cv::GaussianBlur(im, tmp, cv::Size(5,5), 5);
            cv::addWeighted(im, 1.5, tmp, -0.5, 0, im);
        }

    | However, unsharp mask can cause artifacts on edge borders which may lead to double edge.
    | I'm dealing with it by adding laplaccian component to final result:

    pre.
        cv::cvtColor(filtered, grayImg, cv::COLOR_BGR2GRAY);
        grayImg.convertTo(grayscale, CV_32F);
        
        cv::GaussianBlur(grayscale, blurred, cv::Size(5,5), 0);
        
        cv::Laplacian(blurred, laplaccian, CV_32F);
        
        cv::Mat sharpened = 1.5f * grayscale
                          - 0.5f * blurred
                          - weight * grayscale.mul(scale * laplaccian);

    strong Result of sharpening
    div.beforeafter.twentytwenty-container
        img(src="filtered.jpg",alt="before")
        img(src="filteredGray.jpg",alt="after")

h3 Step 3 - Detect circles

p
    | After we got nicely filtered and enchanced image we can pass it to cv::HoughCircles to detect circles on the image.
    | According to problem task, we can limit the maximum radius of circles that we are interested in (we need only small circles).
    | In addition eggs can be close to each other, so it make sense to set minimal distance between two circles to minimal 
    | allowed diameter of the circle.

    pre.
       cv::HoughCircles(filteredGray, 
                        circles, 
                        cv::HOUGH_GRADIENT, 
                        2,   // Accumulator resolution
                        12,  // Minimum distance between the centers of the detected circles.
                        cannyThreshold, 
                        accumulatorThreshold, 
                        5,   // Minimum circle radius
                        20); // Maximum circle radius


h4 Step 4 - Validate eggs

p
    | I've skipped validation step since I don't know what are the requirements to detected eggs. 
    | But I suppose that after detection of possible candidates using Hough the algorithm should validate each contour
    | to verify it is exactly what we were looking for. Here are some hints what we can check:

    ul
        li Contour defects - how egg shape is close to ideal circle
        li Contour breaks - is there any breaks in egg boundary or not
        li Egg color - perhaps we need to count objects only of particular color
        li Neighbours - maybe (or maybe not) we need only isolated objects.

h4 Step 5 - GPU Speed-up

p
    | The given implementation is very slow since it use CPU and doing a lot of stuff that can be efficiently computed on GPU. 
    | Fortunately, OpenCV has GPU implementations for mean-shift segmentation and hough transform and image blending.
    | You can easily speed-up this algorithm by using cv::gpu types and functions instead. 

h4 Bonus content

p
    | Readed to the end of article? I'm impressed :) Here you go, the full source code of the algorithm and small playground to
    | tweak setting in runtime:

    script(src="https://gist.github.com/BloodAxe/943fb14220021113d405.js")

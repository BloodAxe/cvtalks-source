+++
title =  "Image and video processing: From Mars to Hollywood with a stop at the hospital - Week 2"
date = "2013-02-05"
tags =  ["coursera"]
+++

This is a second part of ["Image and video processing: From Mars to Hollywood with a stop at the hospital"][1] class.

In optional taks for Week 2 we were asked to test various predictors for JPEG-LS compression algorithm. In this post i want to share my results and discuss how we can achieve better prediction rate.
<span class="more"></span>

# Compare prediction algorithms for loseless image compression

**Problem statement**

> Compute the histogram of a given image and of its prediction errors. If the pixel being processed is at coordinate (0,0), consider predicting based on just the pixel at (-1,0); predicting based on just the pixel at (0,1); predicting based on the average of the pixels at (-1,0), (-1,1), and (0,1). Compute the entropy for each one of the predictors in the previous exercise. Which predictor will compress better? 

**A very, very little theory** So, first of all, JPEG-LS is a lose-less image compression algorithm. It uses variable-length coding (Huffman coding in particular) to minimize the amount of data needed to represent our image. So the idea behind this to predict the next pixel values using values of previous pixels and encode this error instead of pixels itself. So if we guess a lot, the distribution of the error would be non-uniform with a peak in near-zero area. So Huffan coding algorithm will be able reduce the code length.

# Solution

Based on the entropy of the input image we can describe prediction rate as follows: $$R=H_s / H_e$$, where $$H_s$$ is the entropy of the input image, and $$H_e$$ - is the entropy of the prediction error.

So, for my tests i wrote a simple OpenCV program that contains predictors test. You can find the full source code here: **<http://pastebin.com/zWz9vaD7>**.

For all the tests i used well-known lena image:

![lena.jpg][2]

Which has following parameters of the gray-value representation:

> Std.Dev. 731.663 Entropy: 5.16456

Here i'm gonna describe only result of predictors. So we start from trivial predictors that use value of the previous pixel:
    
    
    int predict_by_previous_pixel_in_row(const cv::Mat_& img, int row, int col)
    {
        if (col > 0)
            return img(row, col - 1);
        return 0;
    }
    
    int predict_by_previous_pixel_in_col(const cv::Mat_& img, int row, int col)
    {
        if (row > 0)
            return img(row - 1, col);
        return 0;
    }
    

These two predictors give us fair enough prediction rate of ~1.47:

> predict_by_previous_pixel_in_row: Std.Dev. 3322.94 Entropy: 3.50525
> 
> predict_by_previous_pixel_in_col: Std.Dev. 3940.22 Entropy: 3.20661

Then i tried a predictor based on average value of two previous consecutive pixels:
    
    
    int predict_by_2prev_pixels_in_row(const cv::Mat_& img, int row, int col)
    {
        if (col > 1)
            return ((int)img(row, col - 2) + (int)img(row, col - 1)) / 2;
        else
            return 0;
    }
    

Regarding to the benchmark, this prediction strategy is worth than previous.

> predict_by_2prev_pixels_in_row: Std.Dev. 2908.91 Entropy: 3.70031

Intuitively it's correct the more pixels we use for prediction, the more precise it will be. And the less prediction error we achieve. The fourth predictor i've implemented was neighbor averaging predictor:
    
    
    int predict_by_neighbors(const cv::Mat_& img, int row, int col)
    {
        if (col > 0 && row > 0)
        {
            int a = img(row, col - 1);
            int b = img(row - 1, col - 1);
            int c = img(row - 1, col);
            int s = a + b + c;
            
            float m = s / 3.0f;
            return (int)m;
        }
        else
            return 0;
    }
    

It predicts the value of the next pixel by averaging values of it's closest neighbors (top, left and top-left pixel). This predictor shows better results than **predict_by_2prev_pixels_in_row**, **predict_by_previous_pixel_in_row**, but for some reason it cannot beat **predict_by_previous_pixel_in_col** algorithm:

> predict_by_ neighbors: Std.Dev. 3511.97 Entropy: 3.3431

Then i remembered as Guillermo mentioned that JPEG-LS uses some sort of edge-detection to do this. So i tried to calculate gradient in X and Y directions using neighbor pixel values and compute the prediction using it:
    
    
    int predict_by_gradient(const cv::Mat_& img, int row, int col)
    {
        if (col > 0 && row > 0)
        {
            int a = img(row, col - 1);
            int b = img(row - 1, col - 1);
            int c = img(row - 1, col);
                    
            int gx = a - b;
            int gy = c - b;
    
            return (int)(b + (gx + gy) / 2.0f);
        }
        else
        {
            return 0;
        }
    }
    

Well, at this moment this algorithm demonstrate the best prediction rate:

> predict_by_gradient: Std.Dev. 3881.62 Entropy: 3.18335

Can we increase even more? Yes! I think we should use 3x3 neighbor area based prediction for this. I'm pretty sure i'm reinventing the wheel, but i think it's very fun and useful to train the brain with puzzles like this :) So, does anyone wants to share their predictors? Let's do a small challenge!

**Useful information for OpenCV users**. The source code of my benchmark you can find here: <http://pastebin.com/zWz9vaD7>.

   [1]: https://www.coursera.org/course/images
   [2]: lena.jpg
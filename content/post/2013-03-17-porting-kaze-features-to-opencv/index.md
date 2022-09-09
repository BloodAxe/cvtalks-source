+++
title = "Porting KAZE features to OpenCV"
date = "2013-03-17"
tags =  ["opencv", "algorithms"]
+++

Recently i came across the publications to a new features called [KAZE][1] (Japanesee work meaning "Wind"). They interested me, because KAZE authors provided very promising evalutaion results and i decided to evaluate them too using my OpenCV features comparison tool. Fortunately KAZE algorithm is based on OpenCV, so it was not too hard to wrap KAZE features implementatino to cv::Feature2D API.

<!--more-->

<div class="alert alert-danger">
    <p class="lead">This post is outdated.</p>
    <p>
        Please, visit updated post: 
        <a href="/articles/kaze-1.6-in-opencv/">Integration of KAZE 1.6 in OpenCV</a>
    </p>
</div>

**_Impatient readers:_** you can grab the most recent version of KAZE port to OpenCV here: [kaze-features][2].

Short video demonstration:

## KAZE & OpenCV

I'm not gonna describe the algorithm by itself or implementation details, you may wish to read the [original paper][3] or look at the [code][4] if you have enough mana.

If not - don't worry, using KAZE features is easy like any other feature algorithm:
    
    
    cv::Mat image = /* ... your image goes here ... */
    
    cv::KAZE kazeFeatures;
    kazeFeatures(image, keypoints, descriptors);
    
    

**_KAZE algorithm is not included in the official OpenCV repository yet_**. It exists only in my fork of the opencv on github. You can clone it and build OpenCV with KAZE algorithm by yourself:
    
    
    git clone https://github.com/BloodAxe/opencv kaze-features
    

I'm going to cooperate with KAZE authors and help them to include KAZE algorithm to OpenCV library. So [kaze-features][2] branch in my opencv fork will be updated constantly with more recent versions.

## KAZE Estimation

I did a comparison of KAZE features with other descriptors and here are the results that i got using my [OpenCV Comparison Tool][5]:

![Rotation invariance][6]

![Scale invariance][7]

![Brightness invariance][8]

![Blur invariance][9]

## Conclusion

<strike>Regarding to the results of estimation the KAZE algorithm is much more robust than state of the art algorithms like SURF and FREAK or BRISK.</strike>

**_It's amazing! In all tests KAZE features shows best results !!!_**

Currently KAZE features are somewhat slow. I contacted with one of it's authors and he assured me that they are working on this problem so i beleive they will improve their performance in the near future.

## Licensing

I've contacted with one of authors of KAZE features and asked him about code license and usage rights. Here it is:

**_Pablo Fernandez Alcantarilla_**

> The code is released under BSD license. So it is completely free. That was my initial goal, I wanted to do something open source better than SIFT, SURF so people don't need to mess up with the patents.

   [1]: http://www.robesafe.com/personal/pablo.alcantarilla/kaze.html
   [2]: https://github.com/BloodAxe/opencv/tree/kaze-features
   [3]: http://www.robesafe.com/personal/pablo.alcantarilla/papers/Alcantarilla12eccv.pdf
   [4]: http://www.robesafe.com/personal/pablo.alcantarilla/code/kaze_features_1_4.tar
   [5]: https://github.com/BloodAxe/OpenCV-Features-Comparison
   [6]: Rotation-Test.png
   [7]: Scale-Test.png
   [8]: Brightness-Test.png
   [9]: Blur-Test.png


+++
title =  "Success-story: Fueling ARBasketball up with NEON"
date = "2013-06-30"
tags = ["opencv", "neon", "xcode"]
+++

img.pull-left.img-thumbnail(src="arbasketball-logo.jpg",alt="ARBasketball")

p ARBasketball was one of the first augmented reality-based games in App Store. It has been published in 2010. In these days not many people have even heard about AR. I mean it wasn't so popular as it became now. But there were people who saw the great potential in this growing market. One of them was Konstantin Tarovik, the author of [ARBasketball][1]. I must confess - I saw this application before, but had no idea it's author lives in Ukraine, and in the same city as I am! It was really surprising to discover there is another passionate person in Odessa and is also interested in computer-vision and AR! While I was concentrating on back-end development and most of my projects were under NDA, Konstantin aimed for product development. Actually that was the reason he contacted me... 

h2 First contact

p Konstantin contacted me in fall 2012. He wanted me to help him optimize performance of ARBasketball on iOS devices. I agreed, because it was very interesting task and I was happy to help countryman. There were two problems we were going to solve: - Performance - Robustness

p In computer vision choice of any algorithm is a trade-off between speed and quality: you lower the frame resolution and get more speed, but lose precision; you increase the number of features and get more robust tracking but lose speed, etc. In our case we had an issue that game was not very stable in low-light or in over saturated scenes. The second issue was performance of marker detection algorithm. To provide the best game experience you should have at least 25 fps. In other words, you have only 30 milliseconds to acquire the new frame, find the marker on it, update physics and render the frame. Real-time is always a challenge.

h2 Wrong way

p 
    | To provide lighting-invariant marker detection I decided to apply [adaptive threshold][2] since this filter can easily deal with non-uniform lighting and it works perfect for low-light cases too. Speed of adaptive threshold depends on window size, in our case the best results were achieved with window size of 7 pixels. Unfortunately, OpenCV implementation of adaptive threshold was too slow and we decided to utilize GPU for thresholding. Unfortunately, the OpenCL technology is not yet supported by iOS devices, so we had to use OpenGL ES 2.0 shaders and textures to simulate GPGPU. A great Brad Larson's library [GPUImage][3] helped us a lot in this. After we performed the test it became clearly visible there is a huge lag when you try to access the result of GPU processing:
    img(src="chart_1-2.png",alt="GPU-accelerated adaptive threshold")

p While thresholding took less than 5 milliseconds per frame (including gray-scale conversion), the frame download took almost twice more time. This news was really discouraging for us. Of course, I spent a lot of time trying to identify the reason of this lag. It can be a topic for another post, but cutting a long story shorter - the GPU is shared between applications and OS itself. Each application can have multiple GPU contexts (EAGLContext). In GPGPU you can get result of your operation using render-to-texture technique. To get valid results you should call **glFlush** and **glFinish** commands before you access your texture. These calls are blocking operations - they force the CPU to wait until GPU pipeline is empty. This synchronization can take really long time. In our case it took almost 10 ms per frame. In addition we have found that low-end devices like iPhone touch or iPhone 4 does not have GPU powerful enough. So we decided to look for an alternative approach.

h2 NEON comes in scene

p It was logical that rendering and GPGPU-processing would fight each other for resources. So why not to separate these tasks? Let the whole GPU be occupied with the scene rendering and utilize CPU for image processing. Starting from this point I cannot reveal particular optimization details. But in general we re-designed existing algorithm to process less data and re-wrote most heavy functions with assembly language using NEON SIMD instructions. This was a win:
    ul
        li iPhone3Gs: 24ms per frame
        li iPod touch: 17 ms per frame
        li iPad 2: 8ms per frame
        li iPhone 4S: 8 ms per frame

p Remember I was talking about speed-quality trade-off? Since we had enough spare time we could spent it to compute final basket pose more precisely. In particular, our algorithm is choosing the optimal pose estimation parameters depending on the current hardware ARBasketball is running on. On low-end hardware pose estimation is not too-precise, but anyway it is still providing smooth 30 FPS gameplay. On modern devices we have more power and can do more work. This adaptive parameter tuning is really helpful when you have to secure the same level of performance on many devices.

h2 Conclusion

p Optimize wisely and remember about your environment. Your application cannot not always have 100% of hardware power, besides there are other tasks that require CPU and GPU power: user-interaction handling, 3D rendering, audio decoding, device motion updates, notifications, network, physic simulation, threading. Keep in mind that algorithm you optimize can (and it will) interfere to the environment. It was very interesting to work with Konstantin on ARBasketball and I'm glad I had this opportunity.

p Recently, Paladone has announced [ARBasketball Mug][5]. A coffee/tea mug with printed marker on it's side that allow you to play ARBasketball. Really amazing idea. By the way, I've already got mine ;)

   [1]: https://itunes.apple.com/ru/app/arbasketball-augmented-reality/id393333529?mt=8
   [2]: http://docs.opencv.org/modules/imgproc/doc/miscellaneous_transformations.html?highlight=threshold#adaptivethreshold
   [3]: https://github.com/BradLarson/GPUImage
   [4]: 
   [5]: http://www.ebay.com/itm/AR-BASKETBALL-APP-MUG-Augmented-Reality-Tea-Coffee-Mug-for-iPhone-5-4-3-iPad-/350786629462?pt=UK_HG_Crockery_RL&hash=item51ac832f56
   [6]: 
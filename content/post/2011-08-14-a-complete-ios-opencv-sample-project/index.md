+++
title =  "A complete iOS + OpenCV sample project"
date = "2011-08-14"
tags =  ["opencv", "xcode"]
+++

![OpenCV iOS Sample Project][1]Hello everyone! Today i want to introduce the all new tutorial project of using OpenCV in your iOS projects. In this post i'll show you the right and correct way of interoperation between native OpenCV C/C++ API and Objective-C. I know many of you asked me how to solve this nasty "statement-expressions are allowed only inside functions" error. Here is a solution.

<span class="more"></span>

The conflict occurs because both UIKit.h and opencv_core.h define the same MIN symbol. But in UIKit it’s a function, and a macro in OpenCV. If you include bot iOS framework and core.h header file from OpenCV they will conflict with MIN symbol because iOS define MIN function, but the OpenCV has the MAX macros. The simple way would be to ask OpenCV maintainers to give other name to MIN macros. But it's too easy and i do not believe it's right solution.

The correct way is to split the business logic from the presentation layer into different parts of the project, so the iOS frameworks and OpenCV includes never appears in single compilation unit.

In this tutorial i consider a sample application, which takes images from the photo album, does some image processing and then displays the processed bitmaps to the user. So the idea to differentiate the UI from the image processing.

Let's do it in this way.

  1. Declare the API interface used by the View. This API is the only thing that View know about. 
  2. Write and particular C++ implementation of the image processing component. 
  3. Write an implementation of the API protocol that implements desired methods and delegates calls to the C++ implementation doing data conversion if necessary. 

A small diagram that illustrates the idea:

![OpenCV  iOS Architecture][2]

Here is a code of the protocol declaration:
  
```objectivec
@protocol ImageProcessingProtocol < NSObject >
- (UIImage*) processImage:(UIImage*) src;
@end
```

As you can see it's pretty simple - it sends the image to processing component and return the result. I pay your attention to the fact that user code at this point know nothing about OpenCV. In the view you work with this API like this: 
    
```objectivec
- (void)imagePickerController:(UIImagePickerController *)picker
        didFinishPickingImage:(UIImage *)image
                  editingInfo:(NSDictionary *)editingInfo
{
  [picker dismissModalViewControllerAnimated:YES];

  iOSplusOpenCVAppDelegate * appDelegate = [[UIApplication sharedApplication] delegate];

  // Process the input image and present the result:
  UIImage * processedImage = [appDelegate.imageProcessor processImage:image];
  self.mainButton.imageView.image = processedImage;
}
```

Now let's write the actual processing code. In this project I use Boost.Gil to pass raw image data to OpenCV because I prefer this strongly typed image concepts very much and use Boost for it is a good choice. You can read about Boost.Gil here: [Boost Gil library overview][3]. 

Declaration of OpenCVImageProcessor class:
    
```cpp
struct _IplImage;
typedef _IplImage IplImage;

class OpenCVImageProcessor
{
public:
    void process(boost::gil::bgr8_view_t src, boost::gil::bgr8_view_t dst);

private:
    IplImage getIplImageView(boost::gil::bgr8_view_t srcView);
    IplImage getIplImageView(boost::gil::gray8_view_t srcView);
};
```

Implementation:
    
```cpp
// A helper macro
#define GIL2CV(GilView) getIplImageView(GilView)

void OpenCVImageProcessor::process(boost::gil::bgr8_view_t src, boost::gil::bgr8_view_t dst)
{
    BOOST_ASSERT(src.dimensions() == dst.dimensions());

    boost::gil::gray8_image_t srcGray(src.dimensions());
    boost::gil::gray8_image_t edges(src.dimensions());

    IplImage srcIplImage     = GIL2CV(src);
    IplImage srcGrayIplImage = GIL2CV(srcGray._view);
    IplImage edgesIplImage   = GIL2CV(edges._view);
    IplImage dstIplImage     = GIL2CV(dst);

    cvCvtColor(&srcIplImage;, &srcGrayIplImage;, CV_BGR2GRAY);

    cvCanny(&srcGrayIplImage;, &edgesIplImage;, 10, 50);

    cvCvtColor(&edgesIplImage;, &dstIplImage;, CV_GRAY2BGR);
}
```

Almost done. Now we have write and implementation for the ImageProcessingProtocol (I omit the image conversion functions because they are pretty standard and simple): 
    
```objectivec    
@interface ImageProcessingImpl : NSObject
- (UIImage*) processImage:(UIImage*) src;
@end

@implementation ImageProcessingImpl

- (UIImage*) processImage:(UIImage*) src
{
  OpenCVImageProcessor processor;

  boost::gil::bgr8_image_t srcImage = [self convertUIImageToGILImage:src];
  boost::gil::bgr8_image_t dstImage(srcImage.dimensions());

  processor.process(view(srcImage), view(dstImage));

  return [self convertGILImageToUIImage:dstImage];
}

@end
```

And that’s all. With this approach you receive flexible design with strict API and platform-independent implementation. You probably can even port it to Android platform by simply writing an adopted ImageProcessingAndroidImpl wrapper.

The final application looks like on this screenshots:

![1][4]

At startup user asked to choose the source image by tapping on screen and then the detected edges are shown. To choose another image – just tap on the screen again.

**Advantages:**

  * No explicit dependency from OpenCV in your UI. 
  * No violation of MVC and SRP paradigm. 
  * The View code looks more clear and easy to understand. 
  * Really cross-platform solution. 

**Disadvantages:**

  * Need to write a bit more code. 
  * You need some proxy types to pass Obj-C data (images) to the OpenCV. I use Boost.Gil for this. 

#### **Conclusion:**

The source code for this sample can be found on my GitHub: <https://github.com/BloodAxe/opencv-ios-template-project>.

   [1]: img_thumb.png (OpenCV iOS Sample Project)
   [2]: OpenCV-iOS-Architecture_thumb.png (OpenCV  iOS Architecture)
   [3]: http://www.boost.org/doc/libs/1_46_1/libs/gil/doc/index.html
   [4]: 1_thumb.png (1)

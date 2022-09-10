+++
title = "OpenCV Tutorial - Part 1"
date = "2012-06-23"
tags =  ["opencv","xcode", "tutorials"]
+++


As i recently mentioned, i decided to write a brand new OpenCV tutorial application for iPhone/iPad devies. This development is open-source and anyone can access it on <https://github.com/BloodAxe/OpenCV-Tutorial> repository page. Your help are welcome to write a UI for this app and help writing sample demonstration cases. Feel free to clone repository and make your contribution! 

<!--more-->

## OpenCV Tutorial

The application startup screen will present master-detail view as a list of available samples. The master table view will present a list of samples available, while the detailed view - a sample description and it's icon perhaps. For each sample a still image can be used as a input data for processing. Also, processing of a video source from device camera will be also supported. We start from creating a new XCode project as shown on next figure: ![][1] Then we link OpenCV library with this project. Here is a brief task list: 1) Get a pre-build or build your own OpenCV static library using OpenCV build script and copy it to the same folder where XCode project is located like this: ![][2] 2) Add $(SRCROOT)/opencv to header search path and $(SRCROOT)/opencv/lib/debug for library search path for debug configuration and $(SRCROOT)/opencv/lib/release for release build. 3) Add OpenCV libs to linker input by modifying "Other Linker Flags" option with "-lopencv_calib3d -lzlib -lopencv_contrib -lopencv_legacy -lopencv_features2d -lopencv_imgproc -lopencv_video -lopencv_core". 4) Add OpenCV headers to precompiled header file like that: 
    
```cpp
#ifdef __cplusplus
#include <opencv2/opencv.hpp>
#endif

#ifdef __OBJC__
  #import <UIKit/UIKit.h>
  #import <Foundation/Foundation.h>
#endif
```

Please note that OpenCV include directive placed before import of iOS frameworks. It's important to keep this order, otherwise you'll get compilation errors caused by conflicts of MIN symbol. 

## Sample interface

Each demonstration of particular algorithm usually required some image as an input argument (which is obviously since we working with image processing, right?). To reflect this nature of our application and to make our application really flexible we introduce a base class for all sample scenarios. Derived classes should implement methods that provides generic information about this example and main processing routine. Here are it's signature: 
    
```cpp
//! Base class for all samples
class SampleBase
{
public:
  //! Gets a sample name
  virtual std::string getName() const = 0;

  //! Returns a detailed sample description
  virtual std::string getDescription() const = 0;

  //! Returns true if this sample requires setting a reference image for latter use
  virtual bool isReferenceFrameRequired() const = 0;

  //! Sets the reference frame for latter processing
  virtual void setReferenceFrame(const cv::Mat& reference) = 0;

  //! Processes a frame and returns output image 
  virtual bool processFrame(const cv::Mat& inputFrame, cv::Mat& outputFrame) = 0;

  const std::vector& getOptions();

protected:
  void registerOption(std::string name, bool  * value);
  void registerOption(std::string name, int   *  value, int min, int max);
  void registerOption(std::string name, float *  value, float min, float max);

private:
  std::vector m_options;
};
```

The processFrame function takes input image as first argument, performs some image processing and puts the result to the outputFrame argument. Our application can use either images taken with a camera or photo gallery or it can process frames from a camera in real-time mode.

Of course, almost every algorithm can have adjustable parameters that affects the processing result. To reflect this we introduce a bunch of registerOption functions that allows to expose parameters that can be modified in runtime.

In Part 2 we will create a Master-Detail view to present our samples on the iPad and iPhone screen and write first sample - Edge Detection Sample.

   [1]: Screen-Shot-2012-06-23-at-2.59.18-PM.png (Master-detail XCode template project)
   [2]: Screen-Shot-2012-06-23-at-8.40.23-PM.png (Screen Shot 2012-06-23 at 8.40.23 PM)
   [4]: http://developer.apple.com/library/ios/#documentation/userexperience/conceptual/mobilehig/IconsImages/IconsImages.html (iOS Human Interface Guidelines)
+++
title =  "OpenCV Tutorial - Part 3"
date = "2012-06-27"
tags =  ["opencv", "xcode", "tutorials"]
+++

In [Part 1][1] and [Part 2][2] we created base application for our "OpenCV Tutorial" application. In this part we add video source to process frames using our samples and present the result to user. As usual, you can find source code for this application at [github][3].

## Video capture in iOS

At this moment (as far as i know) there OpenCV's cv::VideoCapture does not support iOS platform. Therefore we have to use iOS AVFoundation API to setup video capture. This is more complicated that write cv::VideoCapture("YourVideoFileName.avi") but it's not a rocket science.  There is a great Apple documentation article [AV Foundation Programming Guide][4] that i strongly advice you to read. **By the way, before I forget - video capture is not supported on iOS simulator. You'll need real device to test your app!** To manage the capture from a device such as a camera or microphone, you assemble objects to represent inputs and outputs, and use an instance of `AVCaptureSession` to coordinate the data flow between them. Minimally you need: 

  * An instance of `AVCaptureDevice` to represent the input device, such as a camera or microphone
  * An instance of a concrete subclass of `AVCaptureInput` to configure the ports from the input device
  * An instance of a concrete subclass of `AVCaptureOutput` to manage the output to a movie file or still image
  * An instance of `AVCaptureSession` to coordinate the data flow from the input to the output
  
To put all things together we introduce VideoSource class which incapsulate initialization and video capture routine. Let's take a look on it's interface: 
    
```objectivec
@protocol VideoSourceDelegate 

- (void) frameCaptured:(cv::Mat) frame;

@end

@interface VideoSource : NSObject

@property id delegate;

- (bool) hasMultipleCameras;
- (void) toggleCamera;
- (void) startRunning;
- (void) stopRunning;

@end
```

The VideoSourceDelegate protocol defines a callback procedure that user code can handle. The frameCaptured method is called when the frame is received from a camera device. We wrap it in cv::Mat structure for latter use. Initialization of VideoCamera is also not complicated: we create capture session, add video input and video output: 
    
```objectivec
- (id) init
{
  if (self = [super init])
  {
    currentCameraIndex = 0;
    session = [[AVCaptureSession alloc] init];
    [session setSessionPreset:AVCaptureSessionPreset640x480];
    captureDevices = [AVCaptureDevice devicesWithMediaType:AVMediaTypeVideo];

    AVCaptureDevice *videoDevice = [captureDevices objectAtIndex:0];

    NSError * error;
    captureInput = [AVCaptureDeviceInput deviceInputWithDevice:videoDevice error:&error];

    if (error)
    {
      NSLog(@"Couldn't create video input");
    }

    captureOutput = [[AVCaptureVideoDataOutput alloc] init];
    captureOutput.alwaysDiscardsLateVideoFrames = YES; 

    // Set the video output to store frame in BGRA (It is supposed to be faster)
    NSString* key = (NSString*)kCVPixelBufferPixelFormatTypeKey; 
    NSNumber* value = [NSNumber numberWithUnsignedInt:kCVPixelFormatType_32BGRA]; 
    NSDictionary* videoSettings = [NSDictionary dictionaryWithObject:value forKey:key]; 
    [captureOutput setVideoSettings:videoSettings];    

    /*We create a serial queue to handle the processing of our frames*/
    dispatch_queue_t queue;
    queue = dispatch_queue_create("com.computer-vision-talks.cameraQueue", NULL);
    [captureOutput setSampleBufferDelegate:self queue:queue];
    dispatch_release(queue);

    [session addInput:captureInput];
    [session addOutput:captureOutput];
  }

  return self;
}
```

Please not that we query all video devices available using the [AVCaptureDevice devicesWithMediaType:AVMediaTypeVideo] call to get them all. This feature allows us to toggle between front and rear cameras in runtime. Also we set capture session preset to 640x480 because the larger the image the more it it need to be processed by our algorithms. 640x480 is a good choice. As our last step we configure capture output to pass our frames in BGRA format. For our image processing it's the ideal case. In case when our video source has several video inputs we can toggle between them using toggleCamera method: 
    
```objectivec    
- (void) toggleCamera
{
  currentCameraIndex++;
  int camerasCount = [captureDevices count];
  currentCameraIndex = currentCameraIndex % camerasCount;

  AVCaptureDevice *videoDevice = [captureDevices objectAtIndex:currentCameraIndex];

  [session beginConfiguration];

  if (captureInput)
  {
    [session removeInput:captureInput];
  }

  NSError * error;
  captureInput = [AVCaptureDeviceInput deviceInputWithDevice:videoDevice error:&error];

  if (error)
  {
    NSLog(@"Couldn't create video input");
  }

  [session addInput:captureInput];
  [session setSessionPreset:AVCaptureSessionPreset640x480];
  [session commitConfiguration];
}
```

In this function we cycle through available inputs and add them to capture session (do not forget to remove previous input) and set the capture preset again (just in case). Switching between from and rear camera is initiated by the user by tapping on toggle button (we will talk about it a bit later). Just one thing left - we should implement AVCaptureVideoDataOutputSampleBufferDelegate in our VideoSource class to receive frames from AVFoundation API like this: 
    
```objectivec
- (void)captureOutput:(AVCaptureOutput *)captureOutput 
didOutputSampleBuffer:(CMSampleBufferRef)sampleBuffer 
       fromConnection:(AVCaptureConnection *)connection 
{ 
  if (!delegate)
    return;

  CVImageBufferRef imageBuffer = CMSampleBufferGetImageBuffer(sampleBuffer); 

  /*Lock the image buffer*/
  CVPixelBufferLockBaseAddress(imageBuffer,0); 

  /*Get information about the image*/
  uint8_t *baseAddress = (uint8_t *)CVPixelBufferGetBaseAddress(imageBuffer); 
  size_t width = CVPixelBufferGetWidth(imageBuffer); 
  size_t height = CVPixelBufferGetHeight(imageBuffer);  
  size_t stride = CVPixelBufferGetBytesPerRow(imageBuffer);
  //NSLog(@"Frame captured: %lu x %lu", width,height);

  cv::Mat frame(height, width, CV_8UC4, (void*)baseAddress);

  [delegate frameCaptured:frame];

  /*We unlock the  image buffer*/
  CVPixelBufferUnlockBaseAddress(imageBuffer,0);
}
```

In this method we obtain raw data in BGRA format from CoreMedia and put them input cv::Mat (no data is copied here). And then we call [delegate frameCaptured:] to inform user code that new frame is available. 

## Displaying processed frames

Since we working with video processing it's obviously to present the result of video processing in real-time on the screen. To secure the appropriate FPS we have to redraw our screen fast. The existing UIImageView control cannot be used here because we'll spent to much time on unnecessary image conversion (cv::Mat to UIImage) and drawing UIImage on the ImageView. For our goal the fastest way to draw the picture is to use OpenGL ES and GLView to draw our bitmap using GPU. In this section i'll show you how to create simple view that able to draw cv::Mat in real-time. To secure this'll UIView and to EAGLContext using it. This allows us to use OpenGL API to draw textured rectangle on top of our view as fast as possible. For user code we expose only single function that is can use: 
    
```objectivec
@interface GLESImageView : UIView

- (void)drawFrame:(const cv::Mat&) bgraFrame;

@end
```

   [1]: http://computer-vision-talks.com/2012/06/opencv-tutorial-a-collection-of-opencv-samples-for-iphoneipad-part-1/ (OpenCV Tutorial – a collection of OpenCV samples for iPhone/iPad – Part 1)
   [2]: http://computer-vision-talks.com/2012/06/opencv-tutorial-part-2/ (OpenCV Tutorial – Part 2)
   [3]: https://github.com/BloodAxe/OpenCV-Tutorial (OpenCV Tutorial Repository)
   [4]: http://developer.apple.com/library/ios/#DOCUMENTATION/AudioVideo/Conceptual/AVFoundationPG/Articles/00_Introduction.html (AV Foundation Programming Guide)


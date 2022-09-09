+++
title = "Maximizing performance of CV_BGRA2GRAY conversion using NEON and cv::parallel_for"
date = "2012-11-06"
tags =  ["opencv", "xcode", "neon", "iphone"]
+++

I continue playing with powerful NEON engine in iPhone and iPad devices. Recently i bought iPhone 4S that replaced my HTC Mozart and i decided to check how to speed up BGRA to GRAY color conversion procedure using multithreading. 
<!--more-->

Recently Itseez announced a minor release of OpenCV 2.4.3 with a lot of new major features: 

  * Added universal parallel_for implementation using various backends: TBB, OpenMP, GCD, Concurrency
  * Improved OpenCV Manager, new Java samples framework, better camera support on Android,
  * opencv2.framework is now iOS6- and iPhone5- (armv7s) compatible.

The rest of changes can be seen here: <http://code.opencv.org/projects/opencv/wiki/ChangeLog>. In the past i wrote a NEON-optimized grayscale conversion algorithm (<http://computer-vision-talks.com/2011/02/a-very-fast-bgra-to-grayscale-conversion-on-iphone/>) which showed a pretty nice speed-up (About 2x times faster than cv::cvtColor(input, output, CV_BGRA2GRAY)). In this post we will use these results as a baseline and try to write something faster. To be fair we start from defining a set of rules for all implementations: 

  * We will process square images of size (8, 10, 12, 14, 16, 18, ...., 1596, 1598, 1600) pixels. Processing small and big images will reveal hidden bottlenecks of each implementation.
  * Each image will be processed 1000 to avoid measurement fluctuations.
  * The destination image will be preallocated prior the test to exclude memory allocations influence.
  * We use mach_absolute_time() to measure processing time. As far as i know it's the most precise measuring method.

Our main test method looks like this: 
    
```cpp 
typedef void ConversionFunctionPrototype(const cv::Mat& input, cv::Mat& output);

void testConversion(const cv::Mat& input, int numRuns, int& avgTimeInMicroseconds, ConversionFunctionPrototype conversionFn)
{
    assert(input.type() == CV_8UC4);
    assert(input.empty() == false);

    cv::Mat output(input.size(), CV_8UC1);

    uint64_t conversionStart, conversionFinish;
    uint64_t totalTime = 0;

    for (int i = 0; i < numRuns ; i++)
    {
        conversionStart = mach_absolute_time();
        conversionFn(input, output);
        conversionFinish = mach_absolute_time();

        totalTime += (conversionFinish - conversionStart);
    }

    mach_timebase_info_data_t timebase;
    mach_timebase_info(&timebase);

    avgTimeInMicroseconds = (int) ((double)totalTime * (double)timebase.numer / ((double)timebase.denom * (double)numRuns));
}
```

The testConversion function computes the average BGRA2GRAY conversion time of the input image by doing color conversion N times (specified by numRuns argument). 

## OpenCV implementation

The reference color conversion function from OpenCV: 
    
```cpp      
static void convert_opencv(const cv::Mat& input, cv::Mat& output)
{
    cv::cvtColor(input, output, CV_BGRA2GRAY);
}
```

## ARM NEON using assembler

The old assembler version of NEON-accelerated color conversion from my old post: 
    
```cpp 
static void convert_neon_asm_8px(const cv::Mat& input, cv::Mat& output)
{
    uint8_t __restrict * dest = output.data;
    uint8_t __restrict * src  = input.data;
    int numPixels             = input.total();

    __asm__ volatile("lsr          %2, %2, #3      \n"
                     "# build the three constants: \n"
                     "mov         r4, #28          \n" // Blue channel multiplier
                     "mov         r5, #151         \n" // Green channel multiplier
                     "mov         r6, #77          \n" // Red channel multiplier
                     "vdup.8      d4, r4           \n"
                     "vdup.8      d5, r5           \n"
                     "vdup.8      d6, r6           \n"
                     "0:                           \n"
                     "# load 8 pixels:             \n"
                     "vld4.8      {d0-d3}, [%1]!   \n"
                     "# do the weight average:     \n"
                     "vmull.u8    q7, d0, d4       \n"
                     "vmlal.u8    q7, d1, d5       \n"
                     "vmlal.u8    q7, d2, d6       \n"
                     "# shift and store:           \n"
                     "vshrn.u16   d7, q7, #8       \n" // Divide q3 by 256 and store in the d7
                     "vst1.8      {d7}, [%0]!      \n"
                     "subs        %2, %2, #1       \n" // Decrement iteration count
                     "bne         0b            \n" // Repeat until iteration count is not zero
                     :
                     : "r"(dest), "r"(src), "r"(numPixels)
                     : "r4", "r5", "r6"
                     );
}
```

## ARM NEON using intrinsics

You know you can write the same code using arm intrinsics? Here it is: 
    
```cpp 
static void convert_neon8px_int(const cv::Mat& input, cv::Mat& output)
{
    uint8_t __restrict * dest = output.data;
    uint8_t __restrict * src  = input.data;
    int numPixels             = input.total();

    uint8x8_t rfac = vdup_n_u8 (77);
    uint8x8_t gfac = vdup_n_u8 (151);
    uint8x8_t bfac = vdup_n_u8 (28);

    register int n = numPixels / 8;
    uint16x8_t  temp;
    uint8x8_t result;
    uint8x8x4_t rgb;

    for (register int i = 0; i < n; i++, src += 8*4, dest += 8)
    {
        rgb  = vld4_u8 (src);

        temp = vmull_u8 (rgb.val[0],      rfac);
        temp = vmlal_u8 (temp,rgb.val[1], gfac);
        temp = vmlal_u8 (temp,rgb.val[2], bfac);

        result = vshrn_n_u16 (temp, 8);
        vst1_u8 (dest, result);
    }
}
``` 

## Parallel NEON version

Both **convert_neon_asm_8px** and **convert_neon8px_int** functions process 8 pixels at once. This color conversion function is a great candidate for multithreading optimization. Since iPhone 4S and iPad 2 have two Cortex A8 CPU we can run our color conversion in two threads. For this purpose we want to use Grand Central Dispatch mechanism from iOS. It provides easy mechanism to execute tasks in multithreaded environment. OpenCV developers offers new nice feature to perform parallel calculation - **cv::parallel_for**. This function provides abstraction layer between multithreading backend and user code. To use **cv::parallel_for** you have to implement a cv::ParallelLoopBody functor: 
    
```cpp 
// a base body class
class CV_EXPORTS ParallelLoopBody
{
public:
    virtual ~ParallelLoopBody();
    virtual void operator() (const Range& range) const = 0;
};

CV_EXPORTS void parallel_for_(const Range& range, const ParallelLoopBody& body, double nstripes=-1.);
```

Our color conversion function simply wraps the code from the ARM NEON version with slight modifications. When counting the number of pixels to process we take into account the range argument passed to the function body. 

```
struct convert_ParallelLoopBody : public cv::ParallelLoopBody
{
    convert_ParallelLoopBody(const cv::Mat& input, cv::Mat& output)
    : _in(input)
    , _out(output)
    {
    }

    void operator() (const cv::Range& range) const
    {
        uint8_t __restrict * dest = _out.data + range.start;
        uint8_t __restrict * src  = _in.data  + range.start * 4;

        int numPixels             = range.size();

        uint8x8_t rfac = vdup_n_u8 (77);
        uint8x8_t gfac = vdup_n_u8 (151);
        uint8x8_t bfac = vdup_n_u8 (28);

        register int n = numPixels / 8;
        uint16x8_t  temp;
        uint8x8_t result;
        uint8x8x4_t rgb;

        for (register int i = 0; i < n; i++, src += 8*4, dest += 8)
        {
            rgb  = vld4_u8 (src);

            temp = vmull_u8 (rgb.val[0],      rfac);
            temp = vmlal_u8 (temp,rgb.val[1
        }
    }
}
```

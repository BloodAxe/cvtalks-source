+++
title =  "Integration of KAZE 1.6 in OpenCV"
date = "2014-04-03"
tags = ["opencv", "algorithms"]
+++

<div class="featured-image">
![AKAZE logo][akaze-logo]
</div>


A new version of KAZE and AKAZE features is a good candidate to become a part of OpenCV. 
So i decided to update KAZE port i made a while ago with a new version of these features
and finally make a pull request to make it a part of OpenCV.

<span class="more clearfix" />

<div class="alert alert-info">
<p class="lead">KAZE are now a part of OpenCV library</p>

The OpenCV has accepted my pull-request and merged KAZE port into master branch of the OpenCV library. KAZE and AKAZE features will become available in OpenCV 3.0. Of course, you can grab development branch and build it from scratch to access it now.    
</div>

<p class=lead>
** Looking for source code? It's all there: [KAZE &amp; AKAZE in OpenCV][kaze-branch]. **
</p>

**[TL;DR; Scroll down to estimation charts][results]**.

# Integration roadmap

I'm going to keep KAZE sources intact if possible to simplify their further support. 
Original KAZE and AKAZE implementations will be placed in `kaze/` and `akaze/` folders under `features2/` module and what we want is to write a facade-wrappers for these algorithms.  

To integrate KAZE featues we need to adopt sources code to OpenCV coding guidelines, make consistent with headers include system,
integrate into build system, implement wrapped from KAZE to Features2D API and add unit tests. This will be split into three steps:

 - Adopt KAZE and AKAZE sources (Remove unused functions, fix includes, macros)
 - Implement Features2D wrappers and expose properties for runtime configuration of KAZE. 
 - Add unit tests and remove duplicate functions that are already exists in OpenCV.
 - Replace OpenMP with cv::parallel_for_ 

## Changes in KAZE sources

First, we need to adopt existing sources.

### Step 1 -Update OpenCV includes

Since we're making KAZE a part of the library, it's impossible to reference OpenCV types using standard.

```cpp
#include "opencv2/opencv.h"
```

Instead one may want to use precomp.hpp header like this:

```cpp
#include "precomp.hpp"
```

### Step 2 - Cleaning up the code

p There is a C-style assert(cond) macro that I will replace with CV_Assert for convinience.

#### Dump of KAZE internal structures

KAZE and AKAZE algorithm can 'dump' internal buffers to disk using imwrite function. 
But features2d module can not be available and i assume the goal of this feature was to simplify debugging of KAZE features.
Since we may expect it is mature enough, we will remove these functions (`Save_Scale_Space`, `Save_Detector_Responses`, `Save_Flow_Responses`, `Save_Nonlinear_Scale_Space`) from sources.

#### Fixing the PI constant

KAZE uses `M_PI` symbol to represent Pi number. We will use `CV_PI` replacement instead.

#### Cleanup utils.cpp

Helper file utils.cpp contains auxilar functions that is not used by KAZE directly but rather used for precision esitmation. 
We don't need these functions in OpenCV packages. So we say goodbye to following functions:

```cpp
void draw_keypoints(cv::Mat& img, const std::vector<cv::KeyPoint>& kpts);
int save_keypoints(const std::string& outFile,
                   const std::vector<cv::KeyPoint>& kpts,
                   const cv::Mat& desc, bool save_desc);

void matches2points_nndr(const std::vector<cv::KeyPoint>& train,
                         const std::vector<cv::KeyPoint>& query,
                         const std::vector<std::vector<cv::DMatch> >& matches,
                         std::vector<cv::Point2f>& pmatches, float nndr);
void compute_inliers_ransac(const std::vector<cv::Point2f>& matches,
                            std::vector<cv::Point2f>& inliers,
                            float error, bool use_fund);
void compute_inliers_homography(const std::vector<cv::Point2f>& matches,
                                std::vector<cv::Point2f> &inliers,
                                const cv::Mat&H, float min_error);
void draw_inliers(const cv::Mat& img1, const cv::Mat& img2, cv::Mat& img_com,
                  const std::vector<cv::Point2f>& ptpairs);
void draw_inliers(const cv::Mat& img1, const cv::Mat& img2, cv::Mat& img_com,
                  const std::vector<cv::Point2f>& ptpairs, int color);
void read_homography(const std::string& hFile, cv::Mat& H1toN);
void show_input_options_help(int example);
```

### Step 3 - Fix constant expression bug in compute_derivative_kernels

This is very similar to a bug - a static array get initialized with ksize that is not compile-time defined.

```cpp
void compute_derivative_kernels(cv::OutputArray _kx, 
                                cv::OutputArray _ky,
                                int dx, int dy, int scale) {

    int ksize = 3 + 2*(scale-1);
    ...
    float kerI[ksize];

}
```

We can quickly fix this issue with std::vector:

```cpp
void compute_derivative_kernels(cv::OutputArray _kx, 
                                cv::OutputArray _ky,
                                int dx, int dy, int scale) {

    int ksize = 3 + 2*(scale-1);
    ...
    std::vector<float> kerI(ksize);

}
```

### Step 5 - Wrapping KAZE for OpenCV

OpenCV provides three base types for extending features2d API:

 - `cv::FeaturesDetector`
 - `cv::DescriptorExtractor`
 - `cv::Feature2D`

Since KAZE features provides both detector and descriptor extractor features, we will derive
our class from `cv::Feature2D`. 
First, we should implement helper functions to indicate depth and size of feature descriptor and matcher type:

```cpp
// returns the descriptor size in bytes
int KAZE::descriptorSize() const
{
    return extended ? 128 : 64;
}

// returns the descriptor type
int KAZE::descriptorType() const
{
    return CV_32F;
}

// returns the default norm type
int KAZE::defaultNorm() const
{
    return NORM_L2;
}
```

In order to make OpenCV happy we need to implement three virtual functions from Feature2D: `detectImpl`, `computeImpl` 
and `operator()` also known as **detectAndCompute**.

### Step 6 - Detection of KAZE keypoints:

```cpp
void KAZE::detectImpl(InputArray image, std::vector<KeyPoint>& keypoints, InputArray mask) const
{
    Mat img = image.getMat();
    if (img.type() != CV_8UC1)
        cvtColor(image, img, COLOR_BGR2GRAY);

    Mat img1_32;
    img.convertTo(img1_32, CV_32F, 1.0 / 255.0, 0);

    KAZEOptions options;
    options.img_width = img.cols;
    options.img_height = img.rows;
    options.extended = extended;

    KAZEFeatures impl(options);
    impl.Create_Nonlinear_Scale_Space(img1_32);
    impl.Feature_Detection(keypoints);

    if (!mask.empty())
    {
        cv::KeyPointsFilter::runByPixelsMask(keypoints, mask.getMat());
    }
}
```

Please note that we conver input image to grayscale normalized to [0;1] floating-point image. 
This is a requirement of KAZE algorithm.

### Step 7 - Extraction of KAZE descriptors:

```cpp
void KAZE::computeImpl(InputArray image, 
                       std::vector<KeyPoint>& keypoints, 
                       OutputArray descriptors) const
{
    cv::Mat img = image.getMat();
    if (img.type() != CV_8UC1)
        cvtColor(image, img, COLOR_BGR2GRAY);

    Mat img1_32;
    img.convertTo(img1_32, CV_32F, 1.0 / 255.0, 0);

    cv::Mat& desc = descriptors.getMatRef();

    KAZEOptions options;
    options.img_width = img.cols;
    options.img_height = img.rows;
    options.extended = extended;

    KAZEFeatures impl(options);
    impl.Create_Nonlinear_Scale_Space(img1_32);
    impl.Feature_Description(keypoints, desc);

    CV_Assert(!desc.rows || desc.cols == descriptorSize() && 
              "Descriptor size does not match expected");
    CV_Assert(!desc.rows || (desc.type() & descriptorType()) && 
              "Descriptor type does not match expected");
}
```

Two asserts at the end of the function to ensure that KAZE returns consistent with `descriptorType()` and `descriptorSize()` results.

### Step 8 - Detection and extraction at single call:

```cpp
void KAZE::operator()(InputArray image, InputArray mask,
    std::vector<KeyPoint>& keypoints,
    OutputArray descriptors,
    bool useProvidedKeypoints) const
{
    cv::Mat img = image.getMat();
    if (img.type() != CV_8UC1)
        cvtColor(image, img, COLOR_BGR2GRAY);

    Mat img1_32;
    img.convertTo(img1_32, CV_32F, 1.0 / 255.0, 0);

    cv::Mat& desc = descriptors.getMatRef();

    KAZEOptions options;
    options.img_width = img.cols;
    options.img_height = img.rows;
    options.extended = extended;

    KAZEFeatures impl(options);
    impl.Create_Nonlinear_Scale_Space(img1_32);

    if (!useProvidedKeypoints)
    {
        impl.Feature_Detection(keypoints);
    }

    if (!mask.empty())
    {
        cv::KeyPointsFilter::runByPixelsMask(keypoints, mask.getMat());
    }

    impl.Feature_Description(keypoints, desc);

    CV_Assert(!desc.rows || desc.cols == descriptorSize() && 
              "Descriptor size does not match expected");
    CV_Assert(!desc.rows || (desc.type() & descriptorType()) && 
              "Descriptor type does not match expected");
}
```

### Step 9 - Algorithm configuration

OpenCV provides an option to create and configure algorithm in runtime by it's name. This is done by using special CV_INIT_ALGORITHM marco, that initialize all OpenCV algorithms during startup. Using this macro we register new KAZE and AKAZE algorithms under Feature2D module and expose additional properties that user can change in runtime:

**features2d_init.cpp**:

```cpp
CV_INIT_ALGORITHM(KAZE, "Feature2D.KAZE",
                  obj.info()->addParam(obj, "extended", obj.extended))
	
CV_INIT_ALGORITHM(AKAZE, "Feature2D.AKAZE",
                  obj.info()->addParam(obj, "descriptor_channels", obj.descriptor_channels);
                  obj.info()->addParam(obj, "descriptor", obj.descriptor);
                  obj.info()->addParam(obj, "descriptor_size", obj.descriptor_size))
```              
Please note, that KAZE and AKAZE has much more properties. They (and documentation for them) will be added later. 
                  
### Step 10 - Unit tests

There is a nice feature detectors and descriptors unit testing system in OpenCV. Using it is very simple, but it performs many sanity checks and validates both parths of Features2D API.

**test_keypoints.cpp**:

All we need to do is to add a new unit test suites:

```cpp
TEST(Features2d_Detector_Keypoints_KAZE, validation)
{
    CV_FeatureDetectorKeypointsTest test(Algorithm::create<FeatureDetector>("Feature2D.KAZE"));
    test.safe_run();
}

TEST(Features2d_Detector_Keypoints_AKAZE, validation)
{
    CV_FeatureDetectorKeypointsTest test(Algorithm::create<FeatureDetector>("Feature2D.AKAZE"));
    test.safe_run();
}
```

In addition to simple checks that our implementation does some job, there are more sophisticaed tests to verify rotation and scale invariance of the computed features. 

**test_rotation_and_scale_invariance.cpp**:

```cpp
TEST(Features2d_ScaleInvariance_Detector_KAZE, regression)
{
    DetectorScaleInvarianceTest test(Algorithm::create<FeatureDetector>("Feature2D.KAZE"),
        0.08f,
        0.49f);
    test.safe_run();
}

TEST(Features2d_ScaleInvariance_Detector_AKAZE, regression)
{
    DetectorScaleInvarianceTest test(Algorithm::create<FeatureDetector>("Feature2D.AKAZE"),
        0.08f,
        0.49f);
    test.safe_run();
}
```

### Step 11 - Enabling multithreading

Both, feature detection and extraction stage can be made faster by using multi-threading. 
Fortunately, AKAZE designed very clear and one may are find OpenMP instructions in critical sections:

```cpp
#pragma omp parallel for
for (int i = 0; i < (int)(kpts.size()); i++) {
    Compute_Main_Orientation(kpts[i]);
    Get_SURF_Descriptor_64(kpts[i], desc.ptr<float>(i));
}
```

But OpenCV uses abstraction layer for multithreading called `cv::parallel_for_`. Personally I think it's very wise architectural desing decision since it allows to get rid of specific cavetas for particular threading backends (OpenCV, TBB, Concurrency, GCD, etc). You can read more about using cv::parallel_for_ in one of my [previous posts][parallel_for_post] or visit [OpenCV documentation][parallel_for_docs]. 

For instance, here is how to parallelize building of the nonlinear scale space for AKAZE. The old version of OpenMP version of `Compute_Multiscale_Derivatives` function looked like this:

```cpp
/**
 * @brief This method computes the multiscale derivatives for the nonlinear scale space
 */
void AKAZEFeatures::Compute_Multiscale_Derivatives(void) {

    #pragma omp parallel for
    for (int i = 0; i < (int)(evolution_.size()); i++) {

        float ratio = pow(2.f, (float)evolution_[i].octave);
        int sigma_size_ = fRound(evolution_[i].esigma*options_.derivative_factor / ratio);

        compute_scharr_derivatives(evolution_[i].Lsmooth, evolution_[i].Lx, 1, 0, sigma_size_);
        compute_scharr_derivatives(evolution_[i].Lsmooth, evolution_[i].Ly, 0, 1, sigma_size_);
        compute_scharr_derivatives(evolution_[i].Lx, evolution_[i].Lxx, 1, 0, sigma_size_);
        compute_scharr_derivatives(evolution_[i].Ly, evolution_[i].Lyy, 0, 1, sigma_size_);
        compute_scharr_derivatives(evolution_[i].Lx, evolution_[i].Lxy, 0, 1, sigma_size_);

        evolution_[i].Lx = evolution_[i].Lx*((sigma_size_));
        evolution_[i].Ly = evolution_[i].Ly*((sigma_size_));
        evolution_[i].Lxx = evolution_[i].Lxx*((sigma_size_)*(sigma_size_));
        evolution_[i].Lxy = evolution_[i].Lxy*((sigma_size_)*(sigma_size_));
        evolution_[i].Lyy = evolution_[i].Lyy*((sigma_size_)*(sigma_size_));
    }
}
```

Using `cv::parallel_for_` we introduce an 'invoker' function object that perform a discrete piece of job on small subset of whole data. Threading API does all job on scheduling multithreaded execution among worker threads:

```cpp
class MultiscaleDerivativesInvoker : public cv::ParallelLoopBody
{
public:
    explicit MultiscaleDerivativesInvoker(std::vector<TEvolution>& ev, const AKAZEOptions& opt) 
    : evolution_(ev)
    , options_(opt)
    {
    }


    void operator()(const cv::Range& range) const 
    {
        for (int i = range.start; i < range.end; i++)
        {
            float ratio = pow(2.f, (float)evolution_[i].octave);
            int sigma_size_ = fRound(evolution_[i].esigma * options_.derivative_factor / ratio);

            compute_scharr_derivatives(evolution_[i].Lsmooth, evolution_[i].Lx, 1, 0, sigma_size_);
            compute_scharr_derivatives(evolution_[i].Lsmooth, evolution_[i].Ly, 0, 1, sigma_size_);
            compute_scharr_derivatives(evolution_[i].Lx, evolution_[i].Lxx, 1, 0, sigma_size_);
            compute_scharr_derivatives(evolution_[i].Ly, evolution_[i].Lyy, 0, 1, sigma_size_);
            compute_scharr_derivatives(evolution_[i].Lx, evolution_[i].Lxy, 0, 1, sigma_size_);

            evolution_[i].Lx = evolution_[i].Lx*((sigma_size_));
            evolution_[i].Ly = evolution_[i].Ly*((sigma_size_));
            evolution_[i].Lxx = evolution_[i].Lxx*((sigma_size_)*(sigma_size_));
            evolution_[i].Lxy = evolution_[i].Lxy*((sigma_size_)*(sigma_size_));
            evolution_[i].Lyy = evolution_[i].Lyy*((sigma_size_)*(sigma_size_));
        }
    }

private:
    mutable std::vector<TEvolution> & evolution_;
    AKAZEOptions                      options_;
};

/**
 * @brief This method computes the multiscale derivatives for the nonlinear scale space
 */
void AKAZEFeatures::Compute_Multiscale_Derivatives(void) {

    cv::parallel_for_(cv::Range(0, evolution_.size()), 
                      MultiscaleDerivativesInvoker(evolution_, options_));
}
```

<a name="results"></a>
# KAZE performance


After integration, I ran KAZE and AKAZE using [feature descriptor estimation framework][features_evaluator] to see how they perform. I was really impressed about matching precision on rotation and scaling tests. Look at these self-explaining charts where **AKAZE beats all other features**! 


```html
# TODO: Replace embeddings with static images
<script type="text/javascript" src="//ajax.googleapis.com/ajax/static/modules/gviz/1.0/chart.js">
{"dataSourceUrl":"//docs.google.com/spreadsheet/tq?key=0AuBBvmQlA4pfdGhzcWVFUWhqZkdzb01HTnIwQllIQlE&transpose=0&headers=1&range=A41%3AG78&gid=0&pub=1","options":{"titleTextStyle":{"bold":true,"color":"#000","fontSize":16},"curveType":"","animation":{"duration":0},"width":900,"lineWidth":2,"hAxis":{"useFormatFromData":true,"title":"Rotation degree","minValue":null,"viewWindow":{"max":null,"min":null},"maxValue":null},"vAxes":[{"useFormatFromData":true,"title":"Percent of correct matches","minValue":null,"logScale":false,"viewWindow":{"max":null,"min":null},"maxValue":null},{"useFormatFromData":true,"minValue":null,"logScale":false,"viewWindow":{"max":null,"min":null},"maxValue":null}],"booleanRole":"certainty","title":"Rotation test","height":600,"interpolateNulls":false,"domainAxis":{"direction":1},"legend":"right","focusTarget":"series","annotations":{"domain":{}},"useFirstColumnAsDomain":true,"tooltip":{"trigger":"none"}},"state":{},"view":{},"isDefaultVisualization":false,"chartType":"LineChart","chartName":"Chart 1"}
</script>

<script type="text/javascript" src="//ajax.googleapis.com/ajax/static/modules/gviz/1.0/chart.js">
{"dataSourceUrl":"//docs.google.com/spreadsheet/tq?key=0AuBBvmQlA4pfdGhzcWVFUWhqZkdzb01HTnIwQllIQlE&transpose=0&headers=1&range=A80%3AG98&gid=0&pub=1","options":{"titleTextStyle":{"bold":true,"color":"#000","fontSize":16},"curveType":"","animation":{"duration":0},"width":900,"lineWidth":2,"hAxis":{"useFormatFromData":true,"title":"Scale factor","minValue":null,"viewWindow":{"max":null,"min":null},"maxValue":null},"vAxes":[{"useFormatFromData":true,"title":"Percent of correct matches","minValue":null,"logScale":false,"viewWindow":{"max":null,"min":null},"maxValue":null},{"useFormatFromData":true,"minValue":null,"logScale":false,"viewWindow":{"max":null,"min":null},"maxValue":null}],"title":"Scale invariance","booleanRole":"certainty","height":600,"legend":"right","focusTarget":"series","annotations":{"domain":{}},"useFirstColumnAsDomain":true,"tooltip":{"trigger":"none"}},"state":{},"view":{},"isDefaultVisualization":false,"chartType":"LineChart","chartName":"Chart 1"}
</script>

<script type="text/javascript" src="//ajax.googleapis.com/ajax/static/modules/gviz/1.0/chart.js">
{"dataSourceUrl":"//docs.google.com/spreadsheet/tq?key=0AuBBvmQlA4pfdGhzcWVFUWhqZkdzb01HTnIwQllIQlE&transpose=0&headers=1&range=A30%3AG39&gid=0&pub=1","options":{"titleTextStyle":{"bold":true,"color":"#000","fontSize":16},"series":{"0":{"hasAnnotations":true}},"curveType":"","animation":{"duration":0},"width":900,"lineWidth":2,"hAxis":{"title":"Half of the gaussian kernel size","useFormatFromData":true,"minValue":null,"viewWindow":{"max":null,"min":null},"maxValue":null},"vAxes":[{"title":"Percent of correct matches","useFormatFromData":true,"minValue":null,"viewWindow":{"max":null,"min":null},"logScale":false,"maxValue":null},{"useFormatFromData":true,"minValue":null,"viewWindow":{"max":null,"min":null},"logScale":false,"maxValue":null}],"booleanRole":"certainty","title":"Robustness to blur","height":600,"legend":"right","focusTarget":"series","useFirstColumnAsDomain":true,"tooltip":{"trigger":"none"}},"state":{},"view":{},"isDefaultVisualization":true,"chartType":"LineChart","chartName":"Chart 3"}
</script>

<script type="text/javascript" src="//ajax.googleapis.com/ajax/static/modules/gviz/1.0/chart.js">
{"dataSourceUrl":"//docs.google.com/spreadsheet/tq?key=0AuBBvmQlA4pfdGhzcWVFUWhqZkdzb01HTnIwQllIQlE&transpose=0&headers=1&range=A2%3AG28&gid=0&pub=1","options":{"titleTextStyle":{"bold":true,"color":"#000","fontSize":16},"curveType":"","animation":{"duration":0},"width":900,"lineWidth":2,"hAxis":{"title":"Change of brightness","useFormatFromData":true,"minValue":null,"viewWindow":{"max":null,"min":null},"maxValue":null},"vAxes":[{"title":"Percent of correct matches","useFormatFromData":false,"formatOptions":{"source":"inline"},"minValue":null,"format":"0.##","logScale":false,"viewWindow":{"max":null,"min":null},"maxValue":null},{"useFormatFromData":true,"minValue":null,"viewWindow":{"max":null,"min":null},"maxValue":null}],"title":"Brightness invariance","booleanRole":"certainty","height":600,"legend":"right","focusTarget":"series","useFirstColumnAsDomain":true,"tooltip":{"trigger":"none"}},"state":{},"view":{},"isDefaultVisualization":true,"chartType":"LineChart","chartName":"Chart 4"}
</script>
```



# References

 - [KAZE Integration branch on GitHub][kaze-branch]
 - [KAZE 1.6 implementation](http://www.robesafe.com/personal/pablo.alcantarilla/code/kaze_features_1.6.0.tar.gz)
 - [AKAZE 1.2 implementation](http://www.robesafe.com/personal/pablo.alcantarilla/code/akaze_features_1.1.0.tar.gz)

[kaze-branch]: https://github.com/BloodAxe/opencv/tree/kaze
[akaze-logo]: akaze.jpg
[parallel_for_post]: /articles/2012-11-06-maximizing-performance-grayscale-color-conversion-using-neon-and-cvparallel_for/
[parallel_for_docs]: http://answers.opencv.org/question/3730/how-to-use-parallel_for/
[features_evaluator]: https://github.com/BloodAxe/OpenCV-Features-Comparison
[results]: #results
[opencv-pull-request]: https://github.com/Itseez/opencv/pull/2673
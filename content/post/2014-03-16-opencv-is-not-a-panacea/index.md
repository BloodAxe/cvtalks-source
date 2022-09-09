+++
title =  "OpenCV is not a panacea"
date =  "2014-03-16"
tags =  [ "opencv" ]
+++


Perhaps, someone may find this post provocative or offensive. But in fact it's not. 
Very often i receive offers from all kind of CXX (CEO, CTO, COO, C-bla-bla-bla) that can be formulated like "We want to build product X using OpenCV".
What's wrong with you guys? OpenCV is not a panacea. In this post i'll try to reveal this myth.

Although OpenCV does a great help on getting proof-of-concept software that every start-up needs most of all at early stages, it can make a nightmare for developers in production stage.
The first thing to remember - OpenCV is an open-source project, but there is an official maintainer Itseez company. 
From one size this looks fine since you may expect that library development will not be abandoned one day. 
From the other side do you think it's a good idea to build your product around a product of another company that you cannot control? 
No doubts, Itseez doing it best to fix bugs and add new features to OpenCV, but there is no SLA between you and OpenCV's maintainers on terms of bug-fixing.
Even more - sometimes new releases (with critical bugfixes) break backward compatibility. 

p
    | Here are real-world issues related to OpenCV that I have encountered in past:

    ul
        li Implicit element type of the cv::Mat.
        li Sick way of passing arguments to some functions. cv::Mat(2,4, CV_32FC1) != cv::Mat(1, 4, CV_32FC2).        
        li Somewhere in between 2.1 and 2.2 Lukas-canade optical flow (LKT tracker) algorithm returned different tracking results (coordinates of tracked points were slightly different).
        li A cv::findHomography function clould to return empty homography matrix (zero rows, zero cols) in ~2.4. 
        li Inconsistent ownership transfer with cv::Mat and cv::InputArray.
        li Lack of OpenGL rendering on OSX platform (while CMake report everything's fine).
        li Did ANYONE managed to get cv::VideoWriter working?

        li 
            | What will happen in this case (yes, the size of the destination matrix is 65x65):
            pre.
                cv::Mat src(480, 640, CV_8UC1);
                src(cv::Rect(10,10, 64, 64)) = cv::Mat::eye(65,65, CV_8UC1);
            | And in this:

            pre.
                cv::Mat src(480, 640, CV_8UC1);
                cv::Mat roi = cv::Rect(10,10, 64, 64);
                cv::Mat::eye(65,65, CV_8UC1).copyTo(roi);

        li.
            OpenCV's implementation of the SURF features are much weaker (and slower) than OpenSURF.

        li.
            cv::Ptr<T> ? O'RLY? What about std::shared_ptr and boost::shared_ptr?

        li.
            OpenCV's non-linear solver used for homography and PnP problem gives bad precision and poor stability. Reason? Straighforward implementation. 
            No special care taken to deal with numeric precision. 

    | If your code is heavily coupled with OpenCV data types or API calls - you're in trouble. 
    | If you don't use regression testing for your project you're in a big trouble - there is a chance that you'll know that your product isn't working anymore form customers.

p.lead Encapsulate OpenCV calls into a kind of service.

p.
    A sign of a good design when you able to change the implementation of this service easily. 
    Imagine that you replace OpenCV with EgmuCV for instance. How much work has to be done for this? 

h1 Avoid 'Not invented here' 

p.
    Don't get me wrong, for many cases using OpenCV is ok. Even more, use OpenCV unless you have strong reasons to implement Lev-Mar algorithm by yourself.
    Personally I'm using OpenCV if I want to make a quick proof-of-concept application. 
    However, for performance-critical applications sometimes you have to throw OpenCV away and write it from scratch. 
    Motivation to make this step could vary from low-level code optimization using Assembly language for example.

p
    | It's a bad sign when third-party libraries affect the design of your application. 
    strong Don't let the cv::Mat spread across your codebase! 
    | OpenCV is a external dependency so try to keep it deep inside and don't expose it's API to public code please. 
    | There is a big temptation to use cv::Mat for storing images, matrices everywhere. Think twice before doing that. 
    | A question that you ask yourself before doing this - "how much code I'll have to change if we replace OpenCV with something other someday?".


p.
    If you're looking for a good containers for matrices, please take a look on Eigen library or boost::ublas. 
    You may want to use Eigen for storing images too since it guarantee memory alignment and there is a nice benefit on accessing aligned pointers.
    Also take a look on boost::gil library.

h1 So how to hide OpenCV?

p
    | It's relatively simple with a facade in your business logic. Suppose one may want to have a function to compute strong corners on the input image. 
    | The implementation can looks like this (The Image8U and Corner are used-defined types):

    pre.
        int ComputeStrongCorners(Image8U& src, std::vector<Corner>& corners)
        {
            corners.clear();

            #if HAS_NEON_SUPPORT
                return ComputeStrongCornersNeon(src, corners);
            #elif HAS_AVX_SUPPORT 
                return ComputeStrongCornersAVX(src, corners);
            #elif HAS_OPENCL_SUPPORT
                return ComputeStrongCornersOpenCL(src, corners);        
            #endif

            return ComputeStrongCornersDefault(src, corners);
        }

        int ComputeStrongCornersDefault(src, corners)
        {
            cv::goodFeaturesToTrack( Map< cv::Mat >(src), Map< std::vector<cv::Keypoint> >(corners) );
        }

    | This code snippet illustrate an idea of compile-time call dispatching to particular implementation. 
    | Pleas note, that public API of ComputeStrongCorners function does not expose OpenCV types to the user code. 
    | During build compiler will use one of the optimized routines if they are available or default one. 
    | The Map<T> class is a service class for mapping user types to OpenCV and vice versa. 
    | I'll publish a post on how to map user structures to OpenCV without data copy some day.

h1 OpenCV is a trend, nothing more

p.lead
    | It's important to remember that OpenCV is just a library with a set of standard functions, nothing more. 
    | No one is looking for 'STL programmer', but why 'OpenCV programmer' is a top trending search in linked-in?
p
    | Dozens of examples on object recognition, tracking, face detection makes a fake illusion of understanding computer vision. 
    | The fact that someone called two functions with right arguments doesn't make this person a computer vision pro.


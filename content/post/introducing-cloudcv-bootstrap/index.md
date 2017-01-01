+++
title =  "Introducing CloudCV bootstrap"
date =  "2015-04-14"
tags =  ["tutorials", "nodejs", "cloudcv", "opencv"]
+++

Here's an open-source ready to use bootstrap project written in Node.js that lets
you to quickly build a REST service to host your image processing and computer vision code
in a cloud environment. 
Please welcome: [cloudcv-bootstrap][cloudcv-bootstrap].

![](cloudcv-bootstrap.jpg)

<span class="more" />

I made this project aside of CloudCV to keep it simple but functionaly. It is self-contained
Node.js project that helps you to get quick results on building and deploying your first 
server-based image processing service. 

## Features

 - Ready to use. No need to download extra dependencies. Just run ``npm install`` and that's all.
 - Built-in REST-API support. As a bonus, a Swagger 2.0 specification file comes too. You can use it as a template to build client SDKs.
 - Shipped with OpenCV 3.0.0 
 - Interopability between C++ and Node.js code 
 - Covered with unit tests
 - Logging support

With cloudcv-bootstrap you can quickly wrap your C++ code into web-service using simple and 
clear syntax:

```js
app.post('/api/v1/image/analyze/dominantColors/', function (req, res) {
    cv.analyzeImage(req.files.image.buffer, function(error, result) {
        res.setHeader("Content-Type", "application/json");
        res.write(JSON.stringify(MapAnalyzeResult(result)));
        res.end();
    });
});
```

Error handling and logging here omited for the sake of simplicity, but this is full-functional snippet. 
It accepts uploaded image using POST request and transfers image data to C++ backend. 
[cloudcv-bootstrap][cloudcv-bootstrap] fully follows Node.js programming paradigm and schedule C++ code on libuv thread pool and leave main thread free for requests processing.

## Quick start

```bash
git clone https://github.com/CloudCV/cloudcv-bootstrap.git
npm install
npm start &
curl localhost:3000/api/v1/image/analyze/dominantColors?image=https%3A%2F%2Fraw.githubusercontent.com%2FCloudCV%2Fcloudcv-bootstrap%2Fmaster%2Ftest%2Fdata%2Fopencv-logo.jpg
```

Produces:

```bash
{
    "aspect":
    {
        "width":599,
        "height":555
    },
    "size":
    {
        "width":0,
        "height":0
    },
    "dominantColors":
    [
        {"color":[252,252,252],"totalPixels":201655,"interclassVariance":7.83907795204613e-37,"error":0},
        {"color":[252,0,0],"totalPixels":43612,"interclassVariance":7.83907795204613e-37,"error":0},
        {"color":[0,0,252],"totalPixels":43591,"interclassVariance":7.83907795204613e-37,"error":0},
        {"color":[0,252,0],"totalPixels":43587,"interclassVariance":7.83907795204613e-37,"error":0}
    ]
}
```

Congratulations, you've just computed dominant colors of the OpenCV logo image:

![](https://raw.githubusercontent.com/CloudCV/cloudcv-bootstrap/master/test/data/opencv-logo.jpg)

## Extending with your code

This module uses node-gyp build system. It produces Node C++ addon and require you to do minimal changes into this module:

1. Have C++ code you want to host
2. Write module binding 
3. Register it
4. Write unit tests

Let's go step by step using camera calibration as example. For quick results we won't reinvent the wheel and use code from OpenCV samples. I will just refactor it slightly. Here's our public interface of calibration algorithm:

```cpp
enum PatternType {
    CHESSBOARD = 0,
    CIRCLES_GRID = 1,
    ACIRCLES_GRID = 2
};

class CameraCalibrationAlgorithm
{
public:
    typedef std::vector<cv::Point3f>               VectorOf3DPoints;
    typedef std::vector<cv::Point2f>               VectorOf2DPoints;
    typedef std::vector<std::vector<cv::Point3f> > VectorOfVectorOf3DPoints;
    typedef std::vector<std::vector<cv::Point2f> > VectorOfVectorOf2DPoints;
    typedef std::vector<cv::Mat>                   VectorOfMat;

    CameraCalibrationAlgorithm(cv::Size patternSize, PatternType type);

    bool detectCorners(const cv::Mat& frame, VectorOf2DPoints& corners2d) const;

    bool calibrateCamera(
        const VectorOfVectorOf2DPoints& gridCorners,
        const cv::Size imageSize,
        cv::Mat& cameraMatrix,
        cv::Mat& distCoeffs
    ) const;

protected:

    // .. plenty of helper methods

private:
    cv::Size                 m_patternSize;
    PatternType              m_pattern;
};
```

We want to wrap it into V8 code. First, we need to register corresponding function that we will expose to JS:

```cpp
void RegisterModule(Handle<Object> target)
{
    // ...

    NODE_SET_METHOD(target, "calibrationPatternDetect", calibrationPatternDetect);
    NODE_SET_METHOD(target, "calibrateCamera",          calibrateCamera);
}
```

Implementation of ``calibrationPatternDetect`` and ``calibrateCamera`` needs to parse input arguments, schedule a task to thread pool and invoke a user-passed callback on completition. 
Marshalling between C++ and V8 is tricky. 
Fortunately, NaN module does a great help on data marshalling. 
To simplity developer's life even more [cloudcv-bootstrap][cloudcv-bootstrap] offers complex data marshalling and argument checking:

```cpp
NAN_METHOD(calibrationPatternDetect)
{
    TRACE_FUNCTION;
    NanEscapableScope();

    Local<Object>   imageBuffer;
    Local<Function> callback;
    cv::Size        patternSize;
    PatternType     pattern;
    std::string     error;

    if (NanCheck(args)
        .Error(&error)
        .ArgumentsCount(4)
        .Argument(0).IsBuffer().Bind(imageBuffer)
        .Argument(1).Bind(patternSize)
        .Argument(2).StringEnum<PatternType>({ 
            { "CHESSBOARD",     PatternType::CHESSBOARD }, 
            { "CIRCLES_GRID",   PatternType::CIRCLES_GRID }, 
            { "ACIRCLES_GRID",  PatternType::ACIRCLES_GRID } }).Bind(pattern)
        .Argument(3).IsFunction().Bind(callback))
    {
        LOG_TRACE_MESSAGE("Parsed function arguments");
        NanCallback *nanCallback = new NanCallback(callback);
        NanAsyncQueueWorker(new DetectPatternTask(
            CreateImageSource(imageBuffer), 
            patternSize, 
            pattern, 
            nanCallback));
    }
    else if (!error.empty())
    {
        LOG_TRACE_MESSAGE(error);
        NanThrowTypeError(error.c_str());
    }

    NanReturnUndefined();
}
```

You may read about NanCheck in separate post: [NanCheck][nan-check].

## Data marshalling

Natively, marshaller supports:

**C++ plain types**:
 - char, unsigned char
 - short, unsighed short
 - int, unsigned int
 - long, unsigned long
 - float, double
 - T[N]

**STL types**:
 - std::array<T,N>
 - std::pair<A,B>
 - std::vector<T>
 - std::map<K,V>

**OpenCV types**:
 - cv::Point2i, cv::Point2f, cv::Point2d
 - cv::Size_<int>, cv::Size_<float>, cv::Size_<double>
 - cv::Mat

Data marshalling of user-defined structures implemented in similar to boost::serialization fashion:

```cpp
struct CalibrationResult
{
    cv::Mat  m_distCoeffs;
    cv::Mat  m_cameraMatrix;
    bool     m_calibrationSuccess;

    template <typename Archive>
    void serialize(Archive& ar)
    {
        ar & serialization::make_nvp("calibrationSuccess", m_calibrationSuccess);

        if (Archive::is_loading::value || m_calibrationSuccess)
        {
            ar & serialization::make_nvp("cameraMatrix",m_cameraMatrix);
            ar & serialization::make_nvp("distCoeffs",  m_distCoeffs);
        }
    }
};
```

User-defined types will be marshalled to regular V8 object containing fields serialized within ``serialize()`` function.

To marshal C++ object to V8 object:

```cpp
CalibrationResult cpp_result = ...;
auto v8_result = marshal(cpp_result);
```

To marshal from V8 object to C++ object:

```cpp
v8::Local<v8::Value> v8_result = ...;
auto cpp_result = marshal<CalibrationResult>(v8_result);
```

## Roadmap

This is very beta version of cloudcv-bootstrap and it's codebase about to change. 
Please keep in mind that and feel free to ask for help in [twitter][cvtalks] or on [GitHub](https://github.com/CloudCV/cloudcv-bootstrap/issues).

According to plan:

1. Add Dockerfile to run this code in a container environment
2. Write more documentation on data marshalling
3. Implement easier REST API mapping and arguments checking

[nan-check]: http://computer-vision-talks.com/articles/how-to-convert-args-from-js-to-cpp
[cloudcv-bootstrap]: https://github.com/CloudCV/cloudcv-bootstrap
[cvtalks]: https://twitter.com/cvtalks
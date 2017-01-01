+++
title =  "Argument checking for native addons for Node.js. Do it right!"
date = "2014-09-11"
tags =  ["cloudcv", "nodejs", "tutorials"]
+++

<div class="featured-image">
![NanCheck](logo.jpg)
</div>

During development of [CloudCV][cloudcv] I came to the problem on converting `v8::Arguments` to 
native C++ data types in my Node.js native module. If you are new to C++ and Node.js, I suggest you to read how to write C++ modules for Node.js and connecting OpenCV and Node.js first.

Mapping V8 data types to native C++ equivalents is trivial, but somewhat wordy. One should take the 
argument at given index, check whether it is defined, then check it's type and finally cast to C++ type. 
This works fine while you have function that receive two or three arguments of trivial type (That can be mapped directly to built-in C++ types). What about strings? Arrays? Complex types like objects or function callback? 
You code will grow like and became hard-to-maintain pasta-code some day. 

In this post I present my approach on solving this problem with a laconic way on describing what do you expect as input arguments.

<span class="more"></span> 
<div class="clearfix"></div>

To illustrate the difference between imperative approach I included source code for calibrationPatternDetect method that expose function to detect calibration pattern on a single image to Node.js code. As you may see below, there are a lot of *if* conditions, magic numbers and no type checking for a half of arguments. But even without it, this function occupy 50 lines of code. 
What even worse, 90% of this code is going to be the same for other functions. The main purpose of code of any `NAN_METHOD` implementation - to marshal data in such a way it can be used by C++ code.

```cpp
NAN_METHOD(calibrationPatternDetect)
{
    NanScope();

    if (args.Length() != 5)
    {
        return NanThrowError("Invalid number of arguments");  
    }

    if (!args[0]->IsObject())
    {
        return NanThrowTypeError("First argument should be a Buffer");      
    }

    // 0 - image
    // 1 - width
    // 2 - height
    // 3 - pattern
    // 4 - callback

    int w  = args[1]->Uint32Value();
    int h  = args[2]->Uint32Value();
    int pt = args[3]->Uint32Value();
    PatternType pattern;

    switch (pt)
    {
        case 0:
            pattern = CHESSBOARD;
            break;

        case 1:
            pattern = CIRCLES_GRID;
            break;

        case 2:
            pattern = ASYMMETRIC_CIRCLES_GRID;
            break;    

        default:
            return NanThrowError("Unsupported pattern type. Only 0 (CHESSBOARD), 1 (CIRCLES_GRID) or 2 (ASYMMETRIC_CIRCLES_GRID) are supported.");
    };

    if (!args[4]->IsFunction())
    {
        return NanThrowTypeError("Last argument must be a function.");
    }

    // The task holds our custom status information for this asynchronous call,
    // like the callback function we want to call when returning to the main
    // thread and the status information.

    NanCallback *callback = new NanCallback(args[4].As<Function>());
}
```

So the goal is to add more syntax sugar for argument checking. 
Basically, it should provide a convenient way to check number and type of arguments passed. 
For [CloudCV][cloudcv] project I've ended with a declarative approach because I found it fit my needs very much
and makes argument checking self-explanatory. Here is how new implementation of `calibrationPatternDetect` looks like:


```cpp
NAN_METHOD(calibrationPatternDetect)
{
    NanScope();

    Local<Object>   imageBuffer;
    Local<Function> callback;
    cv::Size        patternSize;
    PatternType     pattern;

    try
    {
        if (NanCheck(args).ArgumentsCount(5)
            .Argument(0).IsBuffer().Bind(imageBuffer)
            .Argument(1).Bind(patternSize.width)
            .Argument(2).Bind(patternSize.height)
            .Argument(3).StringEnum<PatternType>({ 
                { "CHESSBOARD",     PatternType::CHESSBOARD }, 
                { "CIRCLES_GRID",   PatternType::CIRCLES_GRID }, 
                { "ACIRCLES_GRID",  PatternType::ACIRCLES_GRID } }).Bind(pattern)
            .Argument(4).IsFunction().Bind(callback))
        {
            NanCallback *nanCallback = new NanCallback(callback);
            NanAsyncQueueWorker(new DetectPatternTask(imageBuffer, patternSize, pattern, nanCallback));
            NanReturnValue(NanTrue());
        }

        NanReturnValue(NanFalse());
    }
    catch (ArgumentMismatchException exc)
    {
        return NanThrowTypeError(exc.what());
    }
}
```

I hope you agree that second version is much more easy to read. Fluent architecture allows to write predicates in a chain, which actually is very similar to the way we thing. All predicate has self-telling names made from verb and a noun. So let me give you a brief overview what `NanCheck` is capable of.

## Fluent API

[Method chaining](http://en.wikipedia.org/wiki/Method_chaining) (aka Fluent API) makes it very easy to build final predicate for argument checking via consecutive checks. 
Each next step will be made **if and only if** all previous predicates were successful. 
In case of error, predicate will throw an `ArgumentMismatchException` exception that will terminate all further checks. `NanCheck(args)` can be evaluated to `bool` which makes it possible to use NanCheck in a condition statement:  

```cpp
if (NanCheck(args). ...) {
    // This code will be executed if argument parsing
    // will be successful        
}
```


## Type checking

To check particular argument at given index, `NanCheckArguments` provide a `Argument(index)` function. This function lets you to build a sub-predicate for given argument and bind it's value with particular local variable:

```cpp
    if (NanCheck(args).Argument(0).IsBuffer()) {
        // This code will be executed if argument parsing
        // will be successful        
    }
```

Currently, `NanCheck` support type checking of the following built-in V8 types:
1. `v8::Function`
2. `v8::Object`
3. `v8::String`

In addition, it offers `NotNull` predicate to ensure argument is not null or empty.
The list of predicates will grow for sure. New functions to check whether argument is `v8::Array`, `v8::Number`, `v8::Integer`, `v8::Boolean` will be added in a next updates.

## Binding

After type checking, it's necessary to complete sub-predicate construction by *binding* argument to a local variable. Binding is a assignment of the argument (with data marshaling, if it's necessary) to a variable that will be used later;

```cpp
    Local<Object>   imageBuffer;
    if (NanCheck(args).Argument(0).IsBuffer().Bind(imageBuffer)) {
        // This code will be executed if argument parsing
        // will be successful        
    }
```

NanCheck support transparent binding to all v8 types (Number, String, Function, Object, Array, etc.), native C++ and OpenCV types (via [marshaling system][marshal]):

```cpp
    cv::Size        patternSize;
    if (NanCheck(args).Argument(1).IsObject().Bind(patternSize)) {
        // This code will be executed if argument parsing
        // will be successful        
    }
```

There is a special case of string arguments called `StringEnum` - that is, a string argument, which can be one of a priory defined values. It introduced to support **C++ enum* types and pass them 
as string constants. `StringEnum` predicate allow to parse string value and map to C++ enum type:

```cpp
    PatternType     pattern;
    if (NanCheck(args)
        .Argument(3).StringEnum<PatternType>({ 
            { "CHESSBOARD",     PatternType::CHESSBOARD }, 
            { "CIRCLES_GRID",   PatternType::CIRCLES_GRID }, 
            { "ACIRCLES_GRID",  PatternType::ACIRCLES_GRID } }).Bind(pattern)) {
        // This code will be executed if argument parsing
        // will be successful        
    }
```

## Implementation highlights

Thanks to C++11, it's really easy to construct predicate chain using lambda functions. Basically predicate chain is nothing but a recursive anonymous function of the following form:

```cpp
    auto initFn = [innerPredicate, outerPredicate](const v8::Arguments& args) {
        return innerPredicate(args) && outerPredicate(args);
    };
```

To illustrate an idea of building predicate chain, let's take a look on ArgumentsCount implementation: 

```cpp
    NanCheckArguments& NanCheckArguments::ArgumentsCount(int count)
    {
        return AddAndClause([count](const v8::Arguments& args) 
        { 
            if (args.Length() != count)
                throw ArgumentMismatchException(args.Length(), count); 

            return true;
        });
    }
```

Here we construct outer predicate which compare number of arguments to expected value 
and throw an exception if it does not match.

With a help of `std::initializer_list` it became really simple to declare string enum with minimal syntax overhead:

```cpp
    class NanMethodArgBinding
    {
    public:
    ...
        template <typename T>
        NanArgStringEnum<T> 
        StringEnum(std::initializer_list< std::pair<const char*, T> > possibleValues);
    ...
    };
```

Now we're able to call this function with arbitrary number of elements for this enum using 
`std::initializer_list` syntax:

```cpp
    { 
        { "CHESSBOARD",     PatternType::CHESSBOARD }, 
        { "CIRCLES_GRID",   PatternType::CIRCLES_GRID }, 
        { "ACIRCLES_GRID",  PatternType::ACIRCLES_GRID }
    }
```

## Conclusion 

[**NanCheck**][nancheck] helped me to reduce amount of code required to check arguments passed to [CloudCV][cloudcv] back-end. There are many cool ideas that I'll probably add as soon as there will be necessity to have them in my library:
- Strongly typed objects (Objects with required fields)
- Optional parameters with default values
- Automatic type inference based on `Bind<T>(...)` type.
- Support of multiple types per argument (Parameter can be either of type A or B)

Please leave your comments on this post. I've spent many hours on figuring out how to implement data marshaling and type checking in V8 and Node.js, so please help information to
spread out - share and re-tweet this post. Cheers!

[cloudcv]: https://cloudcv.io
[nancheck]: https://github.com/BloodAxe/CloudCVBackend/blob/master/src/framework/NanCheck.hpp
[marshal]: https://github.com/BloodAxe/CloudCVBackend/blob/master/src/framework/marshal/opencv.cpp
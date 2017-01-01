+++
title =  "Using Travis-CI for continuous testing your projects"
date = "2014-02-23"
tags =  ["tutorials", "cloudcv", "nodejs"]
+++

img.img-thumbnail.pull-left(src="travis-logo.png")

p
    | In this post i will show you how i implemented continuous integration and testing in my  
    a(href="http://cloudcv.io") CloudCV 
    |  project. Healthy unit tests and easy and continuous integration workflow is a must in any project goes beyound "Hello, world" application. 
    | Today software is a mixture of technologies of all kind. Therefore it can break literally everywhere. Each integration point is a place of risk.
    | The CloudCV has a C++ backend that is using OpenCV library, it's front-end is written in Node.js, they both using V8 JavaScript engine and libuv
    | library to execute jobs asynchronously. Below you'll find a solution how i implemented CI and unit testing for this project.

span.more

p
    a(href="https://travis-ci.org") Travis-CI 
    |  is a continuous integration platform for almost any project. It supports C, C++, Closure, Erlang, Haskell, Scala, PHP, Ruby, Python, Java and JavaScript and many other languages.
    | You probably saw any of these images on the landing page on some GitHub repositories:
    img(src="buildpassing.png")
    img(src="buildfailing.png")
    | These are build status indicators that display the last build results of a repository. 
    | Travis-CI can run tests after each commit so you can know where exactly the problem is. 

div.alert.alert-success
    p And by the way - it's free for open-source projects! 

h2 Adding Travis-CI to your project

p
    | Technically, Travis-CI runs build job in a temporary virtual instance that is created for your project and deleted after it finish CI tasks. 
    | There is no persistentcy between consecutive builds.
    | To add Travis-CI to your project you may want to do two steps:
    ul
        li Add a ".travis.yml" manifest file to the root of your repository
        li Register your repository in Travis-CI website.

h3 Writing your first Travis-CI manifest

p
    | Travis manifest is a YAML file that define how to build the code in your repository, what is the programming language, which dependencies your project has. 
    | For example, you may want to install third-party dependencies, or create dummy database for unit tests. That all you can do with manifest file. 

p
    | Let's take a look on Travis-CI manifest for CloudCVBacked. This project is written in C++, it has two library dependencies: OpenCV and Node.js. 
    | Here i will show a full listing of my manifest and explain it line by line:  
    
pre.
    language: node_js
    before_install: 
     - sudo apt-get install build-essential
     - curl -sL https://github.com/Itseez/opencv/archive/2.4.6.1.zip > opencv.zip
     - unzip opencv.zip
     - rm opencv.zip
     - mkdir opencv-build
     - cd opencv-build/
     - cmake -DCMAKE_BUILD_TYPE=RELEASE -DBUILD_DOCS=OFF -DBUILD_EXAMPLES=OFF -DBUILD_opencv_java=OFF -DBUILD_JASPER=ON -DWITH_JASPER=ON -DBUILD_ZLIB=ON -DBUILD_SHARED_LIBS=OFF -DBUILD_TESTS=OFF -DBUILD_PERF_TESTS=OFF -DWITH_OPENEXR=OFF -DBUILD_PNG=ON -DWITH_PNG=ON -DWITH_TIFF=ON -DBUILD_TIFF=ON -DWITH_WEBP=OFF -DWITH_JPEG=ON -DBUILD_JPEG=ON ../opencv-2.4.6.1/
     - sudo make install
     - cd ..

p 
    | The first line define the programming language we will use. It is the most important instruction since it affects the build environment. 
    | For example, "node_js" option will create a virtual machine with a pre-installed
    | Node.js. In addition, it will tell Travis-CI to use package.json file for further build instruction.

    pre language: node_js

p.info 
    | You can specify a set of Node.js versions you want to test. This is called a 
    a(href="http://docs.travis-ci.com/user/build-configuration/#The-Build-Matrix") Build Matrix
    |. For example, in the example below we ask build server to build project on four versions of node:
    pre language: node_js
        | node_js:
        |   - "0.11"
        |   - "0.10"
        |   - "0.8"
        |   - "0.6" 

h4 Installing additional dependencies

p
    | By default, build server uses "npm install" command to install node.js-based projects. 
    | The "npm install" itself read the package.json file and install every dependency from it using Node Pacakage Manager (npm) tool.
    | To install non-node.js dependencies we can use "before_install" instruction to install additional software into build environment. 
    | Here we download and instal OpenCV 2.4.6.1:
    ul
        li First, we install build essential software (gcc, cmake, etc..)
        ll Download stable opencv package from Github.
        li Unzip it and go into build directory
        li Configure OpenCV using cmake
        li Build and install OpenCV
pre.
    before_install: 
     - sudo apt-get install build-essential
     - curl -sL https://github.com/Itseez/opencv/archive/2.4.6.1.zip > opencv.zip
     - unzip opencv.zip
     - rm opencv.zip
     - mkdir opencv-build
     - cd opencv-build/
     - cmake -DCMAKE_BUILD_TYPE=RELEASE -DBUILD_DOCS=OFF -DBUILD_EXAMPLES=OFF -DBUILD_opencv_java=OFF -DBUILD_JASPER=ON -DWITH_JASPER=ON -DBUILD_ZLIB=ON -DBUILD_SHARED_LIBS=OFF -DBUILD_TESTS=OFF -DBUILD_PERF_TESTS=OFF -DWITH_OPENEXR=OFF -DBUILD_PNG=ON -DWITH_PNG=ON -DWITH_TIFF=ON -DBUILD_TIFF=ON -DWITH_WEBP=OFF -DWITH_JPEG=ON -DBUILD_JPEG=ON ../opencv-2.4.6.1/
     - sudo make install
     - cd ..

h3 Building Node.js C++ module with Travis-CI
p
    | When the build environment is ready (If before_install target succeeded), Travis-CI performs install task (by default it's "npm install"). Now, it's time for NPM to act. 
    | Each module in Node.js usually has "package.json" file. 
    | Usually it contains name, version of repository url of the module, it's description, list of dependencies, pre/post install and test scripts.
    | The pre-install target in CloudCVBacked builds the dynamically-linked C++ library that exports Node.js functions:

    pre 
        | {
        | "name": "cloudcv",        
        | ...
        | "scripts": {
        |     "preinstall": "node-gyp clean rebuild",
        |     "test": "expresso test/*"
        | },
        | 
        | "devDependencies": {
        |     "expresso": "*"
        | },
        | 
        | ...
        | }

div.alert.alert-info
    p To read more about using C++ and Node.js together please follow related posts:
        ul
            li  
                a(href="http://computer-vision-talks.com/articles/2013-08-27-connecting-opencv-and-node-js-inside-cloud9-ide/"): strong Connecting OpenCV and Node.js inside Cloud9 IDE
            li
                a(href="http://computer-vision-talks.com/articles/2013-08-19-cloud-image-processing-using-opencv-and-node-js/"): strong Cloud image processing using OpenCV and Node.js

h4 Node.js modules testing with expresso

p
    | For testing of CloudCV I'm using 
    a(href="http://visionmedia.github.io/expresso/") Expresso
    |  test framework. It fits my needs and doesn't bother me with complex configuration. So i'm using it, but you're free to use any other test framework.
    | The Travis-CI runs "npm test" command, which in fact execute. 
    pre "expresso test/*"

p
    | Expresso runs all JavaScript tests in the specified folder and returns the status code to Travis-CI. 
    | If any test fails the Travis-CI job will be marked as failed and you will get a "Build Failing" notification.

h2 Enabling Travis-CI for your repository
    
p 
    | When you've added manifest file to your repository it's time to register on travis-ci.org website and add your repository there. 
    | I'm not going to describe the registration process since it's trivial.
    | But i want to point you to manifest validatio tool that can save a lof of your time:  
    a(href="http://lint.travis-ci.org/") Travis manifest validation tool
    |. This page helps you to check whether your manifest is correct.

p
    | After you've added your project to Travis-CI, it will do the rest. 
    | Each time you push changes to repository, Travis-CI will build your project and send you notification if the error occurs.

h2 Adding Build Status to the README

p
    | One of the cool features of travis is that you can view the build status from your github repository. 
    | Open the README.md file and add the following lines. 
    | Replace GITHUB_USERNAME with your github username and PROJECT_NAME with the name of your github project.
pre
   | [![Build Status](https://travis-ci.org/[GITHUB_USERNAME]/[PROJECT_NAME].png)](https://travis-ci.org/[GITHUB_USERNAME]/[PROJECT_NAME])

p
    | This will add the build status icon image to your readme page, so you will always know how healthy your project is.

img.image-thumbnail(src="build-status.png")

h2 Conclusion

p
    | As a developer and computer vision consultant, I work with many heterogenous software/hardware platforms. I've been using TeamCity, Jenkins and Team Foundation as a CI and build servers. 
    | Although they provide much more features than Travis-CI does, no one can compete with Travis-CI if you want to have free, easy-to-use, no-headache continuous-integration service for your projects.
    | It will fits well for both web-based (php, ruby, python) application, tools and libraries in C++, Python, Scala and Java. 

h2 References
p
    ul
        li: a(href="https://travis-ci.org") Travis-CI 
        li: a(href="http://lint.travis-ci.org/") Travis manifest validation tool
        li: a(href="http://visionmedia.github.io/expresso/") Expresso TDD for JavaScript
        li: a(href="http://computer-vision-talks.com/articles/2013-08-27-connecting-opencv-and-node-js-inside-cloud9-ide/") Connecting OpenCV and Node.js inside Cloud9 IDE
        li: a(href="http://computer-vision-talks.com/articles/2013-08-19-cloud-image-processing-using-opencv-and-node-js/") Cloud image processing using OpenCV and Node.js
br
p.lead 
    | I hope you enjoyed reading this post. Please, describe your experience using Travis-CI in the comments. Thank you!. 

+++
title =  "Image processing in your browser - Unit Test automation"
date = "2014-10-31"
tags =  ["tutorials", "nodejs"]
+++


JavaScript. Do you like debug JavaScript code? I hate it. Literally.
What what if you have to? In this post I'm going to show you how to
simplify your life by automating unit testing of the JavaScript code 
for the browser.

![](monkeys.jpg)

To get things more interesting - let's automate unit-testing of the 
image processing library called [JSFeat][jsfeat]. JSFeat provides a
JavaScript implementation of the basic image processing operations 
that let you to process images in your browser and build sophisticated
algorithms. **It's like OpenCV for web-browser**.

The [source code][example] for this tutorial is available on my Github page: https://github.com/BloodAxe/jsfeat.

<!--more-->

Typically, when we test C++ software, we end up with test framework of your
choise and test runner. It can be continuous integration server like Jenkins 
in your company or public Travic-CI service for open-source project. 

With browser JavaScript things gets more complicated. I'm not talking about 
JS unit-test frameworks - [mocha][mocha] is more than enough. I'm talking about 
browser testing itself. Basically you have to open a webpage in your browser to
invoke a test cases. Manually. Ew!

Moreover, due to browser sandbox, you are not allowed to access canvas data for
local files. In practice it means that code like showed below **won't work if you
open a HTML page as a local file**:

```js
var img = new Image();
img.src = 'dummy.jpg';
img.onload = function() {
    // This will never happen
};
```

You will have to setup a local HTTP server to serve these tests pages to get this 
works. One simple way to do it by using python: ``python -m SimpleHTTPServer 8000`` will 
do the job. However it's only a partial solution.

Now let's imagine that you have a lot of these tests. Would you open each page 
individually, watch how they run and wait to analyze their results? No doubts, 
you can do it, but this is not automated testing in any way.

To recap, here are a list of problems that exists in browser JS testing:
 - A test framework
 - Have to open test pages in a browser
 - Hard to automate and collect results
 - Need local webserver

Since I'm used to Mocha, i will use it. However it's not obligatory and you can chose any other test framework you like. But as you will see later, with Mocha it's really simple. ith mocha you can write your scripts like showed below. This is a real test I wrote as an example for JSFeat:

```js
// Source test/test_grayscale.js
'use strict';

describe('jsfeat', function(){

  describe('imgproc', function(){

    it('grayscale', function(done) {

      var img = new Image();
      img.src = 'lena.png';
      img.onload = function() {

        var width = img.width;
        var height = img.height;

        var canvas = document.createElement('canvas');
        var context2d = canvas.getContext('2d');

        context2d.drawImage(img, 0, 0, width, height);
        var image_data = context2d.getImageData(0, 0, width, height);
 
        var gray_img = new jsfeat.matrix_t(width, height, jsfeat.U8_t | jsfeat.C1_t);
        var code = jsfeat.COLOR_RGBA2GRAY;

        jsfeat.imgproc.grayscale(image_data.data, width, height, gray_img, code);
        done();
      };

    });

  });
   
});
```

There is a really powerful tool in Nodejs world called [Grunt][grunt] that we will use
to automate tasks like JavaScript static code checking, minification and testing. 

```bash
npm install -g grunt-cli
npm install grunt --save-dev
```

A headless browser is a full-featured browser engine without graphical interface. It was designed to simulate a real browser including DOM and JavaScript. The most important headless browser is [PhantomJS][phantomjs]. I found that it works like a charm for this task. With phantomjs we can run arbitrary HTML page inside and execute JavaScript code. This tool let's us to get rid of the manual tabs openning.
    
```js
// script.js:
// Simple Javascript example

console.log('Loading a web page');
var page = require('webpage').create();
var url = 'http://www.phantomjs.org/';
page.open(url, function (status) {
  //Page is loaded!
  phantom.exit();
});

phantomjs script.js
```

PhantomJS and Mocha are already connected together in a single grunt task called [grunt-mocha-phantomjs][grunt-mocha-phantomjs]. And we use [grunt-contrib-connect][grunt-contrib-connect] to host a local webserver.
    
```bash
npm install grunt-mocha-phantomjs --save-dev
```

This extension does exactly what we need: it starts a local webserver, open page in phantomjs and run JS test cases.
With help of it, we are able to run all our tests using simple command:

```bash
grunt test
```

Here's an example output:

```bash
Running "concat:jsfeat" (concat) task
File build/jsfeat.js created.

Running "uglify:build" (uglify) task
>> 1 file created.

Running "connect:server" (connect) task
Started connect web server on http://0.0.0.0:8000

Running "mocha_phantomjs:all" (mocha_phantomjs) task


  jsfeat
    imgproc
      ✓ grayscale 


  1 passing (41ms)


Done, without errors.
```


Within this setup you are now able to automate testing of JavaScript code that require interaction with HTML5 Canvas features. This way I test the code that I write for browser image processing. I hope you enjoyed this post and I'm looking forward to see your questions and mentions in a comments!  

The [source code][example] for this tutorial is available on my Github page: [https://github.com/BloodAxe/jsfeat][example].

[example]: https://github.com/BloodAxe/jsfeat
[jsfeat]: http://inspirit.github.io/jsfeat/
[mocha]: https://github.com/mochajs/mocha
[phantomjs]: http://phantomjs.org/
[grunt-mocha-phantomjs]: https://github.com/jdcataldo/grunt-mocha-phantomjs
[grunt-contrib-connect]: https://github.com/gruntjs/grunt-contrib-connect

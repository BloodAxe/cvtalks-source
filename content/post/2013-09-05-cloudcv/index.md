+++
title =  "CloudCV - Cloud image processing platform"
date = "2013-09-05"
tags =  ["opencv", "nodejs", "cloudcv", "cloud-computing"]
+++

    | Hey everyone! I continue to play with clouds and today it's time to reveal the CloudCV \- a cloud-based image processing project. 
    | Based on my previous posts i host a server in the Digital Ocean's cloud. 
    | I have to say, everything is working like a charm. 
    | The cheapest 5$/month plan gives me whatever i may need for this project. 
    | All the source-code is already sits on Github and you are more than welcome to study it. 
    | This is my goal - to share the experience. So i'm looking forward to see you in the comments! 

img.full-width(src="cloudcv-image-processing-platform.png", alt="CloudCV - Cloud image processing platform")

p Inside of this post you'll find the detailed instructions how to deploy the Node.js and OpenCV in your personal cloud. 

h2 Configuring your instance

p First, you need to create a Droplet - instance of your virtual machine running in Digital Ocean's cloud. You can find the big "Create Droplet" button in top right corner of your control panel. For our needs a smallest 1 core instance will be enough: 

img.full-width(src="Screen-Shot-2013-09-13-at-5.42.20-PM-1024x805.png", alt="Create droplet")
    
p Next we choose the image for the new Droplet, region and SSH keys. You can pick the closest region to your location to have lower latency: 

img.full-width(src="Screen-Shot-2013-09-13-at-5.44.27-PM-1024x805.png", alt="")

p One word about your choice for the OS for Droplets. Actually you can pick any of them. Since i more familiar with Ubuntu i'll use it, but the following steps should be very similar to other Linux distributives (Of course, you'll have to change _apt-get_ on Debian). And let's pick Ubuntu 64-bit, since we're tough guys, aren't we? 

img.full-width(src="Screen-Shot-2013-09-13-at-5.44.33-PM-1024x805.png", alt="")

p Don't forget to add SSH keys to this Droplet, otherwise you'll have to add them manually after your droplet is ready. When you're done hit "Create Droplet" and it will be ready in a minute. It's the fastest deployment i've ever seen! When it's done you should be able to login using _ssh_ client to your virtual machine. From this moment, we will use only a console to configure our server. 

h2 Node.js and OpenCV installation

p Let's recap the plan. Our goal is to make image processing using OpenCV library possible in the cloud environment. This means your Droplet will be executing a native code, but it needs a public API to communicate with the others. Such things as REST API and JSON may come in handy here. We will be serving user requests that comes as HTTP requests. For this we need a server software to route requests to native code. Actually, i think, Apache, ngnix or any other web server can do it for us. But i've chosen Node.js for two reasons - first of all, i want to study JavaScript and the second one - it's kind of a trend now :) I wrote a helper snippet that install all dependencies for Node.js and OpenCV. First of all we update the package list and install git, cmake and build essentials. As a next step this script configure an OpenCV in a proper way (disable unsued image codecs, disable tests and apps and turns on static builds). And finally this script install latest Node.js (this recipe was found somewhere in the Google). 

h2 Source Code

p 
    | There are two source code repositories for this project: 
    ul
        li
            a(href='https://github.com/BloodAxe/CloudCV') CloudCV
            span  - is web-application written in Node.js.
        li
            a(href='https://github.com/BloodAxe/CloudCVBackend') CloudCVBackend
            span  - is a C++ Module for Node.js. This module contains all image processing stuff inside of it.

h2 Architecture overview

img(src='cloudcv_architecture-1.png')

h2 Disclaimer

p 
    | You can support this project by following this referral link to purchase a cloud hosting. 
    | A really recommend Digital Ocean for both production and testing purposes due to their low prices, 
    | SSD disks, good support and intuitive administration console. 
    a.btn.btn-primary(href='https://www.digitalocean.com/?refcode=b93faa829f80') Buy Digital Ocean cloud hosting
    

+++
title = "Publications"
date = "2014-04-09"
+++

My slowly growing list of papers I had pleasure to work on

## 2021

### One-shot Facial Expression Reenactment using 3D Morphable Models

Roman Vey, Eugene Khvedchenya

> The recent advance in generative adversarial networks has shown promising results in solving the problem of head reenactment. It aims to generate novel images with altered poses and emotions while preserving the identity of a human head from a single photo. Current approaches have limitations, making them inapplicable for real-world applications. Specifically, most algorithms are computationally expen- sive, have no apparent tools for manual image manipulation, require audio or take multiple input images to generate novel images.
Our method addresses the single-shot face reenactment problem with an end-to- end algorithm. 
> The proposed method utilizes head 3D morphable model (3DMM) parameters to encode identity, pose, and expression. 
> With the proposed approach, the pose and emotion of a person on an image is changed by manipulating its 3DMM parameters. 
> Our work consists of a face mesh prediction network and a GAN-based renderer. 
> A predictor is a neural network with simple encoder architecture that regresses 3D mesh parameters. 
> A renderer is a GAN network with warping and rendering submodules that renders images from a single source image and target image 3DMM parameters.
This work proposes a novel head reenactment framework that is computationally efficient and uses 3DMM parameters that are easy to alter, making the proposed method applicable in real-life applications. It is first to our knowledge approach that simultaneously solves two of these problems: 3DMM parameters prediction and face reenactment, and benefits from both.

### Fully convolutional Siamese neural networks for buildings damage assessment from satellite images

https://arxiv.org/abs/2111.00508

Eugene Khvedchenya, Tatiana Gabruseva

> Damage assessment after natural disasters is needed to distribute aid and forces to recovery from damage dealt optimally. This process involves acquiring satellite imagery for the region of interest, localization of buildings, and classification of the amount of damage caused by nature or urban factors to buildings. In case of natural disasters, this means processing many square kilometers of the area to judge whether a particular building had suffered from the damaging factors.
> In this work, we develop a computational approach for an automated comparison of the same region's satellite images before and after the disaster, and classify different levels of damage in buildings. Our solution is based on Siamese neural networks with encoder-decoder architecture. We include an extensive ablation study and compare different encoders, decoders, loss functions, augmentations, and several methods to combine two images. The solution achieved one of the best results in the Computer Vision for Building Damage Assessment competition.

<br>
<hr>
<br>

## 2020

### ImageNet Pre-trained CNNs for JPEG Steganalysis

http://ws2.binghamton.edu/fridrich/Research/Alaska-2-Revised.pdf

Yassine Yousfi , Jan Butora , Eugene Khvedchenya, Jessica Fridrich

> In this paper, we investigate pre-trained computer-vision deep architectures, such as the EfficientNet, MixNet, and ResNet for steganalysis. These models pre-trained on ImageNet can be rather quickly refined for JPEG steganalysis while offering significantly better performance than CNNs designed purposely for steganalysis, such as the SRNet, trained from scratch. We show how different architectures compare on the ALASKA II dataset. We demonstrate that avoiding pooling/stride in the first layers enables better performance, as noticed by other top competitors, which aligns with the design choices of many CNNs designed for steganalysis. We also show how pre-trained computer-vision deep architectures perform on the ALASKA I dataset. 

<br>
<hr>
<br>

## 2018

### Albumentations: fast and flexible image augmentations

https://arxiv.org/abs/1809.06839

Alexander Buslaev, Alex Parinov, Eugene Khvedchenya, Vladimir I. Iglovikov, Alexandr A. Kalinin

> Data augmentation is a commonly used technique for increasing both the size and the diversity of labeled training sets by leveraging input transformations that preserve output labels. In computer vision domain, image augmentations have become a common implicit regularization technique to combat overfitting in deep convolutional neural networks and are ubiquitously used to improve performance. While most deep learning frameworks implement basic image transformations, the list is typically limited to some variations and combinations of flipping, rotating, scaling, and cropping. Moreover, the image processing speed varies in existing tools for image augmentation. We present Albumentations, a fast and flexible library for image augmentations with many various image transform operations available, that is also an easy-to-use wrapper around other augmentation libraries. We provide examples of image augmentations for different computer vision tasks and show that Albumentations is faster than other commonly used image augmentation tools on the most of commonly used image transformations. 

<br>
<hr>
<br>

## 2010 

### Mastering OpenCV with Practical Computer Vision Projects

Daniel Lélis Baggio, Shervin Emami, David Millán Escrivá, Khvedchenia Ievgen , Naureen Mahmood , Jasonl Saragih , Roy Shilkrot 

> This is the definitive advanced tutorial for OpenCV, designed for those with basic C++ skills. The computer vision projects are divided into easily assimilated chapters with an emphasis on practical involvement for an easier learning curve.
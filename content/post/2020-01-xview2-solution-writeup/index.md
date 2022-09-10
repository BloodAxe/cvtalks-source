+++
title = "Getting a medal and publishing follow-up paper on NeurIPS"
date = "2022-09-09"
tags =  ["challenge", "diu", "rgb", "pytorch", "deep-learning"]
thumbnail = "post/2020-01-xview2-solution-writeup/final_leaderboard.png"
+++

Automatic damage assessment after natural disasters allows to optimally distribute
aid and forces to recovery from damage dealt. This process involves acquisition of satellite
imagery for the region of interest, localization of buildings and classification of amount of
damage caused by nature or urban factors to buildings. In case of natural disasters, this
often means processing of many square kilometers of area before and after the event in
order to judge whether particular building had suffered from the damaging factors or not.
The main objective of xView 2 challenge was to build a solution, capable of
processing pair of satellite images of same region (called hereinafter “pre” and “post”
images) to detect buildings and classify level of damage dealt to them.
In this white paper I describe my solution to this problem and explain few know-how
ideas, that resulted in a top-performing solution. The source code for my solution is available
at https://github.com/BloodAxe/xView2.

# Introduction to Solution

The formal problem statement can be formulated as follows: given a pair of images
(“pre” and “post”), output a semantic mask of same spatial size, that corresponds to “pre”
image and contains values {0,1,2,3,4} representing a “no building” class for 0, 1 for “building,
no damage”, 2 for “building, minor damage”, 3 for “building, major damage” and 4 for
“building, destroyed”. A target objective in the xView 2 challenge was a weighted sum of F1
score for building localization and F1 score for damage classification. Final score was
computed as 0.3 * localization F1 + 0.7 * damage classification F1 score.
Since the evaluation metric is pixel-based and problem itself can be treated as
semantic segmentation, I used deep learning approach and fully-convolutional neural
network (CNN) models to train an ensemble of semantic segmentation models. All models
were solving multi-class segmentation problem with five classes (One class for “Non a
building” and 4 classes for classifying damage level). Although single-model performance
was enough to stay in Top-50 on the leaderboard, ensembling multiple models was a key to
increase both localization and damage classification performance by N% on the
leaderboard. A detailed overview of architecture of segmentation models I used will be given
in “Details on Modeling Tools and Techniques” section.
It worth to note that although ground-truth buildings were given in JSON format as
separate polygons. I converted them to raster masks. Another approach to this problem
could be instance segmentation with Mask-RCNN. However, I believe this approach is an
overkill as it makes model and training routine more complex, and more importantly -
competition metric does not require to separate buildings to instances. Should this be the
case, my approach probably would be different.
In a nutshell, my solution is a pretty big ensemble of segmentation models. As
ensembling technique I used weighted averaging after softmax activations. I used 5 folds
and 13 models in total, so for each fold there were at least 2 models and I trained different
model architectures (different encoder backbones and UNet or FPN decoder). As an
additional trick to increase models accuracy, I optimized weights for each class for each
model. Second trick that I used was pseudo-labeling. After obtaining a good-performing
ensemble (LB 0.801) I made predictions for test set and used these predictions to fine-tune
some of my models on both train and test. Pseudo-labeling increased model accuracy by 2
points to (LB 0.803).
Data Cleaning Techniques
Training data contained 18337 images made at different time, locations and disaster
types. All images has size of 1024x1024 pixels. Corresponding “pre-” and “post-” images
were well-aligned, and quality of ground-truth annotation was found to be high. During the
competition I found no obvious labeling errors in the training set. Therefore no specific data
cleaning was performed.
I also tried to align “pre” and “post” images even more, using image registration
techniques, like coarse-to-fine template matching and ECC [2]. My line of thinking was it may
give a small boost to damage classification. In practice, additional alignment did no help and
in fact, made score even worse. So I abandoned this idea and used images “as is” for the
training models. My understanding why it did not help is the following: Input pair of images is
already aligned decently. Therefore, when image goes through encoder part of the
segmentation CNN, this slight misalignment gets decimated with every new layer. E.g for
output feature map of stride 4, spatial information is compressed 4 times, for stride 8 - 8
times and so on. At the same time, as we decrease spatial size of feature maps, they
become more and more semantically rich and receptive field size is increasing. Therefore
even with some misalignment, feature maps computed for pre- and post- images will be
well-aligned.
Data Processing Techniques
To be able to validate my models locally, I splitted training dataset into 5 folds. As a
splitting algorithm I used multilabel stratification using presence flags of non-damaged,
minor-damaged, major-damaged and destroyed buildings in each image and event name as
labels. This split guaranteed that each fold will contain same distribution of damage classes
with respect to even types. Alternative approach is to use group K-fold to ensure images
from one event belongs only to one fold.
Having a 5-fold cross-validation allowed me to find optimal model architectures using
cross-validation and fine-tune best-performing models afterwards. In addition, it allowed to
use simple yet effective ensembling via arithmetic averaging of individual models predictions
from different folds.
For the training, I converted ground-truth from JSON data to raster masks of
1024x1024 size using code from official baseline repository
https://github.com/DIUx-xView/xview2-baseline. Generated masks contained 5 classes (
Details on Modeling Tools and Techniques
For training deep learning models I used PyTorch framework and Catalyst[3] library
for running training experiments. PyTorch is known as a very efficient and flexible framework
to construct various model architectures which worked very well in this challenge due to
non-standard input format to the model.
Usually, segmentation models performs image-to-image translation, and their input is
a tensor of BCiHW shape and output is BCoHW tensor, but in this competition the input were
two images (Two tensors of BCiHW shape, with Ci = 3) and the output is tensor of
BCoHW (With Co equals 5).
There are a couple of ways how one can feed two images to the model. An easiest
and the most obvious way is to concatenate input “pre” and “post” images at the very
beginning to create a 4D tensor of six channels and feed it through model. In practice, this
approach did not work quite well, since misalignment in image pair is propagated to all
feature maps and amplified at the encoder stage. Models, trained using this image
composition strategy could barely reach 0.7 score on the leaderboard and therefore this
approach was abandoned.
A more advanced approach is to extract feature maps from each image
independently using a shared encoder. This would lower the influence of misalignment in the
image pairs and would enforce encoder to learn building features for both “pre” and “post”
disaster images. After computing feature maps for both images, I concatenated feature
maps from corresponding layers and used new feature maps as input to segmentation
decoder head. In practice, this approach worked really well - even a light model built on top
of ResNet-34 encoder and UNet decoder scored 0.73 on the leaderboard without any
sophisticated augmentations or training tricks.
Training Pipeline
Training was performed using Catalyst[3] framework on top of PyTorch library. Use of
Catalyst greatly reduced amount of boilerplate code needed to create a training pipeline,
manage experiments, monitor predictions. In addition it supports Apex mixed-precision
training and multi-GPU support our of the box.
All models were trained on random-sized crops of 512x512 size and validated on full
1024x1024 images. For saving a best model checkpoint during the training I used the official
competition metric score. Training was done on 4 NVidia GeForce 1080Ti GPUs and a
p3.8xlarge AWS Instance equipped with 4 NVidia Tesla V100 GPUs.
Depending on model architecture, I was able to fit from 32 to 64 images in a batch. I used
RAdam[5] as optimizer, with weight decay 1e-5 and no dropout. Depending on a model, I used
different set of image augmentations (For lighter models I used less strong augmentations, and
for heavier models I had more aggressive set of augmentation parameters). Full list of
hyper-parameters for training will be included in the source-code repository.
Loss
Image segmentation can be considered as a problem of pixel classification, which
means we can use well-studied cross-entropy loss to train a model to predict a target mask.
In practice this approach works very well, although many new auxiliary losses has been
developed in recent years: soft dice/jaccard loss, focal loss, lovasz loss to name a few.
However in this competition, what I found, is cross-entropy loss with experimentally tuned
class weights worked the best. Specifically, I assigned weight of 1.0 to “not building” and “no
damage” classes and 3.0 to “minor”, “major” and “destroyed” classes. This weighting of
underrepresented classes seemed to work quite well in practice and I did not make any
under-/oversampling during the training:
loss = nn.CrossEntropyLoss(weight=torch.tensor([1.0, 1.0, 3.0, 3.0, 3.0]))
Augmentations
Image augmentation pipeline played crucial part of my training pipeline.
Augmentation technique not only helped to prevent model overfitting, but in fact was a key
component that increased model performance, especially on large displacement of “pre” and
“post” image. For augmenting images I used open-source library Albumentations [4]. This
library allows to create augmented images on-the-fly during training, and thankfully to large
number of supported augmentations, I was able to find right balance between making
augmented images hard-enough to prevent overfit while keeping the data distribution close
to original data. Albumentations allow to augment image and mask simultaneously, ensuring
that augmented image and mask are consistent, which made this library a perfect fit for this
challenge.
Image augmentation pipeline (Applied only during training phase)
Input: “pre” and “post” image pair
1) Apply spatial transformation only to “post” image:
a) Random rotation by small degree (-3,+3)
b) Random shift of image by (-10,+10) pixels
c) Random zoom (-2%,+2%)
2) Crop 512x512 patch for same region from “pre” and “post” with a
random scale varying from 80% to 120%.
3) Apply color augmentation independently to “pre” and “post”:
a) Random brightness, contrast and gamma change
b) Changes to image either in HSV or RGB colorspace
4) Apply spatial augmentations to “pre” and “post” images with same
transformation parameters (to ensure “pre” and “post” crops are
consistent)
a) Random grid shuffle
b) Random scale (+- 10%) and rotation (+-10 degrees)
c) Random rotation by 90 degrees and image transposition
d) Mask dropout
Ensembling technique
My best-performing solution was an ensemble of 13 segmentation models trained on
different folds and varying with encoder backbone (ResNet, DenseNet, EfficientNet) and
decoder architecture (UNet, FPN).
No Model Architecture Fold Localization Damage
0 resnet34_unet_v2 0 0.865514 0.707850
1 resnet101_fpncatv2_256 0 0.871020 0.714319
2 seresnext50_unet_v2 1 0.858342 0.663708
3 resnet34_unet_v2 1 0.864830 0.683157
4 densenet201_fpncatv2_256 1 0.869787 0.683657
5 inceptionv4_fpncatv2_256 2 0.833248 0.719878
6 densenet169_unet_v2 2 0.871033 0.721195
7 resnet34_unet_v2 2 0.862285 0.722121
8 resnet34_unet_v2 3 0.865738 0.707328
9 seresnext50_unet_v2 3 0.879670 0.733817
10 efficientb4_fpncatv2_256 3 0.860539 0.721291
11 resnet34_unet_v2 4 0.866404 0.700976
12 resnet101_unet_v2 4 0.874912 0.707240
Table 4.1 - Validation metrics for individual models in the best ensemble (on public LB)
Conclusion and Acknowledgments
During the competition I realized the main limiting factor for my solution (I’d speculate
this is likely to be true for other participants) is classification accuracy of minor and major
damage classes. Even on flooding type of disaster events, it was quite hard to discriminate
between these two classes with a naked eye. So it’s not a surprise that models had some
hard time on assigning a correct class. My solution did not leverage sophisticated
post-processing of predicted masks. I believe, post-processing with watershed and instance
segmentation may increase the classification accuracy, especially in hard examples.
According to my experiments, any post-processing I tried lead to decrease in classification
accuracy. Therefore I used none in my final solution.
It worth mentioning that according to public leaderboard, localization accuracy was
quite even for all Top-50 submissions. It may indicate there is an upper limit of localization
accuracy caused by off-nadir imagery, which cause significant offsets between ground-level
building contour and it’s roof. This score is quite close to SOTA score on Inria Satellite
Labeling Dataset[6], which is 80.3 IoU. The main boost of Top-5 submissions is gained from
classification accuracy which is due to use of ensembling of many models.
Figure 5.1 - Scores distribution for Top-50 submissions on public LB

# References
1. Olaf Ronneberger, Philipp Fischer, Thomas Brox, U-Net: Convolutional Networks for Biomedical Image Segmentation. https://arxiv.org/abs/1505.04597
2. Evangelidis, G.D. and Psarakis E.Z. “Parametric Image Alignment using Enhanced Correlation Coefficient Maximization”, IEEE Transactions on PAMI, vol. 32, no. 10, 2008
3. https://github.com/catalyst-team/catalyst
4. https://github.com/albumentations-team/albumentations
5. https://arxiv.org/abs/1908.03265
6. https://project.inria.fr/aerialimagelabeling/
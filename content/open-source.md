+++
title = "Open-Source"
date = "2014-04-09"
+++

You can find full list of my open-source projects under my GitHub account: https://github.com/BloodAxe

But there are some highlights that are worth to mention here:

## [PyTorch-Toolbelt](https://github.com/BloodAxe/pytorch-toolbelt)

https://github.com/BloodAxe/pytorch-toolbelt

Pytorch-toolbelt is a Python library with a set of bells and whistles for PyTorch for fast R&D prototyping and Kaggle farming.

What's inside:
* Easy model building using flexible encoder-decoder architecture.
* Modules: CoordConv, SCSE, Hypercolumn, Depthwise separable convolution and more.
* GPU-friendly test-time augmentation TTA for segmentation and classification
* GPU-friendly inference on huge (5000x5000) images
* Every-day common routines (fix/restore random seed, filesystem utils, metrics)
* Losses: BinaryFocalLoss, Focal, ReducedFocal, Lovasz, Jaccard and Dice losses, Wing Loss and more.
* Extras for Catalyst library (Visualization of batch predictions, additional metrics)
* Showcase: Catalyst, Albumentations, Pytorch Toolbelt example: Semantic Segmentation @ CamVid

Why
Honest answer is "I needed a convenient way to re-use code for my Kaggle career". During 2018 I achieved a Kaggle Master badge and this been a long path. Very often I found myself re-using most of the old pipelines over and over again. At some point it crystallized into this repository.

This lib is not meant to replace catalyst / ignite / fast.ai high-level frameworks. Instead it's designed to complement them.

## [Albumentations](https://github.com/BloodAxe/albumentations)

https://github.com/BloodAxe/albumentations

This project is a fork of [Albumentations](https://github.com/albumentations-team/albumentations).
I was a member of Albumentations core team back in a day. Since the beginning of russian invasion into Ukraine I've left
the team. You can find answers why [here](https://www.linkedin.com/posts/cvtalks_github-albumentations-teamalbumentations-activity-6904157323959627776-POEi?utm_source=share&utm_medium=member_desktop).

This fork contains few improvements/additions that I feel are necessary to have. 
Since this private fork is mainly meant for personal use I feel less obliged to keep backward compatibility with the original project.

At some point it will become deprecated and transform into something new & shiny.

## [Catalyst](https://github.com/bloodAxe/catalyst)

https://github.com/bloodAxe/catalyst

Yet another fork of high-level training [library](https://github.com/catalyst-team/catalyst) around PyTorch. 
Forked quite a while ago to ensure API will remain stable and also added missing features to my taste.
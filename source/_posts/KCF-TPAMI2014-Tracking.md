---
title: KCF-TPAMI2014-Tracking
date: 2019-05-18 14:53:32
categories: paper
tags: Tracking
---

##  KCF-2014
* High-Speed Tracking with Kernelized Correlation Filters
* Paper:https://arxiv.org/abs/1404.7584
* Code:https://github.com/foolwood/KCF

## Overview
1. 相关滤波方向很经典的文章，果然还是数学思维是第一生产力中的核心。看了知乎上关于傅里叶变换的博文，很通透。
2. 文章思想主要是，使用循环矩阵生成样本，使用相关滤波训练一个判别式分类器。

<!--more-->
## what
1. 主要说负样本对模型训练十分重要，但是训练过程中负样本数量少，文中通过循环矩阵来生成样本。
2. 通过带核函数的相关滤波来训练，使用快速傅里叶变换来加速计算。
3. KCF是在CSK（使用灰度图）上做的改进，引入了多通道特征（使用HOG替代了灰度图），核函数。

## how
1. 使用HOG替代了灰度特征
2. 使用循环矩阵生成样本
3. 使用核函数，快速傅里叶变换做加速






























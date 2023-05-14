---
title: SiamDW-CVPR2019-Tracking
date: 2019-05-23 10:20:46
categories: paper
tags: Tracking
---
## SiamDW
* SiamDW：Deeper and Wider Siamese Networks for Real-Time Visual Tracking
* Paper：http://openaccess.thecvf.com/content_CVPR_2019/papers/Zhang_Deeper_and_Wider_Siamese_Networks_for_Real-Time_Visual_Tracking_CVPR_2019_paper.pdf
* Code：https://github.com/researchmm/SiamDW

## Overview
1. 针对深度网络(如Resnet)无法在siamese网络取得很好的效果。SiamRPN++也是从这个点出发去做，两者的都找出了其影响最大的原因--padding，不过解决方式不同。
2. siamFC/siamRPN在VOT16上的EAO提升了23.3%和8.8%

<!--more-->

## why
1. 深度网络无法在siamese网络中取得很好的效果，对网络的各部分进行了分析，大体原因如下几点：   
   * 神经元感受野的增大会减小特征分辨和定位的精度
   * 网络的padding会导致网络学习的位置偏见(**主要因素**)
   * 网络stride


## what 
1. 基于残差bottleneck block进行了改进--cropping-inside residual ，用来crop(消除)padding的影响，同时可以控制感受野大小，网络的stride
2. 几点说法
  * 感受野：大的感受野能够把握target的结构信息，感受野太小不利于特征提取
  * Stride：影响定位精度，控制着输出feature的大小；输出feature的大小影响分辨和检测精度。
  * Padding：当物体移动到padding边缘，会产生一定的位置偏见
3. 几点分析
   * 网络越深，感受野就越大，理论上最后一层得感受野最大
   * Stride为4-8-16对应得变化为0.59-0.60-0.55(Alexnet),说明siamese更适合中等strdie**(4,8)**，随着网络深度的增加，stride不应该增加。
   * 最优的感受野大概为image z的**60%-80%**
   * 输出的feature大小较小的话，会降低track精度，因为小的feature没有组后的空间结构信息。
   * siamese训练的数据中object都在image的中心，所以当test数据中的object出现在image边缘时效果不好，且会受到边缘padding的影响。

## how
1. 基于resnet bottleneck改进--CIR Unit
   * 在resnet block之后添加了crop operation，将block之后padding的**最外层元素**给crop掉。这样feature map岂不是会越来越小？
2. 下采样CIR-D
   * 将stride由2设为1，并在block之后也加入了crop operation操作，之后又加了max pooling用于降采样
3. CIR多分枝结构 - CIR-Inception/CIR-NeXt
   * CIR-Inception : 在short-connection部分加入了1x1卷积，并且通过concatenation来将两个分支的feature融合
   * CIR-ResNeXt : 将bottleneck分为了32个分支，最终通过addition融合
   * 这两种结构后也接crop操作，去除padding影响。这两种复杂结构可以学习更加复杂的feature
4. 最终，提到了crop之后feature map变小的解决方案--增大input image的大小，减小网络stride，surprise！？感觉应该有其他的方式来做padding比如说feature的均值填充之类的，没有试过不知道会不会有效果。

## result
1. 最好结果相对SiamRPN而言，提升了4个百分点，为0.301。

## other points
1. siamese网络的输入就是图像对，之前DaSiam就是在图像对这里使用的数据增强
2. 如果我将z的信息提取hog或者就是feature特征，然后作为网络的input，在之后帧中判断其与之前的cosine distance,这种方案感觉也可行。
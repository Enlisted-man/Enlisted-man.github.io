---
title: Bottleneck-LSTM-CVPR2018-VideoDetection
date: 2019-05-21 14:20:33
categories: paper
tags: Video Detection
---
##  Bottleneck-LSTM-2018
- Bottleneck-LSTM：Mobile Video Object Detection with Temporally-Aware Feature 
- Paper: https://arxiv.org/abs/1711.06368
- Code: https://github.com/tensorflow/models/tree/master/research

## Overview

1. 构建Conv Bottleneck-LSTM，并将Bottleneck-LSTM与one stage检测相结合。其中，Bottleneck-LSTM借鉴了深度可分离卷积和bottleneck。
2. 在video detection中的速度，相比现存检测算快，FPS为15（个人感觉比较慢）。

<!--more-->

## why
1. 现存常规的检测器，针对视频检测速度较慢，无法达到实时。
2. 有一部分没有考虑视频检测中的时序信息。
3. 同时部分使用**光流**的方法存在速度较慢的劣势。
4. 使用LSTM的方法并没有将LSTM和卷积网络完全融合。
	* LRCNs将每帧的feature序列输入LSTM
	* ROLO使用YOLO做检测，然后将检测的BBox和feature送入到LSTM

## what
1. 使用CNN提取图像特征，然后将特征送入convolutional LSTM进行历史帧特征融合（记忆重要信息，特征结合），输出。
![Framework](Bottleneck-LSTM-CVPR2018-VideoDetection\main.PNG "Framework")

## how
1. 大体思路。使用前t帧的图像，前t-1帧的feature，来获取当前帧的bboxes和分类置信度。
**F(I_t,S_t-1)  = (D_t , S_t)**   
其中，I_t表示第t帧的视频序列，S_t-1表示前t-1帧的feature，D_t表示相对于I_t帧的bboxes和confidences集合
2. LSTM-SSD中SSD的网络结构更换为MobileNet，并使用深度可分离卷积代替了所有的常规卷积。  
![LSTM-SSD](Bottleneck-LSTM-CVPR2018-VideoDetection\framework.PNG "LSTM-SSD")
3. Bottleneck-LSTM。借鉴了深度可分离卷积，bottleneck。
* Conv LSTM的定义。   
其中，M，N分别为LSTM输入/输出的通道数。LSTM将x_t和h_t-1(featue map)进行channel-wise操作，输出h_t和c_t，W(j,k)⭐X代表深度可分离卷积操作，j表示输入通道，k表示输出通道，o表示矩阵对应元素相乘，∮表示ReLU激活函数。
![Conv-LSTM](Bottleneck-LSTM-CVPR2018-VideoDetection\Conv-LSTM.PNG "Conv-LSTM")
* 使用如下的b_t代替所有门的输入。
![b_t](Bottleneck-LSTM-CVPR2018-VideoDetection\b_t.PNG "b_t")
* 添加通道调整超参数a_base = a,a_lstm = 0.5a,a_ssd = 0.25a来分别控制不同网络模块通道维度，从而来控制网络的计算量。
![BottleNeck-LSTM](Bottleneck-LSTM-CVPR2018-VideoDetection\BottleNeck-LSTM.PNG "BottleNeck-LSTM")
* 优势：减少了标准LSTM中的计算；BottleNeck-LSTM的网络结构更深，效果更好。
4. 具体实现
* 单个LSTM模块的网络结构。

  ![single-lstm-network](Bottleneck-LSTM-CVPR2018-VideoDetection\single-lstm-network.PNG "single-lstm-network")
* 首先finetune 无LSTM的SSD网络，freeze基准网络（到conv13）,然后在train的时候添加LSTM模块。最终在conv13之后的所有feature map后都添加了LSTM模块作为最终模型。

## result
1. 单个LSTM模块，mAP相对提升3.2%   
![single-LSTM](Bottleneck-LSTM-CVPR2018-VideoDetection\single-LSTM.PNG "single-LSTM")
2. 单个LSTM模块，在参数量，和计算量方面的对比，大约可以减少10-20倍的参数量，减少20倍左右的计算量。其中MAC代表multi-adds。
![deferent layers](Bottleneck-LSTM-CVPR2018-VideoDetection\layers-compare.PNG "deferent layers")
3. 多个LSTM模块，LSTM模块的重复堆叠没有提升，多层放置LSTM的效果会比单个放置提高0.9%，计算量不会有太大变化。   
![multi-LSTMs](Bottleneck-LSTM-CVPR2018-VideoDetection\multi-LSTMs.PNG "multi-LSTMs")

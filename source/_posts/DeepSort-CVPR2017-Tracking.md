---
title: Deepsort-CVPR2017-Tracking
date: 2019-05-20 10:36:46
categories: paper
tags: Tracking
---
## Deep Sort-2017
* DeepSort：Simple Online and Realtime Tracking with a Deep Association Metric 
* Paper：https://arxiv.org/abs/1703.07402
* Code：https://github.com/nwojke/deep_sort


## Overview
1. 在Sort的基础上整合了appearance信息，引入了深度信息
2. 将计算复杂度较高放在了离线预训练阶段，在大规模reid的数据集上学习一个关联矩阵，在online阶段，在visual appearance阶段建立了一个mearsure to track的最近邻查询。
3. 减少了ID switch

<!--more-->

## Sort with Deep Association Metric
### 跟踪处理 + 状态估计
1. 使用八个维度值(cx，cy，r，h，vcx，vcy，vr，vh)来估计tracking的状态，同时假设kalman filter是恒定速度和线性观测模型。
2. 对于每一个track，都要计算上次匹配成功后的帧数a。当再次关联的时候置为0，目的是通过计数找出目标消失的情况。当a > 设定Aage时，我们认为该track在image中消失。对于未关联的detection，我们认为其为新出现的目标。这些目标在前3帧中被认为暂定状态，当这些新出现的track在他们出现的前三帧中未成功关联就被删除。
   
### 匹配问题
1. 在kalman filter预测和detection状态之间的分配问题一般使用Hungarian算法。**文章中在这里整合了运动信息，外观特征信息**。
2. 运动信息，通过计算kalman states和detection之间的马氏距离。**这里有公式d(1)(i,j) = (dj −yi)T^Si−1(dj −yi)**。马氏距离通过测量检测远离平均轨道位置的标准偏差的多少来考虑状态估计不确定性。之后通过x2分布来过滤一些不可能存在的关联，**这里有公式b(1) i,j = 1[d(1)(i,j) ≤ t(1)]**。其中的Mahalanobis的阈值t(1)设定为9.4877。
* [注]以上的马氏距离可以无相机运动的时有较好的预测效果。但是在相机快速运动，遮挡的情况下，Mahalanobis距离相当不准确。
3. 外观特征，对于每一个detection dj提取外观特征，表示为rj，且||rj|| = 1，我们为每一个track维护它的最近Lk = 100个特征。第二个度量是计算track_i和detection_j在外观特征上的cosine distance。之后再通过x2分布进行过滤。
* [注]同一物体在不同image中生成的feature向量的余弦距离是很小的。
4. lamda*运动信息 + (1-lamda)外观特征。运动信息适合短期预测，外观特征适合长期预测。在paper中，将lamda设为0，但是mahalanobis gate仍然被用来判断不可行的预测结果。

### 级联匹配
1. 原因：在物体长时间遮挡的情况下，kalman预测会增加物体位置预测的不确定性。概率会在状态空间发散，可能无法观察。且当两个track被关联为同一个detection，mahalanobis distance偏向于不确定性更大的track，因为它有效地减少了任何检测的标准偏差与投影轨道平均值之间的距离。    
   因此，采用级联的方式设定优先级。
2. 级联匹配，优先考虑最小age的track。
3. 在最终的match阶段，统计未确认，和age为1的未匹配的track。这个操作有助于对突然的外观变化等增加一定的鲁棒性。

### 深度外观描述子
1. 使用最近邻查询，所以要提取较好的具有判别力的特征。文章中使用CNN在大规模的reid数据集上进行训练。
2. CNN网络结构使用resnet，两个conv layer加六个res blocks。最后在dense layer计算出来128维的特征映射。
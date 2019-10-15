# Stacked Hourglass Networks for Human Pose Estimation

本文作为姿态估计的开山之作，提出一个卷积神经网络，对所有尺度上的特征进行处理和整合，以最佳地捕捉与身体相关的各种空间关系。文章展示了重复的自下而上、自上而下的处理和中间监控是如何提高网络性能的。该体系结构被称为“叠加沙漏”网络，它基于为生成最终预测集而进行的池和上采样的连续步骤。

### 沙漏设计

沙漏设计的灵感来自于网络需要捕捉不同尺度的信息，局部信息对于识别人脸及手等部位很重要，完整的人体理解需要这些局部的信息，人的方位、四肢的排列以及相邻关节的关系是在图像中不同尺度下最容易识别的线索之一。沙漏结构很适合输出这样像素级别的预测。

与那些输入不同尺度的图片最后再结合起来的网络不同，本文使用沙漏结构以解决多尺度问题，通过使用单一的pipeline以及skip connection来保持不同分辨率的空间结构。

#### hourglass设置： 
1. Conv层和Max Pooling层用于将特征缩放到很小的分辨率； 
2. 每一个Max Pooling(降采样)处，网络进行分叉，并对原来pre-pooled分辨率的特征进行卷积； 
3. 得到最低分辨率特征后，网络开始进行upsampling，并逐渐结合不同尺度的特征信息. 这里对较低分辨率采用的是最近邻上采样(nearest neighbor upsampling)方式，将两个不同的特征集进行逐元素相加. 
4. 整个hourglass是对称的，获取低分辨率特征过程中每有一个网络层，则在上采样的过程中相应低就会有一个对应网络层.

5. 得到hourglass网络模块输出后，再采用两个连续的 1×1 Conv层进行处理，得到最终的网络输出. 
   Stacked Hourglass Networks输出heatmaps的集合，每一个heatmap表征了关节点在每个像素点存在的概率.

### 中继监督

Hourglass网络输出heatmaps集合(蓝色方框部分)，与真值进行误差计算。 其中利用1×1的Conv层对heatmaps进行处理并将其添加回特征空间中，作为下一个hourglass model的输入特征。每一个Hourglass网络都添加Loss层.Intermediate Supervision的作用在[2]中提到：*如果直接对整个网络进行梯度下降，输出层的误差经过多层反向传播会大幅减小，即发生vanishing gradients现象。* 

*为解决此问题，[2]在每个阶段的输出上都计算损失。这种方法称为intermediate supervision，可以保证底层参数正常更新。* 

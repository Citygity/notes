# Boundary-Aware Network for Fast and High-Accuracy Portrait Segmentation

### abstract

本文提出一个轻量级的网络Boundary-aware网络（BANet），可以选择性地提取边界区域的细节信息使得图像可以输出一个高质量的边缘，此外设计了一个refine loss，用来监督网络使用图片级别的梯度信息。本文的方法可以产生一个更精细的分割结果（比标注有更多的细节）。

### introduction

作者认为丢失边界信息主要由于两个原因：

1. CNN网络的结果非常依赖于大量的训练数据，而分割的数据集通常是多边形或者是KNN聚类得到的。因此基本不可能标记精细的边缘，比如头发。

2. 尽管先前的人像分割方法提出了一些有用的预处理方法或者是loss function，但是依然需要使用传统的方法，比如FCN，DeepLab，PSPNet来作为baseline。这就导致分割结果被下采样（4或者16倍）。因此无法保留很好的边缘信息。传统的分割算法着重解决类内一致性以及类间差别。然而人像分割是一个两类别的网络，同时需要速度和精度。

   ### proposed network

   在人像分割的任务中，非边界的区域需要一个很大的感知野来预测全局上下文信息，然后边界区域需要晓的感知野来集中注意力于局部细节。因此他们两个需要被分开对待。本文退出一个边界attention机制以及一个加权loss function来处理边界区域以及非边界区域。

   本文pipeline如图3，首先彩色图像通过语义分支得到一个1/4原尺寸大小的high-level语义特征图。然后语义分支的输出被投影到一个one channel的特征图上，并且被上采用到原图大小作为boundary attention map，这个boundary attention map被监督通过boundary attention（BA） loss，此时，在边界特征挖掘分支，我们concatenate输入图片和boundary attention map以挖掘低层次的特征更加有指向性。最后，混合区域混合了高层次的语义特征以及低层次的细节以产生更好的分割结果。最终的分割结果通过两个loss来监督，segmentation loss被用来控制人像分割的整个流程，refine loss refine边界细节。

   ![](http://pqz0lv0o0.bkt.clouddn.com/fig3.png)

   **Semantic Branch：**本分支主要是为了获得一个可靠的特征表达，整个特征表达主要是通过高层次的特征语义信息来主导的。

   **Boundary feature mining branch**：semantic branch 的输出被投影为1channel通过1*1卷积，此时它被双线性插值，到完整尺寸作为boundary attention map，BA loss用来知道attention map定位boundary 区域，，boundary区域的target 可以被生成不需要手工标注。如fig4，首先我们使用canny边界检测器来在人像标注上提取semantic edge。考虑到人像区域在不同的图片上变化很大，所以把不同的图片上扩大到相同的宽度是不合理的。所以我们把不同图片的边缘用不同的kernelsize扩大。如下图

   ![](http://pqz0lv0o0.bkt.clouddn.com/Kvalue.png)

   W是一个经验值，表示标准宽度，本文设为50.BA loss是一个binary cross-entropy loss，但是我们不想要boundary attention map二值化，因为空间上的不平滑可能会导致数值不稳定，因此我们使用一个sigmoid函数来soft输出。图中的T是一个调节用的值以产生更soft的概率分布。最后，输入图片和attention map被cat在一起，作为一个4 channel 的图片，整个4channel的图片通过一个卷积层以提取更细节的信息。和别的多分支各自独自处理结果的网络不同，本文的低层次的特征是由高层次的特征来指导的。

   


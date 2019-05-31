# training protocol of DeepLab v3+

本篇文章来自于Deeplab v3+

- 学习率：使用poly学习率，初始学习率×(1-(iter/maxiter))<sup>power</sup>,power=0.9

- crop size：在训练时对图片进行了裁剪，为了让rate比较高的ASPP更加有效（裁剪的patch太小会导致ASPP卷的都是0padding的区域），裁剪的尺寸为513\*513
- batch normalization：本文所有的模块都是在resnet上添加的，包括bn的参数。作者发现bn的参数也是需要训练的， 因为需要较大的batch size来训练bn的参数，所以作者使用output_stride（输入尺寸/输出尺寸）为16，并使用batch size=16来训练bn层，bn层的参数使用decay=0.9997，在 trainaug训练集上用初始学习率0.007，之后用30k次迭代来训练，之后freeze bn的参数，使用output_stride=8，然后在官方VOC2012trainval 数据集上训练30k次，base learning rate =0.001，因为作者使用的是ASPP，所以作者可以控制不同训练阶段的输出output_stride不需要额外的参数。然后output stride为16时会快很几倍。因为输出特征图的小了很多，因为output stride为16时会得到更加粗糙的特征图。
- 上采样logits：在之前的工作中，gt被下采样为1/8，在训练阶段output stride为8时，之后作者采用了把logits上采样，而不是下采样gt，因为这样会把gt中的精细的标注给浪费掉，导致没法反向传播细节。
- 数据增强：作者在训练时使用随机缩小放大输入图片(0.5-2)，并且随机左右水平翻转图片。


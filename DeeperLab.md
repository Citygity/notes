# DeeperLab

Panoptic segmentation :作为图像语义分割任务的泛化，对于stuff类的物体进行分割，对于thing类的物体

进行实例分割。对图片中的物体分配both语义和实例标签。目前的方法都是使用独立的模块对constitutent语义和实例分割任务，本文提出联合解决语义分割和实例分割的image parser DeeperLab。





## Encoder

本文实验了两个建在深度可分网络上的网络，标准的Xceotion-71以获得更高的准确度，以及MobileNetV2全新的变体以获得更快的inference。尽管标准的MobileNetV2在ImageNet上效果非常好，但是它的感知野太小，对于image parser来说并不够，因为image parser通常会使用高清图像，例如1441×1441，MobileNetV2的感知野太小，在Imagenet 224×224上，它表现的很棒，按时他对于长距离的上下文信息捕捉并不优秀，因为有限的感知野（491×491），通常来讲，堆叠3x3的卷积核可以增加感知野，但是额外的层会导致更多的feature map，这主宰着内存使用，鉴于有限的计算资源，本文使用5×5卷积来代替3×3，这样的做法把感知野增加到了981*981，然后又保持了特征图相同大小的内存使用，并只是小小地增加了计算消耗。本文还采用了ASPP来增强network backbone，ASPP也能显著地增加感知野。

encoder输出的特征图下采样率为16


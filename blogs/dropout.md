# Drpout

Dropout 和 Inverted Dropout 现在统一被叫做Dropout，但是Dropout是传统的方法， Inverted Dropout 是吴恩达讲的方法，两者本质上是一样的。具体原理请参考上一篇文章。

首先AlexNet用的是传统的dropout，在训练阶段，对应用了dropout的层，每个神经元以keep_prob的概率保留（或以1-keep_prob的概率关闭），然后在测试阶段，不执行dropout，也就是所有神经元都不关闭，但是对训练阶段应用了dropout的层上的神经元，其输出激活值要乘以keep_prob。

AlexNet论文发表于2012年，而现在主流的方法是Inverted dropout，和传统的dropout方法有两点不同：

在训练阶段，对执行了dropout操作的层，其输出激活值要除以keep_prob
测试阶段则不执行任何操作，既不执行dropout，也不用对神经元的输出乘keep_prob。
在深入讲解这两点不同的细节之前，先贴上实现Inverted dropout的部分关键代码，虽然现在像tensorflow这样的框架已经很成熟，dropout只需一行代码就能实现，但是了解dropout的具体实现，能帮助我们后面更直观地理解dropout的数学原理，下面的代码出自cs231n notes Regularization 部分关于dropout的讲解，在学习过程中，我不止一次发现cs231n notes里面的一些讲解相较于吴恩达的深度学习课程更加深入原理，是吴恩达的深度学习课程很好的补充：

```python
""" 
Inverted Dropout: Recommended implementation example.
We drop and scale at train time and don't do anything at test time.
"""

p = 0.5 # probability of keeping a unit active. higher = less dropout

def train_step(X):
# forward pass for example 3-layer neural network
  H1 = np.maximum(0, np.dot(W1, X) + b1) #relu
  U1 = (np.random.rand(*H1.shape) < p) / p # first dropout mask. Notice /p!
  H1 *= U1 # drop!
  H2 = np.maximum(0, np.dot(W2, H1) + b2)
  U2 = (np.random.rand(*H2.shape) < p) / p # second dropout mask. Notice /p!
  H2 *= U2 # drop!
  out = np.dot(W3, H2) + b3

# backward pass: compute gradients... (not shown)
  # perform parameter update... (not shown)

def predict(X):
# ensembled forward pass
  H1 = np.maximum(0, np.dot(W1, X) + b1) # no scaling necessary
  H2 = np.maximum(0, np.dot(W2, H1) + b2)
  out = np.dot(W3, H2) + b3
```


         Inverted-dropout的基本实现原理是在训练阶段每次迭代过程中，以keep_prob的概率保留一个神经元（也就是以1-keep_prob的概率关闭一个神经元），上述代码中利用numpy的具体实现方式为：用U1=(np.random.rand(*H1.shape) < p) / p得到一个mask，再用神经元输出的激活值乘以这个mask，这里numpy.random.rand得到的是一个满足0到1的均匀分布的数组，注意numpy.random.rand和numpy.random.randn两者的区别，后者得到的是标准正态分布的数组。np.random.rand(*H1.shape) < p 得到的是一个布尔值数组，当其元素值小于p时是True，大于p时是False（这里遇到一个python和JavaScript语法不同的地方，就是True和False的首字母必须大写）。那么后面为什么还要除以 p 呢？吴恩达在课里讲的是为了保证神经元输出激活值的期望值与不使用dropout时一致，我们结合概率论的知识来具体看一下：假设一个神经元的输出激活值为a，在不使用dropout的情况下，其输出期望值为a，如果使用了dropout，神经元就可能有保留和关闭两种状态，把它看作一个离散型随机变量，它就符合概率论中的0-1分布，其输出激活值的期望变为 p*a+(1-p)*0=pa，此时若要保持期望和不使用dropout时一致，就要除以 p。

        了解了上述数学细节，我们再回过头来看看AlexNet里传统的dropout，在训练阶段应用dropout时没有让神经元的输出激活值除以 p，因此其期望值为 pa，在测试阶段不用dropout，所有神经元都保留，因此其输出期望值为 a ，为了让测试阶段神经元的输出期望值和训练阶段保持一致（这样才能正确评估训练出的模型），就要给测试阶段的输出激活值乘上 p ，使其输出期望值保持为 pa。

       看到这里你应该已经发现，传统的dropout和Inverted-dropout虽然在具体实现步骤上有一些不同，但从数学原理上来看，其正则化功能是相同的，那么为什么现在大家都用Inverted-dropout了呢？有两点原因：

      1、测试阶段的模型性能很重要，特别是对于上线的产品，模型已经训练好了，只要执行测试阶段的推断过程，那对于用户来说，当然是推断越快用户体验就越好了，而Inverted-dropout把保持期望一致的关键步骤转移到了训练阶段，节省了测试阶段的步骤，提升了速度。

      2、dropout方法里的 keep_prob 是一个可能需要调节的超参数，用Inverted-dropout的情况下，当你要改变 keep_prob 的时候，只需要修改训练阶段的代码，而测试阶段的推断代码没有用到 keep_prob ，就不需要修改了，降低了写错代码的概率。

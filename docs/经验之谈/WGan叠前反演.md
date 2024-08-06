## WGan反演三参数

[生成对抗网络（GANs）—— WGAN 介绍](https://zhuanlan.zhihu.com/p/564784629)

[WGAN的来龙去脉](https://zhuanlan.zhihu.com/p/58260684)

[令人拍案叫绝的Wasserstein GAN](https://zhuanlan.zhihu.com/p/25071913)


WGAN在实践中有一个重要的限制，即判别器有一个重要限制需满足K-Lipschitz条件（一阶导数处处不大于K）。通常来说有两种方法迫使WGAN满足条件，一是权重剪裁（Weight Clipping），二是梯度惩罚（Gradient Penalty）。

正由于WGAN需要满足K-Lipschitz条件，因此WGAN模型中不能使用Sigmoid函数作为激活函数，所以网络不应对输出值进行 0～1 的限制，且由于判别器的损失函数加入了梯度惩罚项，且梯度惩罚项对每一个输入样本的梯度进行限制，因此在判别器的网络结构中不应该加入Batch-Normalization（批归一化，简称BN）层，BN会将同一批其他样本的信息融入到对单个样本的梯度计算中，破坏样本间的独立性，此时算出来的梯度并不是真实的判别器对单个样本的梯度。

D的输出不再是概率，我们不在D的输出处应用sigmoid。


开源代码

官方代码

https://github.com/martinarjovsky/WassersteinGAN/tree/master

https://github.com/caogang/wgan-gp/tree/master

https://github.com/Zeleni9/pytorch-wgan/tree/master

https://necromuralist.github.io/Neurotic-Networking/posts/gans/wasserstein-gan-with-gradient-penalty/index.html

博客

https://www.qixinbo.info/2021/10/21/gan/


## 网络驱动叠前反演

### 监督反演

深度学习训练过程就是不断优化网络结果和标签之间的差异，所以只要训练样本和标签只要足够大，一定可以训练出合适的网络（数据驱动）。但是对于地震反演，比如说输入是地震数据，输出是速度模型，二者本身存在物理关系，且正过程可以通过数学实现，网络只是用来学习地震数据到标签的映射（逆过程），在标签很少的情况，我们可以将标签正演到地震数据（合成数据），计算此时计算合成数据
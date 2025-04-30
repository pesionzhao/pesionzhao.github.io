参考链接：

https://github.com/leeroee/NN-by-Numpy

https://github.com/Phoenix8215/BuildCudaNeuralNetworkFromScratch

https://github.com/SmartFlowAI/LLM101n-CN/blob/master/micrograd/micrograd.cpp

在了解基础的CUDA编程之后，我们可以开始构建一个简单的神经网络训练框架。

仿照pytorch, 首先要定义tensor, 他也是网络运算的基本单元

目前只实现二维张量

```c++
template<typename T>
class Tensor{
public:
    int rows, cols;
    std::shared_ptr<T> data_host;
    std::shared_ptr<T> data_device;
    Tensor(int rows, int cols):rows(rows), cols(cols){}{};
    void allocate(){};
    void copyHostToDevice(){};
    void copyDeviceToHost(){};
    //运算符重载
    Tensor operator+(Tensor& other){}
    Tensor operator-(Tensor& other){}
    Tensor operator*=(T other){}//乘标量
}
```

### layer的forward与backward公式推导

#### 线性层一 

forward:

$y = Wx + b$ 

backward:

$\frac{\partial L}{\partial W} = \frac{\partial L}{\partial y} \frac{\partial y}{\partial W} = \frac{\partial L}{\partial y} X^T$

$\frac{\partial L}{\partial b} = \frac{\partial L}{\partial y} \frac{\partial y}{\partial b} = \frac{\partial L}{\partial y}$

$\frac{\partial L}{\partial x} = \frac{\partial L}{\partial y} \frac{\partial y}{\partial x} = W^T \frac{\partial L}{\partial y}$

这里的左乘右乘可能会一头雾水，说白了就是要对应元素相乘，矩阵的元素用偏导数的形式写出来就会懂了。

pytorch的线性层为什么不是`y=W*x+b` 而是 `y = x*W^T+b`?

说白了就是把列向量当成行向量

行优先符合主流的编程规范，比如sum, max, 所以pytorch将优先的行向量作为特征，行数为批次, 并且按道理存数据时，单个数据内存要连续，如果行数为特征，列数为批次的话，单个数据的不同特征内存不连续

#### softmax
[反向传播之一：softmax函数](https://zhuanlan.zhihu.com/p/37740860)
[SoftMax反向传播推导，简单易懂，包教包会](https://www.bilibili.com/video/BV143411h7PQ/?spm_id_from=333.337.search-card.all.click&vd_source=c43347ef375755d298da8f0c05cfe444)

forward:

$y_i = \frac{e^{x_i}}{\sum_{j=1}^{n} e^{x_j}}$

backward:

$\frac{\partial y_i}{\partial x_j}=\left\{ \begin{aligned} &=y_i-y_iy_i ,当i=j\\ &= 0-y_i\cdot y_j ， 当 i \ne j \\ \end{aligned} \right.$

### loss

#### MSE

$$
L = \frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2
$$

其中：
- \( y_i \) 是真实值
- \( \hat{y}_i \) 是预测值
- \( n \) 是数据点的数量

对 \( \hat{y}_i \) 进行求导：

$$
\frac{\partial L}{\partial \hat{y}_i} = \frac{2}{n} (\hat{y}_i - y_i)
$$
#### CrossEntropy

多类分类问题，对于一个单一样本的交叉熵损失函数，假设真实标签是 \( y \)（是一个 one-hot 编码的向量），模型输出的概率分布是 \( \hat{y} \)，交叉熵定义为：

$$L = -\sum_{i=1}^{n}y_i\ln(\hat y_i)$$

对于多类交叉熵损失函数，对预测概率 \( \hat{y}_i \) 求导数：

\[
\frac{L(y, \hat{y})}{\partial \hat{y}_i} = - \frac{y_i}{\hat{y}_i}
\]

### optimizer

[十分钟搞明白Adam和AdamW，SGD，Momentum，RMSProp，Adam，AdamW](https://www.bilibili.com/video/BV1NZ421s75D/?spm_id_from=333.337.search-card.all.click&vd_source=c43347ef375755d298da8f0c05cfe444)


遇到的一些坑以及解决方法

1. MSE的反向传播要除以元素个数，否则回传的梯度过大，导致梯度爆炸
2. 线性层的定义注意是Y=XW+b, 对注意各个参数的偏导数计算
3. 神经网络权重的初始化，正态分布，如果方差过大同样导致梯度爆炸，参考torch的初始化，0均值0.01方差

TODO 

1. cuda端和cpu端的区分
2. 如果矩阵很大，所有线程铺不满，就要一个线程处理多个元素，比如copyKernel, ramdonInitKernel, 也就是要跳跃blockDim.x*gridDim.x个元素

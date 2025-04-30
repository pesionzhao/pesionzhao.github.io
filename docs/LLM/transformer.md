## Transformer

论文链接：https://arxiv.org/pdf/1706.03762

参考链接：

[Transformer升级之路：1、Sinusoidal位置编码追根溯源](https://spaces.ac.cn/archives/8231)

[位置编码之路：SIN->ALiBi->RoPE ->PI->NKT->YARN](https://zhuanlan.zhihu.com/p/1894384438206505105)

#### 位置编码 

**输入表示**
设输入序列为 $\mathbf{X} \in \mathbb{R}^{n \times d}$，其中 $n$ 是序列长度，$d$ 是特征维度。  加入位置编码后，输入变为：

$$\mathbf{\tilde{X}} = \mathbf{X} + \mathbf{P}$$

其中 $\mathbf{P} \in \mathbb{R}^{n \times d}$ 是位置编码矩阵。

通过 Query、Key、Value 矩阵计算注意力：

$$
\begin{aligned}
\mathbf{Q} &= \mathbf{\tilde{X}} \mathbf{W}_Q, \quad \mathbf{W}_Q \in \mathbb{R}^{d \times d_k} \\
\mathbf{K} &= \mathbf{\tilde{X}} \mathbf{W}_K, \quad \mathbf{W}_K \in \mathbb{R}^{d \times d_k} \\
\mathbf{V} &= \mathbf{\tilde{X}} \mathbf{W}_V, \quad \mathbf{W}_V \in \mathbb{R}^{d \times d_v}
\end{aligned}
$$

注意力分数为：

$$
\text{Attention}(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \text{softmax}\left(\frac{\mathbf{Q} \mathbf{K}^T}{\sqrt{d_k}}\right) \mathbf{V}
$$

**相对位置编码的推导与关系**
相对位置编码的核心思想是：**建模位置 $i$ 与 $j$ 之间的相对距离 $i-j$**，而非绝对位置。

$$
\begin{align}
&\mathbf{Q}_i \mathbf{K}_j^T =  (\mathbf{X}_i + \mathbf{P}_i)\mathbf{W}_Q *(\mathbf{X}_j + \mathbf{P}_j)^T\mathbf{W}_K^T \notag \\
& = \mathbf{X}_i \mathbf{W}_Q \mathbf{X}_j^T \mathbf{W}_K^T + \mathbf{X}_i \mathbf{W}_Q \mathbf{P}_j^T \mathbf{W}_K^T + \mathbf{P}_i \mathbf{W}_Q \mathbf{X}_j^T \mathbf{W}_K^T + \mathbf{P}_i \mathbf{W}_Q \mathbf{P}_j^T \mathbf{W}_K^T \notag
\end{align}
$$

这里我们要求

$$ \mathbf{P}_i \mathbf{W}_Q \mathbf{P}_j^T \mathbf{W}_K^T = g(i-j) $$

简化为

$$ \mathbf{P}_i \mathbf{P}_j^T = g(i-j) $$

找到满足上式的$g$即可， Sinusoidal（正弦位置编码）就是一种解

**Sinusoidal**

将向量[x, y]视作复平面上的向量x+yi, 所以

\[ \langle p_m, p_n \rangle = \text{Re}\left[ p_mp_n^*\right] \]

借助三角函数，$\cos \theta_i \cos \theta_j + \sin \theta_i \sin \theta_j = \cos ( \theta_i - \theta_j )$， 可以满足g的关系式

\[p_{pos} = e^{ipos\theta} =\left( \begin{array} { l } { \cos {pos} \theta } \\ { \sin {pos} \theta } \\ \end{array} \right)\]

定义 $\theta = \frac{2 \pi}{10000^{2i/d}}$, 对于一个pos，对应一个正弦分量和一个余弦分量(复平面上的实部和虚部)，所以对于嵌入维度为\( d_{\text{model}} \)的位置编码有\( d_{\text{model}}/2 \)个\(p_{pos}\)，每个维度的\(\theta\)不同，如下式

对于一个位置 \( pos \)，在维度 \( i \) 上的位置编码定义如下：

\[
PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{\frac{2i}{d_{\text{model}}}}}\right)
\]
\[
PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{\frac{2i}{d_{\text{model}}}}}\right)
\]

- `pos`  序列中 token 的位置（从 0 开始） 
- `i`  embedding 的维度索引（从 0 到 \( d_{\text{model}} - 1 \)）
- \( d_{\text{model}} \)  输入嵌入的维度
- `10000` 控制不同维度的频率的基数，使每一维有不同的周期，形成可区分的波形 

假设相对位置 $k = i-j$，则 $g(k)$ 需满足：
- **对称性**：$g(k) = g(-k)$（若位置方向无关）
- **衰减性**：随着 $|k|$ 增大，$g(k)$ 的幅值应减小（远距离位置相关性低）

pytorch实现，源自GPT

```python
import torch
import math

def get_sinusoidal_encoding(max_len, d_model):
    """
    生成 [max_len, d_model] 大小的位置编码矩阵
    :param max_len: 序列的最大长度
    :param d_model: 每个 token 的 embedding 维度
    :return: Tensor of shape [max_len, d_model]
    """
    pe = torch.zeros(max_len, d_model)
    position = torch.arange(0, max_len).unsqueeze(1).float()  # [max_len, 1]
    div_term = torch.exp(torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model))  # [d_model/2]

    # sin 用于偶数维，cos 用于奇数维
    pe[:, 0::2] = torch.sin(position * div_term)  # 偶数位
    pe[:, 1::2] = torch.cos(position * div_term)  # 奇数位

    return pe  # [max_len, d_model]

# 示例用法：
max_len = 100  # 句子最大长度
d_model = 512  # 嵌入维度
pos_encoding = get_sinusoidal_encoding(max_len, d_model)  # shape: [100, 512]

# 可以将其添加到你的 embedding 上：
# embedding_output = word_embedding + pos_encoding[:seq_len]
```
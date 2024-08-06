## PAPER1: Approximations to the Zoeppritz equations and their use in AVO analysis

Zoeppritz方程的近似值及其在AVO分析中的应用(AVO:Amplitude variation with offset)

## 摘要
推导了p-p波反射和透射系数(($T_{pp},R_{pp}$))与弹性参数相对对比度的二次表达式。提供了Zoeppritz方程的替代近似值，$p^2,p^4$项的显式系数.这是一个紧凑的形式。
传统线性近似只能反演两个弹性系数。通过二阶近似，Frechet矩阵对三个弹性参数的条件数比线性近似有明显改善。
(弹性对比项：p波对比,s波对比，界面处ps比值,即$\varDelta\alpha / \alpha,\varDelta\beta/\beta,\alpha/\beta$
弹性参数：体密度，P波速度，S波速度)

> those quadratic approximations can be used directly with amplitude information to estimate not only two but three parameters: P-wave velocity contrast, S-wave velocity contrast, and the ratio of S-wave and P-wave velocities at an interface. 

## 前言
**传统AVO**
假设：除反射系数外，所有的与偏移相关的振幅效应都被矫正，在这种情况下弹性参数的对比可以从反射振幅理论上估计。需要考虑（反射与传输系数，结构影响）
在反演中，通常采用**精确**的Zoeppritz方程来计算反射和折射系数。近似的好处是振幅的以弹性参数为基础的Frechet导数的灵敏度矩阵可以解析计算出来。

- Ursin and Dahl (1992)
泰勒级数的系数直接由介质参数计算((not from the vertical slownesses)，缺点：冗长，只能计算机实现
- Mallick (1993)
也做了$R_{pp}$近似，省略了二次项

## Zoeppritz方程
$$R_{pp}(p)=\frac{E+Fp^2+Gp^4-Dp^6}{A+Bp^2+Cp^4+Dp^6}\tag {1}$$

$$T_{pp}(p)=\frac{H+Ip^2}{A+Bp^2+Cp^4+Dp^6}\tag {2}$$

p为射线系数，将分母在$p=0$处进行四阶泰勒展开

...
...
...


$R_f$为流体-流体反射系数，即S波在两介质中速度为0时的反射系数，此时传输系数为$T_f=1-R_f$

## 伪四次近似(本文方法)
$$R_{pp}\approx\left[\frac{1}{2}-2\left(\frac{\beta}{\alpha}\right)^2\sin^2\theta\right]\frac{\varDelta\rho}{\rho}+\frac{1}{2}\sec^2\theta\frac{\varDelta\alpha}{\alpha}\\
-4\left(\frac{\beta}{\alpha}\right)^2\sin^2\theta\frac{\varDelta\beta}{\beta}+\left(\frac{\beta}{\alpha}\right)^3\cos\theta\sin^2\theta\left(\frac{\varDelta\rho}{\rho}+2\frac{\varDelta\beta}{\beta}\right)^2\tag {3}$$

$$T_{pp}\approx 1-\frac{1}{2}\frac{\varDelta\rho}{\rho}+\frac{1}{2}(\tan^2\theta-1)\frac{\varDelta\alpha}{\alpha}-\left(\frac{\beta}{\alpha}\right)^3\cos\theta\sin^2\theta\left(\frac{\varDelta\rho}{\rho}+2\frac{\varDelta\beta}{\beta}\right)^2\tag{4}$$

进一步近似，将(5)带入(3)(4)

$$\frac{\varDelta\rho}{\rho}\approx\frac{1}4\frac{\varDelta\alpha}{\alpha}\tag{5}$$


**线性近似方法**忽略了$\left(\frac{\varDelta\rho}{\rho}+2\frac{\varDelta\beta}{\beta}\right)^2$

## 对比
- 伪四次近似得到的$R_{pp}与T_{pp}$比线性近似更接近精确的zoeppritz结果
- 运用在反演问题中，离散化运算符L的特征值更大
- 伪四次近似由更小的条件数（conditio number） 即最大特征值与最小特征值之比

### 结语
引入伪四次逼近以射线参数p为参数的p-p波的反射和透射系数。这些近似具有相当紧凑的形式，其中p2和p4项的系数用垂直慢度表示

## PAPER2: AVO inversion using the Zoeppritz equation for PP reflections
利用Zoeppritz方程反演PP反射

### summary
在众多Zoeppritz近似方法中，偏移距离被限制到小或中等，角度被限制到很小，可逆弹性参数也被限制到两到三个，所以，我们建议使用精确的Zoeppritz方程对无临界角(大角度)的反射进行AVO反演。并进行了对比。最小二乘振幅拟合可求出反射面的密度比和三个速度比。该算法适用于长偏置PP反射的反演。

### 前言
反演通常通过最小二乘使观测值和合成AVO数据之间的差异最小化，观测数据输入之前，应进行预处理，即对包括几何扩散，衰减，色散和传输损失进行矫正(除了RCs:reflection coefficients)
偏移量越大，入射角范围越广,更大的角度范围使反演稳定，反演参数更多，所以重点是如何处理长偏移反射数据。长偏移反射不一定包括广角反射，根据定义，广角反射来自近临界入射角和后临界入射角。产生广角反射需要在相对较浅的深度处有较大正速度对比的模型，需要球波方法来处理这些数据。
本文考虑长偏移非临界PP反射，最终得到
- [x] 精确的Zoepptriz得到的平面波RCs对球波反射长偏移数据也是准确的
- [x] 精确的Zoepptriz方程能用在传统AVO中

### AVO Modeling

#### 精确Zoepptriz方程

$$R_{pp}=\frac{Q^2-r_4T_0T_3+r_4T_1T_2-(1+Q)^2T_0T_2+A-B}{Q^2+r_4T_0T_3+r_4T_1T_2+(1+Q)^2T_0T_2+A+B}$$

#### 有限微分
我们选择一种有限差分解来生成精确的AVO观测数据，用于测试的长偏移精度PP RCs。正演建模采用8阶交错网格弹性动力学方程的FD解。

#### 近似
近似的方法适用的角度都很小，线性近似只适用于小于20的角度，伪四次只适用于小于50的角度，所以精确的Zoepptriz有优势。

### AVO Inversion
通过由角度决定的PP反射系数，只能预测出三个速度的比值与密度比值。且四个比值不独立。$r_1,r_2,r_3由\alpha_!$归一化，$r_4由\rho_1$归一化，所以将$\alpha_1,\rho_1$当作先验，可通过数学工具(Mathematica)计算frechet导数矩阵，在导数计算中，需要确定最佳扰动，这个扰动要够小能到捕获局部梯度，但也不能小到引起高频伪影。

### 结语
一种新的AVO反演算法，使用**精确的Zoeppritz方程**，采用精确的长偏移PP反射(八阶FD模型)对算法进行了验证。
zoepptriz假设的平面波RCs在临界角附近不适用，因此上述模型具有衰减速度对比，因此观测数据中没有临界反射。对于具有正对比的模型，只要利用的角度范围不在临界角的第一个菲涅尔带内，该算法仍然有效。

## :small_orange_diamond:PAPER3 Constrained nonlinear amplitude variation with offset inversion using Zoeppritz equations

利用偏置反演约束非线性振幅变化Zoeppritz方程:

[论文链接](https://www.researchgate.net/publication/338883181_Constrained_nonlinear_amplitude_variation_with_offset_inversion_using_Zoeppritz_equations)

### 摘要

开发了一种有效的非线性AVO前临界反射反演算法，使用精确的多通道和多界面Zoeppritz方程同时估计p波速度，s波速度和密度。在求解正演非线性模型时，采用全变分约束克服了病态性，保持了参数空间中界面的锐度。
> We have developed an efficient algorithm for nonlinear AVO inversion of precritical reflections using the exact Zoeppritz equations in multichannel and multi-interface form for simultaneous estimation of the P-wave velocity, S-wave velocity, and density. The total variation constraint is used to overcome the ill-posedness while solving the forward nonlinear model and to preserve the sharpness of the interfaces in the parameter space.

### 前言

利用AVO的叠前地震反演可以对弹性参数即P波速度(α)、S波速度(β)和密度(ρ)给出可靠的估计。AVO主要受Zoeppritz方程决定，该方程描述了界面在给定角度的反射率RC与界面两侧介质的弹性特性之间的非线性关系。然而这个方程针对大尺度、多层次、多维的地球模型，很难具体求解。所以很多研究提出了Zoepptriz的各种近似。其中Aki and Richards (1980)的线性近似用的最广，尽管简化了计算，但是这些近似方法只适用于弹性特征对比较小的界面，较小入射角的情况。之后的研究基于精确的Zoepptriz方程进行非临界角的AVO反演，通过迭代的方法解决，利用神经网络求解弹性系数。

本文提出了一种高效的全变分算法(TV)基于Zoeppritz方程的正则化非线性多界面AVO反演。与标准AVO反演技术类似，考虑了预临界角的反射响应;利用线性近似的简易性，当反射振幅与弹性性质或其相反性质成线性关系时，使用线性近似求解。

通过(Levenberg, 1944)算法求解非线性Zoeppritz方程，同时使用(全局)线性逼近计算。方法重写为

- 第一步对模型参数进行更新
- 第二步对合成数据进行更新

这个策略可强加约束，以保证参数空间中界面的块状性质。
通过(Goldstein and Osher, 2009)迭代过程解决模型更新时的正则化问题。

### Zoeppritz equations
Zoeppritz于1919年提出的方程

$$\begin{align}&\left[ \begin{matrix}
\cos(\theta)&-\sin(\delta_r)&\cos(\theta_t)&\sin(\delta_t)\\
\sin(\theta)&\cos(\delta_r)&-\sin(\theta_t)&\cos(\delta_t)\\
-\cos(2\delta_r)&\frac{\beta_1}{\alpha_1}\sin(2\delta_r)&\frac{\rho_2\alpha_2}{\rho_1\alpha_1}\cos(2\delta_t)&\frac{\rho_2\beta_2}{\rho_1\alpha_1}\sin(2\delta_t)\\
\sin(2\theta)&\frac{\alpha_1}{\beta_1}\cos(2\delta_r)&\frac{\rho_2\beta_2^2\alpha_2}{\rho_1\beta_1^2\alpha_1}\sin(2\theta_t)&-\frac{\rho_2\beta_2\alpha_1}{\rho_1\beta_1\beta_1}\cos(2\delta_t)
\end{matrix} \right]\notag\\
&\pmb\times
\left[\begin{matrix}
R_{pp}\\
R_{ps}\\
T_{pp}\\
T_{ps}
\end{matrix}\right]
=\left[\begin{matrix}
\cos(\theta)\\
\sin(\theta)\\
\cos(2\delta_r)\\
\sin(2\theta)
\end{matrix}\right]\end{align}$$

### Aki and Richards equation

$$R_{pp}=\zeta(\overline\theta)\frac{\varDelta\alpha}{\overline\alpha}+\eta(\overline\theta)\frac{\varDelta\beta}{\overline\beta}+\xi(\overline\theta)\frac{\varDelta\rho}{\overline\rho}\tag2$$

与弹性参数比值呈线性关系

### Linearized equation

对上式进一步简化，通过三个近似$\kappa=\frac12(\frac{\beta_1^2}{\alpha_1^2}+\frac{\beta_2^2}{\alpha_2^2})\approx\frac{\beta^2}{\alpha^2}$, $\frac{\varDelta x}{\overline x}\approx\varDelta\ln x$, 平均角度$\overline\theta$近似于入射角$\theta$,得到

$$R_{pp}=\zeta(\theta)\varDelta\ln(\alpha)+\eta(\theta)\varDelta\ln(\beta)+\xi(\theta)\varDelta\ln(\rho)\tag3$$

这种近似在小角度时准确，但角度增大时会变得不准确。

### NONLINEAR AVO INVERSION 非线性AVO反演
迭代，类似于最小二乘法

$$\bold X^{k+1}=\bold X^k+(\bold F^T\bold F+\lambda \bold I)^{-1}\bold F^T(\bold Y-f(e^{\bold X^k}))
$$

低频模型通常由叠加速度或测井曲线计算得出，并用作先验信息。

#### TV regularization 全变分正则化

#### Connection to the Bregman iterative method 

#### Model update via split Bregman technique
在反演过程中对弹性参数进行不同的加权，该权重控制每个参数的收缩/正则化量

#### Stopping criterion 停止迭代准则
> based on Morozov’s discrepancy principle, the iteration is stopped at the index $k^*$

在误差项不是随机的实际应用中，可以根据可用测井（地面真实值）与相应的反向参数之间的距离的最小化来确定最优迭代。

### NUMERICAL IMPLEMENTATION
#### Simple acoustic-impedance inversion 声阻抗反演
基于Zoepptriz方程
#### Single-interface model

### 结语
本文提出的算法使用了一个全局线性近似；因此，它不需要在每次迭代时计算Jacobian/Hessian矩阵。使得该算法非常灵活和高效
> In this paper, we have introduced a new algorithm for nonlinear seismic AVO inversion, **using the exact Zoeppritz equations**, for simultaneous estimation of the P-wave velocity, S-wave velocity, and density. Our strategy **uses the traditional Aki and Richards equation to construct the Jacobian matrix** for solving the original nonlinear problem. This allowed for developing a very efficient algorithm for the TV-regularized nonlinear AVO inversion that is difficult to carry out by other means. The synthetic and field data demonstrated that the proposed algorithm can accurately invert the P-wave velocity, S-wave velocity, and density from precritical reflection data. The accuracy and efficiency of this method make it a promising algorithm for inverting large-scale seismic data sets

## PAPER4 基于叠前地震反演的流体识别方法研究进展






## PAPER7 Nonlinear multichannel impedance inversion by total-variation regularization
全变分正则化非线性多通道阻抗反演

### 摘要

声阻抗(AI)分析能够使地震反射数据映射到岩性，反射率计算AI需要求解一个有约束的非线性反问题。针对多通道形式的非线性阻抗问题，在总变差(TV)约束下，提出了两种有效的算法来恢复具有块状结构的阻抗图。

### Introduction

从带限地震记录中获得AI需要两步

- deconvolution 反褶积，用于补偿由震源产生的子波的带限性质所造成的记录地震记录的缺失频率
- inversion 用AI反演该反射系数

由于产生AI的递推公式是按迹进行的，所以不允许阻抗图的空间正则化。

反射系数与阻抗参数呈非线性关系；因此，需要求解一个有约束的非线性反问题来从反射率剖面中恢复一个正则化的阻抗剖面。

本文方法：从数据中提取反射率剖面。然后求解非线性阻抗问题，将生成的反射率模型反演为正则化的AI模型，使用全变分(TV)正则化来支持具有块状结构的AI模型。

假设 $x$ 是 $n$ 维列向量，则 $x^T$ 是 $n$ 维行向量，因此 $x^Tx$ 是一个实数。我们要求的是 $\frac{\partial (x^Tx)}{\partial x}$。

将 $x$ 表示为列向量形式，即 $x=[x_1, x_2, \cdots, x_n]^T$，则 $x^Tx$ 可以表示为：

$$ x^Tx = \sum_{i=1}^n x_i^2 $$

对于第 $i$ 个分量 $x_i$，我们有：

$$ \frac{\partial x_i^2}{\partial x_i} = 2x_i $$

因此，对于整个 $x^Tx$，我们有：

$$ \frac{\partial (x^Tx)}{\partial x} = [\frac{\partial (x^Tx)}{\partial x_1}, \frac{\partial (x^Tx)}{\partial x_2}, \cdots, \frac{\partial (x^Tx)}{\partial x_n}]^T = [2x_1, 2x_2, \cdots, 2x_n]^T = 2x $$

因此，$x^T$ 对 $x$ 求导的结果为 $2x$。

假设 $x$ 是 $n$ 维列向量，$A$ 是 $n \times n$ 的矩阵，则 $x^TAx$ 是一个实数。我们要求的是 $\frac{\partial (x^TAx)}{\partial x}$。

将 $x$ 表示为列向量形式，即 $x=[x_1, x_2, \cdots, x_n]^T$，则 $x^TAx$ 可以表示为：

$$ x^TAx = \sum_{i=1}^n\sum_{j=1}^na_{ij}x_ix_j $$

对于第 $i$ 个分量 $x_i$，我们有：

$$ \frac{\partial (x^TAx)}{\partial x_i} = \sum_{j=1}^na_{ij}x_j + \sum_{j=1}^na_{ji}x_j = \sum_{j=1}^n(a_{ij}+a_{ji})x_j $$

因此，对于整个 $x^TAx$，我们有：

$$ \frac{\partial (x^TAx)}{\partial x} = [\frac{\partial (x^TAx)}{\partial x_1}, \frac{\partial (x^TAx)}{\partial x_2}, \cdots, \frac{\partial (x^TAx)}{\partial x_n}]^T = [(a_{11}+a_{11})x_1 + (a_{12}+a_{21})x_2 + \cdots + (a_{1n}+a_{n1})x_n, \cdots, (a_{n1}+a_{1n})x_1 + \cdots + (a_{nn}+a_{nn})x_n]^T = [(a_{11}+a_{11})x_1 + 2a_{12}x_2 + \cdots + 2a_{1n}x_n, 2a_{21}x_1 + (a_{22}+a_{22})x_2 + \cdots + 2a_{2n}x_n, \cdots, 2a_{n1}x_1 + 2a_{n2}x_2 + \cdots + (a_{nn}+a_{nn})x_n]^T = (A+A^T)x $$

因此，$x^TAx$ 对 $x$ 求导的结果为 $(A+A^T)x$。

---

假设 $x$ 是 $n\times m$ 的矩阵，$A$ 是 $n\times n$ 的矩阵，则 $xA$ 是 $n\times n$ 的矩阵。我们要求的是 $\frac{\partial (xA)}{\partial x}$。

将 $x$ 表示为矩阵的形式，即 $x=[x_{ij}]_{n\times m}$，则 $xA$ 可以表示为：

$$ xA = \begin{bmatrix} \sum_{k=1}^m x_{1k}a_{k1} & \sum_{k=1}^m x_{1k}a_{k2} & \cdots & \sum_{k=1}^m x_{1k}a_{kn} \ \sum_{k=1}^m x_{2k}a_{k1} & \sum_{k=1}^m x_{2k}a_{k2} & \cdots & \sum_{k=1}^m x_{2k}a_{kn} \ \vdots & \vdots & \ddots & \vdots \ \sum_{k=1}^m x_{nk}a_{k1} & \sum_{k=1}^m x_{nk}a_{k2} & \cdots & \sum_{k=1}^m x_{nk}a_{kn} \ \end{bmatrix} $$

对于第 $i$ 行第 $j$ 列的分量 $x_{ij}$，我们有：

$$ \frac{\partial (xA){ij}}{\partial x{kl}} = a_{lj}\delta_{ik} $$

其中 $\delta_{ik}$ 表示克罗内克（Kronecker）符号，当 $i=k$ 时取值为 $1$，否则为 $0$。

因此，对于整个 $xA$，我们有：

$$ \frac{\partial (xA)}{\partial x_{kl}} = \begin{bmatrix} a_{11}\delta_{1k} & a_{12}\delta_{1k} & \cdots & a_{1n}\delta_{1k} \ a_{21}\delta_{2k} & a_{22}\delta_{2k} & \cdots & a_{2n}\delta_{2k} \ \vdots & \vdots & \ddots & \vdots \ a_{n1}\delta_{nk} & a_{n2}\delta_{nk} & \cdots & a_{nn}\delta_{nk} \ \end{bmatrix} = [a_{1l}\delta_{1k}, a_{2l}\delta_{2k}, \cdots, a_{nl}\delta_{nk}]^T $$

因此，$\frac{\partial (xA)}{\partial x}$ 是一个 $n\times nm$ 的矩阵，其中第 $i$ 行由 $A$ 的第 $i$ 行展开成的 $1\times m$ 向量在第 $i$ 个 $m$ 块的位置上，其他位置为 $0$。

---

共轭梯度法可以用于求解形如 $Ax=b$ 的线性方程组，其中 $A$ 是对称正定矩阵。对于形如 $xA=b$ 的方程组，我们可以对其进行变形，得到 $A=x^Tx$ 和 $b=x^Tb$，然后再使用共轭梯度法求解。

具体来说，共轭梯度法的每一步包括以下几个步骤：

初始化：取初始值 $x_0$，初始残差 $r_0=b-Ax_0$，初始搜索方向 $p_0=r_0$，以及初始步长 $\alpha_0$。

搜索方向：根据上一步的搜索方向 $p_{k-1}$ 和残差 $r_{k-1}$，计算新的搜索方向 $p_k=r_{k-1}-\beta_{k-1}p_{k-1}$，其中 $\beta_{k-1}=\frac{r_{k-1}^Tr_{k-1}}{r_{k-2}^Tr_{k-2}}$。

步长计算：根据搜索方向 $p_k$，计算步长 $\alpha_k=\frac{r_{k-1}^Tr_{k-1}}{p_k^TAp_k}$。

更新结果：更新解 $x_k=x_{k-1}+\alpha_kp_k$，以及残差 $r_k=r_{k-1}-\alpha_kAp_k$。

判断终止条件：当残差的范数小于某个阈值时，停止迭代，输出结果 $x_k$。

对于 $xA=b$，我们可以将其变形为 $x^Tx A^{-1}x^Tx = x^Tb$，其中 $A^{-1}$ 是对称正定矩阵，因此可以使用共轭梯度法求解。在每次迭代时，需要计算 $A^{-1}x^Tx$ 和 $A^{-1}b$，可以使用 Cholesky 分解或者 LU 分解来求解 $A^{-1}$，然后再进行计算。

总之，共轭梯度法可以用于求解 $xA=b$ 的线性方程组，只需要将其变形为 $Ax=b$ 的形式，然后使用共轭梯度法进行求解即可。

---
对于形如 $xA=b$ 的线性方程组，我们可以使用矩阵分解或者迭代算法来求解。由于 $x$ 是 $n\times m$ 的矩阵，因此不能直接使用 QR 分解法进行求解。下面介绍一种基于 SVD 分解的方法。

假设 $A=U\Sigma V^T$ 是 $A$ 的 SVD 分解，其中 $U$ 和 $V$ 是正交矩阵，$\Sigma$ 是对角矩阵。则方程组 $xA=b$ 可以变形为 $(U\Sigma V^T)(x^T)=b$，令 $y=x^TV\Sigma^{-1}$，则原方程组变为 $Uy=b$。

由于 $U$ 是正交矩阵，因此可以使用回代法求解 $Uy=b$。具体来说，可以从最后一行开始，依次求解 $y_i=\frac{1}{u_{ii}}(b_i-\sum_{j=i+1}^n u_{ij}y_j)$，直到求解出 $y_1$。然后再计算 $x^T=V\Sigma^{-1}y$，即可求得 $x$。

需要注意的是，由于 SVD 分解的计算量较大，因此该方法适用于小规模问题，对于大规模问题，可能需要使用迭代算法来求解。

总之，对于形如 $xA=b$ 的线性方程组，可以使用 SVD 分解法进行求解。该方法适用于小规模问题，可以通过 SVD 分解来将问题转化为回代求解。

$$φ(ω) = -\arctan((b_1\sin(ω) + b_3\sin(ω)) / (b_1 \cos(ω) - b_3 \cos(ω)))$$
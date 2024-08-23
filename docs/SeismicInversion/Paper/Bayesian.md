# Bayesian seismic AVO inversion with a group optimization strategy
基于分组优化策略的贝叶斯地震AVO反演

Acta Geophysica (2023) 71:2733–2746 

https://doi.org/10.1007/s11600-023-01065-w

### 摘要

基于贝叶斯框架的地震AVO ( amplitude-versus-offset )反演是一种从地震观测数据中获取地层弹性参数的有效方法。通常，该算法通过一个适当的长尾先验概率分布来获得弹性参数的稀疏反射率水平。考虑到地下参数之间的高度相关性，采用分组优化策略进行贝叶斯AVO反演，即在反射界面处对模型参数进行分组反演。设计由柯西约束和l2范数组成的组柯西约束。在约束中，组间执行柯西约束以促进模型参数的稀疏性，每组内执行l2正则化以约束均匀性。新的基于组优化策略的贝叶斯AVO反演算法可以帮助估计具有期望稀疏度的弹性参数的反射率水平。此外，即使在没有协方差矩阵的情况下，该算法仍然可以保证所得解的相关性。从数值和现场实例都证明了该方法取得了良好的效果。

### 引言

贝叶斯反演将模型参数的先验概率分布与目标函数相结合，有助于增强解的稳健性,在贝叶斯地震AVO反演中，通常采用合适的长尾先验概率分布来获得稀疏分布的弹性参数反射率水平. 实证研究表明，地下参数之间存在较高的相关性. 

贝叶斯AVO反演最常见的做法是假设一个包含协方差矩阵的先验概率分布，可以包含AVO参数之间的预估计相关信息, 然而，根据现有的测井数据，协方差矩阵通常是不可用或难以估计的, 我们在本研究的贝叶斯地震AVO反演中使用了组优化策略。在此基础上，提出了一种新的基于组Cauchy约束的贝叶斯AVO反演算法。通过在三项AVO反演中设计群柯西约束，我们可以获得具有所需稀疏度的弹性参数的估计反射率水平。此外，我们可以确保即使不使用协方差矩阵，得到的解也是相关的。新的反演算法在数值和实际算例中都取得了较好的结果。

### 理论

#### 正演模型

使用Aki-Richards方程

$$
\begin{array}{ll}
R_P(\theta)=a \frac{\Delta V_P}{2 V_P}+b \frac{\Delta V_S}{2 V_S}+c \frac{\Delta \rho}{2 \rho}, & \text { where }: \\
a=\frac{1}{\cos ^2 \theta}, & \rho=\frac{\rho_2+\rho_1}{2}, \Delta \rho=\rho_2-\rho_1, \\
b=-8 K \sin ^2 \theta, & V_P=\frac{V_{P 2}+V_{P 1}}{2}, \Delta V_P=V_{P 2}-V_{P 1}, \\
c=1-4 K \sin ^2 \theta, & V_S=\frac{V_{S 2}+V_{S 1}}{2}, \Delta V_S=V_{S 2}-V_{S 1}, \\
K=\left(\frac{V_S}{V_P}\right)^2, & \text { and } \theta=\frac{\theta_1+\theta_2}{2} .
\end{array}
$$

#### 目标函数

根据贝叶斯推断，基于似然函数和r的先验概率分布，可以计算出未知模型参数r的后验概率分布

$$p(r|d)=\frac{p(d|r) * p(r)}{p(d)}\propto p(d|r) * p(r)\tag{1}$$

一般来说，只有PDF的形状是令人感兴趣的，所有正规化函数p(d)可以忽略
一般情况下，似然函数p(d|r)设定为服从零均值、标准方差为σ的高斯分布的噪声概率分布：

$$p(d|r)=\frac{1}{\sqrt{2\pi}\sigma}\exp\left[\frac{-(d-Gr)^T(d-Gr)}{2\sigma^2}\right]\tag{2}$$

对于r的先验概率分布，通常选择柯西分布来提供模型参数r的稀疏解。柯西分布的概率密度为：

$$p(r)=\frac{1}{(\pi\sigma_r)^{3m}}\prod_{i=1}^{3m}\left[\frac{1}{1+\frac{r_i^2}{\sigma_r^2}}\right]  \tag{3}$$

后验概率最大的模型参数r成为AVO反演的最优解,将(2)(3)式代入(1)式,两边取对数,加负号得到

$$-\ln p(r|d) \propto \sum_i^{3m}\ln\left(1+\frac{r_i^2}{\sigma_r^2}\right)+\frac{1}{2\sigma^2}[(d-Gr)^T(d-Gr)]$$

将问题变换为最小值问题,AVO反演的目标函数可以表示右侧项.

**通常，具有有效低频成分的先验模型可以作为正则化项，对原始叠前地震数据中低频成分的缺失做出补偿**

P波速度VP(t)与反射系数RP(t)之间的关系可近似表示为如下形式

$$R_p(t) = \frac{\Delta V_P}{2 V_P}\approx \frac{1}{2}\frac{\partial}{\partial t}\ln V_p(t)$$

两边同时积分得到

$$\frac{1}{2}\ln\frac{V_p(t)}{V_p(t_0)} = \int_{t_0}^tR_p(\tau)d(\tau) $$

矩阵形式为:

$$
\left[\begin{array}{c}
\frac{1}{2} \ln \frac{V_P\left(t_1\right)}{V_P\left(t_0\right)} \\
\frac{1}{2} \ln \frac{V_P\left(t_2\right)}{V_P\left(t_0\right)} \\
\vdots \\
\frac{1}{2} \ln \frac{V_P\left(t_m\right)}{V_P\left(t_0\right)}
\end{array}\right]=
\left[\begin{array}{cccc}
1 & & & \\
1 & 1 & & \\
\vdots & \vdots & \ddots & \\
1 & 1 & \cdots & 1
\end{array}\right]\left[\begin{array}{c}
R_P\left(t_1\right) \\
R_P\left(t_2\right) \\
\vdots \\
R_P\left(t_m\right)
\end{array}\right]
$$

令$\xi_p$为左式, $C_p$为积分矩阵(由于R有vp,vs,rho三项,故积分矩阵也为三项,在除了对应的位置为积分矩阵其他两项均为0), 所以

$$\xi_p = C_p \cdot r_p$$

加入到目标函数中得到

$$f(r) = [(d-Gr)^T(d-Gr)]+2\sigma^2\sum_i^{3m}\ln\left(1+\frac{r_i^2}{\sigma_r^2}\right)+\lambda_P(\xi_p - C_p r_p)^TP(\xi_p - C_p r_p)+\lambda_S(\xi_s - C_s r_s)^T(\xi_s - C_s r_s)+\lambda_\rho(\xi_\rho - C_\rho r_\rho)^T(\xi_\rho - C_\rho r_\rho)$$

其中$\lambda$为补偿系数

可将目标函数进行合并得到

$$f(r) = (\tilde d-\tilde Gr)^T(\tilde d-\tilde Gr)+2\sigma^2\sum_i^{3m}\ln\left(1+\frac{r_i^2}{\sigma_r^2}\right)$$

其中 

$$  \tilde d = [d, \sqrt{\lambda_P}\cdot\xi_P, \sqrt{\lambda_S}\cdot\xi_S, \sqrt{\lambda_\rho}\cdot\xi_\rho]^T \\
 \tilde G = [G, \sqrt{\lambda_P}\cdot C_P, \sqrt{\lambda_S}\cdot C_S, \sqrt{\lambda_\rho}\cdot C_\rho]^T$$

 方程右边的两个项可以分别看作数据匹配项和Cauchy约束项, 详细可参考原论文

 #### 分组反演策略

RP、RS和Rρ之间的相关信息被排除在目标函数之外。因此，基于现有测井数据估计的协方差矩阵通常包含在三项AVO反演中。模型参数的协方差矩阵可写成如下

$$
C_r = 
\left[\begin{matrix}
    \sigma_{R_p}^2 & \sigma_{R_pR_s} & \sigma_{R_sR_\rho} \\
    \sigma_{R_pR_s} & \sigma_{R_s}^2 & \sigma_{R_sR_\rho} \\
    \sigma_{R_pR_\rho} & \sigma_{R_sR_\rho} & \sigma_{R_\rho}^2
\end{matrix} \right]
$$

协方差矩阵的对角线元素分别为反射率水平RP、RS和Rρ的方差。非对角线元素描述了RP、RS和R ρ之间的相互关系。

通过特征值分解可得

$$C_r = V \Sigma V^T$$

特征向量V用于转换模型参数,通过$r' = V^{-1}r$转换使其独立

则目标函数变为

$$f(r) = (\tilde d-\tilde G'r')^T(\tilde d-\tilde G'r')+2\sigma^2\sum_i^{3m}\ln\left(1+\frac{r_i'^2}{\sigma_{r'}^2}\right)$$

其中$\tilde G' = \tilde GV$

协方差矩阵能够解决三项AVO反演中模型参数之间的相关性问题。协方差矩阵通常是不可用或相当难估计的,我们可以采用**分组优化策略**反演相关模型参数





$$\begin{aligned}
p(\mathbf{d}|\mathbf{m})& =\prod_{i=1}^{N_1}p(\mathbf{d}_i|\mathbf{m}_i)  \\
&\propto\exp\left(-\frac12\sum_{i\operatorname{=}1}^{N_1}(\mathbf{d}_i-\mathbf{G}_i\mathbf{m}_i)^T\mathbf{\Sigma}_{d,i}^{-1}(\mathbf{d}_i-\mathbf{G}_i\mathbf{m}_i)\right)
\end{aligned}$$
## 凸优化概括

[变量选择之LASSO—（一）凸正则化方法1](https://zhuanlan.zhihu.com/p/596857609)

## least-square
参考: 

https://textbooks.math.gatech.edu/ila/least-squares.html

https://zhuanlan.zhihu.com/p/31341436

https://en.wikipedia.org/wiki/Ordinary_least_squares


对于方程

$$Ax=b$$

最小二乘法的解为

$$\hat x=(A^TA)^{-1}A^Tb$$

不适用于不满秩的情况。

## Tikhonov regularization/Ridge Regression(RR)
参考： 

https://en.wikipedia.org/wiki/Ridge_regression

也称**岭回归**，岭回归是在线性回归模型中存在多共线性(高度相关)自变量时，作为解决最小二乘估计量不精确问题的一种可能方法.

通过在对角线上添加正元素来缓解，从而减少其条件数。类似于普通的最小二乘估计器，$\lambda$为正则化阻尼Regularization dampings  

$${\displaystyle {\hat {\beta }}_{R}=(\mathbf {X} ^{\mathsf {T}}\mathbf {X} +\lambda \mathbf {I} )^{-1}\mathbf {X} ^{\mathsf {T}}\mathbf {y} }$$

在最小二乘最小化过程中加入正则化项，防止过拟合，获得更加理想的解，类似于L2正则化。

$${\displaystyle \|A\mathbf {x} -\mathbf {b} \|_{2}^{2}+\|\Gamma \mathbf {x} \|_{2}^{2}}$$

解为

$${\displaystyle {\hat {x}}=(A^{\top }A+\Gamma ^{\top }\Gamma )^{-1}A^{\top }\mathbf {b} .}$$

Tikhonov matrix $\Gamma = \alpha I$

### Generalized Tikhonov regularization

$${\displaystyle x^{*}=(A^{\top }PA+Q)^{-1}(A^{\top }Pb+Qx_{0})}$$

其中P为b的协方差矩阵，$x_0$为$x$的期望值

### Lavrentyev regularization

如果A是对称正定的，即有$A=A^{\top }>0$， 所以${\displaystyle \|x\|_{P}^{2}=x^{\top }A^{-1}x}$ ????

故最优解为

$${\displaystyle x^{*}=(A+Q)^{-1}(b+Qx_{0})}$$

可以看作Tikhonov regularization中${\displaystyle A=A^{\top }=P^{-1}}$的情况

## Bregman method
参考：

[如何理解Bregman divergence?](https://www.zhihu.com/question/22426561/answer/209945856)

[Bregman 散度（Bregman divergence）和Bregman信息（Bregman information）](https://blog.csdn.net/wangshun_410/article/details/84963242?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166911133516800182726241%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166911133516800182726241&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-84963242-null-null.142^v66^control,201^v3^add_ask,213^v2^t3_control1&utm_term=bregman&spm=1018.2226.3001.4187)

[机器学习中的数学——距离定义（二十五）：布雷格曼散度（Bregman Divergence）](https://machinelearning.blog.csdn.net/article/details/122395719?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-122395719-blog-84963242.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-122395719-blog-84963242.pc_relevant_default&utm_relevant_index=2)

[Split-Bregman迭代方式](https://blog.csdn.net/weixin_41923961/article/details/80329393)

定义Bregman散度为

$$D_{F}(p,q)=F(p)-F(q)-\langle \nabla F(q),p-q\rangle$$

几何意义如下图
![](https://www.researchgate.net/publication/284899225/figure/fig1/AS:401374723493888@1472706599282/Geometric-interpretation-of-Bregman-Divergence.png)

选择不同的F会有不同的损失函数

![](https://picx.zhimg.com/v2-cbbe99689224cdd829003483938af50e_r.jpg?source=1940ef5c)

例如最小均方误差

$$d^2(x,y)=\|x-y\|^2= \left< x-y,x-y \right> = \|x\| ^2 - \|y\|^2-\left<2y,x-y \right>$$

### Bregman Iteration

$$u_{k+1}=\argmin_uD^{p_k}_J(u,u_k)+\lambda\pmb H(u), p_k\in\partial J(u_k)$$

等价于

$$u_{k+1}=\arg \min _{u} J(u)-\left\langle p, u-u_{k}\right\rangle+\lambda H(u), \quad p_{k} \in \partial J\left(u_{k}\right)$$

Result，证明见文档[《Notes on Bregman Iteration》](https://getreuer.info/posts/bregman.pdf)

$$\pmb H(u_{k+1})\leq \pmb H(u_{k+1})+\frac{1}{\lambda}D^{p_k}_J(u_{k+1},u_k)\leq\pmb H(u_k)$$

$$
\left\{\begin{array}{l}
u_{k+1}=\argmin_uD_{J}^{p_{k}}\left(u, u_{k}\right)+\lambda H(u) \\
p_{k+1}=p_{k}-\lambda \nabla H\left(u_{k+1}\right)
\end{array}\right.
$$

假设$H(u,f)=\frac12||Au-f||^2_2$，则$p_0=0$时的迭代过程等价于
$$
\left\{ \begin{array}{l}
u_{k+1}=\arg \min _{u} J(u)+\lambda H\left(u, f_{k}\right) \\
f_{k+1}=f_{k}+\left(f-A u_{k+1}\right), \quad f_{0}=f
\end{array}\right.
$$

### Linearized Bregman
假设$J(u)=||u||_1$，将H(u)进行矩阵泰勒展开得

$$H(u)\approx H(u_k)+\nabla H(u_k)\cdot(u_k-u)$$

但是这种近似展开只有在$u$和$u_k$十分接近的时候上式才准确，所以加入惩罚项$\frac{1}{2 \delta}\left\|u-u_{k}\right\|_{2}^{2}$，最终如下

$$u_{k+1}=\arg \min _{u} D^{p_k}_J(u,u_k)+\lambda H(u_k)+\left\langle\lambda \nabla H\left(u_{k}\right), u-u_k\right\rangle+\frac{1}{2 \delta}\left\|u-u_{k}\right\|_{2}^{2}$$

化简得
$$
u_{k+1}=\arg \min _{u} J(u)+\left\langle\lambda \nabla H\left(u_{k}\right)-p_{k}, u\right\rangle+\frac{1}{2 \delta}\left\|u-u_{k}\right\|_{2}^{2}
$$

#### Split Bregman
假设$\Phi 和 E(u)$均为凸函数

$$\min _{u}\|\Phi(u)\|_{1}+E(u)$$

通过运算符分裂

$$\min _{u, d}\|d\|_{1}+E(u) \text { subject to } \Phi(u)=d$$

令$J(u,d)=||d||_1+E(u),H(u,d)=\frac12||d-\Phi(u)||^2_2$，带入Bregman方法

$$\left\{\begin{aligned}
\left(u^{k+1}, d^{k+1}\right) &=\min _{u, d} J(u, d)-\left\langle p_{u}^{k}, u-u^{k}\right\rangle-\left\langle p_{d}^{k}, d-d^{k}\right\rangle+\lambda H(u, d) \\
p_{u}^{k+1} &=p_{u}^{k}-\lambda \nabla_{u} H\left(u^{k+1}, d^{k+1}\right) \\
p_{d}^{k+1} &=p_{d}^{k}-\lambda \nabla_{d} H\left(u^{k+1}, d^{k+1}\right)
\end{aligned}\right.$$

lsqr方法

[最小二乘法原理详解](https://zhuanlan.zhihu.com/p/76644659)

[最小平方QR分解法](https://wenku.baidu.com/view/86ca6a43a8956bec0975e330?aggId=86ca6a43a8956bec0975e330&fr=catalogMain_&_wkts_=1676943404187)

$$\min\left\|\left[\begin{array}{c}
A \\
\lambda I
\end{array}\right] x-\left[\begin{array}{l}
b \\
0
\end{array}\right]\right\|_2$$

## 基础方法即代码实现

对于问题

$$\min\left\|\left[\begin{array}{c}
A \\
\lambda I
\end{array}\right] x-\left[\begin{array}{l}
b \\
0
\end{array}\right]\right\|_2$$

在之后的代码实现中，令$G = \left[\begin{array}{c}A \\ \lambda I \end{array}\right],
obs = \left[\begin{array}{c} b \\ 0 \end{array}\right]$

其中$\lambda I$为正则化项，用于为反问题增加约束，减小多解性

### 梯度下降

通过梯度下降最小化目标泛函$f(m)$的迭代流程为

$$
m^{k+1} = m^k - \alpha_k \nabla f(m^k)
$$

对于$Ax=b$的问题，可以求出

$$\min f(m) = ||Gm-d||^2_2$$

$$f(m) = (Gm-d)^T(Gm-d)=m^TG^TGm-2(G^Td)^Tm+d^Td$$

$$\nabla f(m)=2G^TGm-2G^Td$$

$$m^{k+1} = m^k - \alpha_k (2G^TGm^k-2G^Td) $$

步长的选取

$$
\alpha_k \leq \frac{2}{\lambda_{max} (G^TG)}
$$

代码实现

```python
def GD_run(self, init_guess, G, obs, Iter, show=False):
    pre = init_guess
    loss = []
    for k in range(Iter):
        step = 1
        grad = G.T @ G @ pre - G.T @ obs
        residual = self.calculate_residual(pre, G, obs)
        rmse = np.linalg.norm(residual)
        # 原定损失小于沿学习率迭代后的损失，说明步长较大，减小步长
        while rmse < np.linalg.norm(self.calculate_residual(pre - step * grad, G, obs)):
            step = step * 0.5
        pre = pre - step * grad
        loss.append(rmse)

    if show:
        plt.figure()
        plt.plot(loss, linewidth=1, linestyle="solid", label="loss")
        plt.legend()
        plt.title('GD Loss curve(final loss = ' + str(round(loss[-1], 4)) + ')')
        plt.xlabel('iters')
        plt.ylabel('res')

    return pre, loss
```

### 随机梯度下降

类似梯度下降，只不过不是求全部列的梯度，而是求其随机一列的梯度进行下降

$$
\min f(m) = ||Gm-d||^2_2 = \sum^m_{i=1} f_i(m) = \sum^m_{i=1}(Gm-d)^2_i \\
\nabla f(m)_i = 2G^T_iG_im-2d_iG^T_i
$$

代码实现(固定梯度) TODO 变梯度

```python
def SGD_run(self, init_guess, G, obs, iter, show=False):
    self.init_guess = init_guess
    pre = self.init_guess
    rmse_prev = np.inf
    loss = []
    for k in range(iter):
        step = 1  # 步长 
        lossres = self.calculate_residual(pre, self.A, self.b)  # 损失，如果未加正则化,self.A,self.b即为G,obs
        index = round(np.random.random() * (G.shape[0] - 1))  # 随机取行号
        grad = G[index:index + 1, :].T @ G[index:index + 1, :] @ pre - obs[index] * G[index, :].T  # 随机一行的梯度
        pre = pre - step * grad  # 梯度下降
        rmse = np.linalg.norm(lossres)
        loss.append(rmse)

    if show:
        plt.figure()
        plt.plot(loss, linewidth=1, linestyle="solid", label="loss")
        plt.legend()
        plt.title('SGD Loss curve(final loss = ' + str(round(loss[-1], 4)) + ')')
        plt.xlabel('iters')
        plt.ylabel('res')

    return pre, loss
```

### 高斯牛顿法

参考链接: 

[Algorithms from scratch: Gauss-Newton](https://omyllymaki.medium.com/gauss-newton-algorithm-implementation-from-scratch-55ebe56aac2e)

[最小二乘问题的四种解法——牛顿法，梯度下降法，高斯牛顿法和列文伯格-马夸特法的区别和联系](https://zhuanlan.zhihu.com/p/113946848)

对于最小二乘问题

$$\min_x\frac{1}{2}||f(x)||_2^2$$

对$f(x + \Delta x)$进行泰勒展开得到

$$f(x+\Delta x) \approx f(x) + J(x)^T \Delta x + o(\Delta x)$$

所以最小二乘问题化为

$$\Delta x^* = \arg \min \frac{1}{2} ||f(x + \Delta x)||_2^2 \approx \arg \min \frac{1}{2} ||f(x) + J(x)^T \Delta x||_2^2$$

化简

$$m(x) = \frac{1}{2} ||f(x) + J(x)^T \Delta x||^2 = \frac{1}{2}(f(x) + J(x)^T \Delta x)^T(f(x) + J(x)^T \Delta x) $$

求导令导数等于零可得

$$m^`(x) = 0 \quad \rightarrow \quad J(x)J(x)^T\Delta x = - J(x)f(x) \rightarrow \Delta x = \frac{- J(x)f(x)}{J(x)J(x)^T}$$

代码实现

```python
def GN_run(self, init_guess, G, obs, iter, show=False):
    pre = init_guess
    loss = []
    for k in range(iter):
        residual = self.calculate_residual(pre, G, obs)
        lossres = self.calculate_residual(pre, self.A, self.b)
        jacobian = self.calculate_jacobian(pre, G, obs, step=10 ** (-6))
        pre = pre - np.linalg.pinv(jacobian.T @ jacobian) @ (jacobian.T @ residual)
        rmse = np.linalg.norm(lossres)
        loss.append(rmse)
    if show:
        plt.figure()
        plt.plot(loss, linewidth=1, linestyle="solid", label="loss")
        plt.legend()
        plt.title('Gauss-Newton Loss curve(final loss = ' + str(round(loss[-1], 4)) + ')')
        plt.xlabel('iters')
        plt.ylabel('res')

    return pre, loss
```

### 列文伯格-马夸特方法(Levenberg–Marquardt)

基本原理与高斯牛顿法类似，在高斯牛顿法中，可以看到，损失曲线在后期会进入锯齿状，迭代的次数较长。所以，为了避免其步长过大导致的问题，该方法提出了信赖区域，动态调整步长减小迭代次数

在迭代过程中设定一个评判标准用于评判估计的好坏，从而动态调整步长

$$\rho = \frac{f(x+\Delta x) - f(x)}{J(x)^T\Delta x}$$

分为以下三种情况

- $\rho$ 接近1，近似是好的，不需要更改
- $\rho$ 太小，则实际减少的值小于近似减少的值，近似较大，需要缩小近似的范围
- $\rho$ 太大，则实际减少的值大于近似减少的值，近似较小，需要扩大近似的范围。

反问题为

$$\min \frac {1}{2}||f(x) + J(x)^T \Delta x||  \quad s.t \quad ||D \Delta x < \mu||_2$$

其中$D$为系数矩阵，$\mu$为信赖半径

可以构建拉格朗日函数，$\lambda$为系数因子

$$L(\Delta x,\lambda) = \frac {1}{2}||f(x) + J(x)^T\Delta x||^2 + \frac {\lambda}{2}(||D \Delta x||^2 - \mu)$$

求导并化简得

$$J(x)f(x) + J(x)J^T(x)\Delta x + \lambda D^TD \Delta x = 0 \rightarrow (JJ^T + \lambda D^TD) \Delta x = -Jf$$

在实际使用中，通常将$I$代替$D^TD$

代码实现

```python
def LM_run(self, init_guess, G, obs, iter, show=False):
    pre = init_guess
    lam = 0.01  # todo 如何设定lambda
    loss = []
    for k in range(iter):
        residual = self.calculate_residual(pre, G, obs)
        lossres = self.calculate_residual(pre, self.A, self.b)
        jacobian = self.calculate_jacobian(pre, G, obs, step=10 ** (-6))
        pre = pre - np.linalg.pinv(jacobian.T @ jacobian + lam * np.identity(jacobian.shape[1])) @ (
                jacobian.T @ residual)
        rmse = np.linalg.norm(lossres)
        loss.append(rmse)
    if show:
        plt.figure()
        plt.plot(loss, linewidth=1, linestyle="solid", label="loss")
        plt.legend()
        plt.title('Levenberg-Marquardt Loss curve(final loss = ' + str(round(loss[-1], 4)) + ')')
        plt.xlabel('iters')
        plt.ylabel('res')

    return pre, loss
```

LM方法是由牛顿法的基础上得到的，他们之间的联系主要取决于$\lambda$的取值

对于迭代过程

$$
(JJ^T + \lambda I) \Delta x_k = -Jf
$$

- 当$\lambda$较大时，更新方程中$\lambda I$占主要，迭代近似为$\lambda I \Delta x_k = -Jf$，此时为梯度下降法
- 当$\lambda$较大时，更新方程中$JJ^T$占主要，迭代近似为$JJ^T \Delta x_k = -Jf$，此时为高斯牛顿法

### IRLS
#### 正则化反演
参考链接

[变量选择之LASSO—（一）凸正则化方法1](https://zhuanlan.zhihu.com/p/596857609)


对于最小二乘问题$\min ||Ax-b||_2$进行反演时，因为约束条件不够，可能会出现多个不同的$x$满足$\min ||Ax-b||_2$，当并不是我们预期的$x$,也就是问题的多解性，这时我们需要在求解反问题的方程中添加其他约束项，从数学上来讲，就是在损失函数中加个正则项，来防止过拟合。目前主要有L1正则化和L2正则化，即LASSO回归和岭回归

参考链接：[l1正则与l2正则的特点是什么，各有什么优势？](https://www.zhihu.com/question/26485586/answer/91420865)

- L1-Regularization(一范数)
  $$\lambda \sum_{i=0}^k|w_i| $$

- L2-Regularization(二范数)
  $$\lambda \sum_{i=0}^k w_i^2$$

对于地震反演，常用的就是TV正则化，即对模型的差分的一范数，提高模型的不连续性，可以形成稀疏解

$$
\min ||Gm-d||_2^2+\alpha||Lm||_1
$$

其中$L$为差分矩阵

这里介绍一种求解带有一范数正则化反问题的求解方法 **IRLS**

对于上式求梯度得到

$$
\nabla f(m) = 2G^TGm-2G^Td+\alpha \sum_{i=0}^m\nabla|y_i|
$$

由于$y_i$不为零，故

$$
\frac{\partial|y_i|}{\partial m_k} = L_{i,k} \mathrm{sgn}(y_i)
$$

因为$|Lm| = \sum_{i=1}^m |y_i|$，故

$$
\frac{\partial|Lm|}{\partial m_k} = \sum_{i=1}^mL_{i,k} \frac{y_i}{|y_i|}
$$

所以我们令$\mathbf{W}$表示一个对角阵，且

$$W_{i,i}=\frac{1}{|y_i|}$$

则$\nabla|Lm|=L^TWLm$

梯度变为

$$
\nabla f(m) = 2G^TGm-2G^Td+\alpha L^TWLm
$$

因为W依赖于Lm，所以这是一个非线性方程组，且W在任意Lm取值为0的点无定义

为解决不可微问题，我们设定一个容许偏差$\epsilon$，并将W改写如下

令
$$
W_{i, i}= \begin{cases}1 /\left|y_i^{(k)}\right| & \left|y_i^{(k)}\right| \geq \epsilon \\
1 / \epsilon & \left|y_i^{(k)}\right|<\epsilon .\end{cases}
$$

迭代过程为

$$
\mathbf{m}^{(k+1)}=\arg \min \left\|\left[\begin{array}{c}
\mathbf{G} \\
\sqrt{\frac{\alpha}{2}} \sqrt{\mathbf{W}} \mathbf{L}
\end{array}\right] \mathbf{m}-\left[\begin{array}{l}
\mathbf{d} \\
\mathbf{0}
\end{array}\right]\right\|_2
$$

代码实现，示例为高斯牛顿法优化策略

```python
def GN_run(self, init_guess, G, obs, iter, show=False, IRLS=None):  # IRLS为一范数参数[alpha, eps, L]
    pre = init_guess
    loss = []
    with tqdm(total=iter) as t:
        for i in range(iter):
            t.set_description('solver GN')
            if IRLS is not None:
                L = IRLS[-1]
                alpha = IRLS[0]
                eps = IRLS[1]
                Reg = self._IRLSupdate(L,pre,alpha,eps)
                G = np.vstack((self.A, Reg))
                obs = np.hstack((self.b, np.zeros((Reg.shape[0]))))  # TODO 一维数据适用，二维数据要改！！！
            residual = self.calculate_residual(pre, G, obs)
            lossres = self.calculate_residual(pre, self.A, self.b)
            jacobian = self.calculate_jacobian(pre, G, obs, step=10 ** (-6))
            pre = pre - np.linalg.pinv(jacobian.T @ jacobian) @ (jacobian.T @ residual)
            rmse = np.linalg.norm(lossres)
            loss.append(rmse)
            t.set_postfix(loss=rmse)
            t.update(1)
    if show:
        plt.figure()
        plt.plot(loss, linewidth=1, linestyle="solid", label="loss")
        plt.legend()
        plt.title('Gauss-Newton Loss curve(final loss = ' + str(round(loss[-1], 4)) + ')')
        plt.xlabel('iters')
        plt.ylabel('res')

    return pre, loss
```

### ISTA方法
参考链接

[软阈值迭代算法（ISTA算法）](https://zhuanlan.zhihu.com/p/555651497)

对于反问题

$$\min ||Ax-b||_2^2+\lambda||x||_1$$

通过推导得到（具体参照上面的参考链接）

$$
\mathbf{x}_{k+1}=\mathcal{T}_{\lambda t}\left(\mathbf{x}_k-2 t \mathbf{A}^T\left(\mathbf{A} \mathbf{x}_k-\mathbf{b}\right)\right)
$$

其中，软阈值函数为: 

参考链接：[硬阈值 & 软阈值](https://blog.csdn.net/qq_40206371/article/details/122422976)

$$\mathcal{T}_\omega(\mathbf{x})_i=\left(\left|x_i\right|-\omega\right)_{+} \operatorname{sgn}\left(x_i\right)$$

即

$$
[\mathcal{T}_{\omega}(x)]_i=\begin{cases} \begin{array}{c} x_i-\omega ,\quad x_i>\omega\\\end{array}\\ \begin{array}{c} 0,\qquad \quad |x_i|\le \omega\\\end{array}\\ \begin{array}{c} x_i+\omega ,\quad x_i<-\omega\\\end{array}\\\end{cases} \\
$$

当A为单位矩阵时，即

$$\min ||x-b||_2^2+\lambda||x||_1$$

解为

$$x^*=\mathcal{T}_{\frac{\lambda}{2}}(b)$$

代码实现

```python
def Ista_run(self, init_guess, G, obs, iter, show=False):
    pre = init_guess
    lamb = 0.01
    tk = 1 / np.linalg.norm(G.T @ G)  # tk最大值

    loss = []
    for k in range(iter):
        residual = self.calculate_residual(pre, G, obs)
        rmse = np.linalg.norm(residual)
        loss.append(rmse)
        grad = G.T @ G @ pre - G.T @ obs
        grad = np.squeeze(grad)
        zk = pre - tk * grad
        pre = self._softThresh(lamb * tk, zk)
        tk = tk / 1.1
    if show:
        plt.figure()
        plt.plot(loss, linewidth=1, linestyle="solid", label="loss")
        plt.legend()
        plt.title('ISTA Loss curve')
        plt.xlabel('iters')
        plt.ylabel('res')
    return pre, loss
```

### FISTA方法
Amir Beck等人将Nesterov加速算法引入ISTA算法中，并称之为FISTA算法，其将复杂度从$O(1/k)$降低到了$O(1/k^2)$，减小了ISTA的计算复杂度，提高了计算速度，使损失可以得到更快收敛

ISTA方法是直接迭代$x_k$，FISTA是迭代$y_k$，$y_k$由$x_k$计算得到


$$\mathbf y_{k+1}=\mathbf x_k+(\frac{t_k-1}{t_{k+1}})(\mathbf x_k-\mathbf x_{k-1})$$

其中

$$t_{k+1}=\frac{1+\sqrt{1+4t^2_k}}{2}$$


```python
def Fista_run(self, init_guess, G, obs, iter, show=False):
    pre = init_guess
    tk = 1
    t = 1 / np.linalg.norm(G.T @ G)
    lamb = 0.01
    loss = []
    xk = init_guess
    xk_1 = xk
    for k in range(iter):
        tk1 = 0.5 * (1 + np.sqrt(1 + 4 * tk ** 2))
        yk1 = xk + (tk - 1) / tk1 * (xk - xk_1)
        grad = G.T @ G @ yk1 - G.T @ obs
        grad = np.squeeze(grad)
        zk = yk1 - t * grad

        # update iterative
        xk_1 = xk
        xk = self._softThresh(lamb * t, zk)

        tk = tk1
        # err = np.linalg.norm(self.pre - yk1)
        # if k!=0 and err<0.0001:
        #     print('第%d轮已经收敛，增量为%f' %(k,err))
        #     break
        pre = yk1
        t = t / 1.1

        # loss
        residual = self.calculate_residual(pre, G, obs)
        rmse = np.linalg.norm(residual)
        loss.append(rmse)

    if show:
        plt.figure()
        plt.plot(loss, linewidth=1, linestyle="solid", label="loss")
        plt.legend()
        plt.title('FISTA Loss curve')
        plt.xlabel('iters')
        plt.ylabel('res')
    return pre, loss
```

### Split-Bregman方法
参考见《反问题的迭代求解算法》课件

**对于一阶TV反问题**

$$\min _{m}||data-Gm||_2^2+\alpha||L m ||_1$$

令$d^k=Lm$，增加约束项，得到

$$\left(m^{k+1}, d^{k+1}\right)=\min _{m, d}||data-Gm||_2^2+\alpha||d^k||_1+\frac{\lambda}{2}\left\|d^k-Lm-b^{k}\right\|_{2}^{2} \\
b^{k+1} = b^k + (data-Gm-d^{k+1})$$


故，上式可拆解成两个子问题，岭回归问题与LASSO回归问题

子问题一(岭回归)

$$m^{k+1} = \min_m||data-Gm||_2^2+\frac{\lambda }{2} ||d^k-b^k-Lm||_2^2$$

可看作

$$\min\left|\left|\begin{bmatrix}G
 \\ \sqrt[]{\frac{\lambda}{2}} L
\end{bmatrix}m - 
\begin{bmatrix}data
 \\\sqrt[]{\frac{\lambda}{2}}(d-b)
\end{bmatrix}\right|\right|_2$$

子问题二(LASSO回归)

$$d^{k+1}=\min_d\frac{\lambda}{2}||d^k-b^k-Lm||_2^2+\alpha||d^k||_1$$

用软阈值函数可求解

$$
d^*=\begin{cases} \begin{array}{c} b+Lm-\frac{\lambda }{4}  ,\quad b+Lm>\frac{\lambda }{4}\\\end{array}\\ \begin{array}{c} 0,\qquad \qquad \qquad b+Lm\le \frac{\lambda }{4}\\\end{array}\\ \begin{array}{c} b+Lm+\frac{\lambda }{4} ,\quad b+Lm<-\frac{\lambda }{4}\\\end{array}\\\end{cases} \\
$$

$$
d^*=\begin{cases} \begin{array}{c} b+Lm-\frac{2\alpha }{\lambda}  ,\quad b+Lm>\frac{2\alpha}{\lambda}\\\end{array}\\ \begin{array}{c} 0,\qquad \qquad \qquad |b+Lm|\le \frac{2\alpha}{\lambda}\\\end{array}\\ \begin{array}{c} b+Lm+\frac{2\alpha}{\lambda} ,\quad b+Lm<-\frac{2\alpha}{\lambda}\\\end{array}\\\end{cases} \\
$$

代码实现

```python
def run(self, show=True):
    iiter = 0
    flag = 1
    loss = []
    lamb = 0.01
    x = self.x0
    for _ in tqdm(range(self.Iter)):
        # regularized problem
        dataregs = self.d - self.b_
        # subquestion1
        Sub1 = RegularizedInv(self.Op, self.obs, self.x0, lamb, self.Reg, dataregs)  # 初始化岭回归反演
        x, sub1loss = Sub1.run(self.x0 if flag else x, 10, 'GN', True)  # 选择优化方法迭代求解
        flag = 0
        # subquestion2
        dataregs2 = self.Reg @ x + self.b_
        # self.d, sub2loss = self.Ista_run(self.d,np.eye(self.Reg.shape[0]),dataregs2,50,None) # ISTA方法
        self.d = self._softThresh(lamb / 4, dataregs2)  # TODO lambda 应该怎么确定

        residual = np.linalg.norm(self.calculate_residual(x, self.A, self.b))
        loss.append(residual)

        self.b_ = self.b_ + self.tau * (self.Reg @ x - self.d)
        iiter = iiter + 1

    if show:
        plt.figure()
        plt.plot(loss, linewidth=1, linestyle="solid", label="loss")
        plt.legend()
        plt.title('Split Bregman Loss curve(final loss = ' + str(round(loss[-1], 4)) + ')')
        plt.xlabel('iters')
        plt.ylabel('res')

    return x, loss
```

**对于二阶TV反问题**

$$
\min_m\sum_i\sqrt{(\nabla_x m)^2_i+(\nabla_y m)^2_i}+\frac{\mu}{2}||Gm-d||^2_2
$$

令$d_x=\nabla_x m, d_y=\nabla_y m$

进行分裂L1,L2项后，反问题化为

$$
\min_{u,d_k,d_y}\left|\left|\left(d_x,d_y\right)\right|\right|_2+\frac{\mu}{2}||Gm-d||^2_2+\frac{\lambda}{2}||d_x-\nabla_xm-b_x||^2_2+\frac{\lambda}{2}||d_y-\nabla_y m-b_y||^2_2
$$

其中

$$
\left|\left|\left(d_x,d_y\right)\right|\right|_2=\sum_{i,j}\sqrt{|d_{x,i,j}|^2+|d_{y,i,j}|^2}
$$

两个子问题分别为

子问题一

$$
m^{k+1}=\min_m\frac{\mu}{2}||Gm-d||^2_2+\frac{\lambda}{2}||d_x-\nabla_xm-b_x||^2_2+\frac{\lambda}{2}||d_y-\nabla_y m-b_y||^2_2
$$

化为岭回归问题

$$\min\left|\left|\begin{bmatrix}G
 \\ \sqrt[]{\frac{\lambda}{2}} \nabla_x
 \\ \sqrt[]{\frac{\lambda}{2}} \nabla_y
\end{bmatrix}m - 
\begin{bmatrix}d
 \\\sqrt[]{\frac{\lambda}{2}}(d_x-b_x)
 \\\sqrt[]{\frac{\lambda}{2}}(d_y-b_y)
\end{bmatrix}\right|\right|_2$$

或

$$\begin{bmatrix}G
 \\ \sqrt[]{\frac{\lambda}{2}} \nabla_x
 \\ I
\end{bmatrix}m =
\begin{bmatrix}d
 \\\sqrt[]{\frac{\lambda}{2}}(d_x-b_x)
  \\\sqrt[]{\frac{\lambda}{2}}(d_y-b_y)\nabla_y^{-1}
\end{bmatrix}$$


子问题二

$$
(d_x^{k+1},d_y^{k+1})=\min_{d_x,d_y}\left|\left|\left(d_x,d_y\right)\right|\right|_2+\frac{\lambda}{2}||d_x-\nabla_xm-b_x||^2_2+\frac{\lambda}{2}||d_y-\nabla_y m-b_y||^2_2
$$

可以通过**广义阈值函数**求解，参考论文[A Fast Algorithm for Image Deblurring with Total Variation Regularization](https://www.researchgate.net/profile/Yilun-Wang-3/publication/250043278_A_Fast_Algorithm_for_Image_Deblurring_with_Total_Variation_Regularization/links/00b7d52b446aa74347000000/A-Fast-Algorithm-for-Image-Deblurring-with-Total-Variation-Regularization.pdf)

迭代过程为

$$
d^{k+1}_x=\max\left(s^k-\frac{1}{\lambda},0\right)\frac{\nabla_xm^k+b^k_x}{s^k} \\
d^{k+1}_y=\max\left(s^k-\frac{1}{\lambda},0\right)\frac{\nabla_ym^k+b^k_y}{s^k} \\
s^k=\sqrt{|\nabla_xm^k+b^k_x|^2+|\nabla_ym^k+b^k_y|^2}
$$

### 拟牛顿法

[深入机器学习系列17-BFGS & L-BFGS](https://zhuanlan.zhihu.com/p/29672873)

[数值优化（六）——拟牛顿法之LBFGS理论与实战](https://zhuanlan.zhihu.com/p/514576143)

[机器学习基础·L-BFGS算法](https://www.jianshu.com/p/6cf95ad7a5b0)

[数值优化（6）——拟牛顿法：SR1，BFGS，DFP，DM条件](https://zhuanlan.zhihu.com/p/144736223)



### BFGS算法

### L-BFGS算法

### 蒙特卡洛

## 目标泛函构建

### 贝叶斯理论

## 正则化项构建

### CV交叉验证

### 广义交叉验证

### 多道反演

以上的优化方法都只针对于单道，但地震数据往往需要多道进行处理

目前本人实现的方法只是循环进行不同道的单道反演，未完待续。。。
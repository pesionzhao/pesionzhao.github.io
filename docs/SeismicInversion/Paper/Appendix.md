## chapter 4 震源

### 4 . 4震源作为应力-位移矢量不连续面
当我们想要考虑分层介质中的源时，我们需要将源的表达形式用等效应力或矩张量以及基于我们在第二章中介绍的应力-位移向量的波传播技术结合起来

我们将研究力系统的笛卡尔分量相对于原点在震中的坐标系。我们用x轴向北，y轴向东，z轴垂直向下。在柱坐标系中，到x轴的方位角φ遵循地理惯例。

对于小的源，我们将用力$\mathcal{E}$和矩张量Mjk来表示点源，等效力系统是

$$f_j=\mathcal{E}_j\delta(\mathbf{x}-\mathbf{x}_S)-\partial_k\{{M}_{jk}\delta(\mathbf{x}-\mathbf{x}_S)\}\tag{4.58}$$

正是这个力系现在将出现在运动方程中，并最终决定应力-位移向量b的微分方程中的强迫项，( 2.24 ) - ( 2.25 )。对于较大的源区，我们可以通过在空间和时间上分离的若干点源贡献的叠加来模拟辐射特性，以处理传播效应。或者，我们可以在源区域上执行体积积分，在这种情况下，每个体积元$d^3\eta$有一个相关的力$\epsilon d^3\eta$和力矩张量$m_{ij}d^3\eta$。

在每种情况下，我们都有在矢量谐波展开中找到系数Fz, FV, FH的问题(参见2.55)。

$$\mathbf{f}=\frac1{2\pi}\int_{-\infty}^\infty\mathrm{d}\omega\mathrm{~e}^{-\mathrm{i}\omega t}\int_0^\infty\mathrm{d}\mathrm{k}\mathrm{~k}\sum_{{m}}[\mathrm{F}_z\mathbf{R}_k^m+\mathrm{F}_V\mathbf{S}_k^m+\mathrm{F}_H\mathbf{T}_k^m]\tag{4.59}$$

如果我们选择柱面坐标系的z轴通过源点xS，那么这个点源的直角分量fx fy fz在水平面z = zS的原点处都是奇异的。系数Fz, FV, FH只会出现在源深度zS处，并且可以通过利用矢量谐波的正交性来评估，因此，例如:

$${F}_{{V}}=\frac1{2\pi}\int_{-\infty}^{\infty}d\mathrm{t}\:\mathrm{e}^{\mathrm{i}\omega\mathrm{t}}\int_{0}^{\infty}d\mathrm{r}\:\mathrm{r}\int_{0}^{2\pi}d\phi\left[\mathbf{S}_{\mathsf{k}}^{\mathrm{m}}\right]^{*}.\mathbf{f}\tag{4.60}$$

为了求水平面上的积分，我们必须利用$J_m(kr)e^{im\phi}$在原点附近的展开式，这样做在笛卡尔坐标系中更方便。导致Fz, FV, FH的积分将使源项的z依赖性不受影响。因此，我们预计力分量Ex, Ey, Ez和描述水平面上重态的矩张量元素(Mxx, Mxy, Myx, Myy)将具有$\delta(z−z_S)$依赖关系。剩余的矩张量元素$(M_{xz}, M_{zx}, M_{yz}, M_{zy}, M_{zz})$将以$\delta '(z−z_S)$项出现。因此，对于每一个角阶m，总强迫项F(2.26)将与z相关

$$\mathbf{F}(\text{k},\text{m},z,\omega)=\mathbf{F}_1(\text{k},\text{m},\omega)\delta(z-z_{\mathbf{S}})+\mathbf{F}_2(\text{k},\text{m},\omega)\delta^{\prime}(z-z_{{S}})\tag{4.61}$$

它就是我们在2.2.2节中讨论过的形式(2.98)。因此，当我们求解一般点源激励下的应力-位移矢量b时，b在源平面z = zS上存在不连续

$$\begin{aligned}
\mathbf{b}(\text{k},{m},z_\text{S}+,\omega)-\mathbf{b}(\text{k},{m},z_\text{S}-,\omega)& =\mathbf{S}({k},{m},z_{S},\mathbf{\omega})  \\
&=\mathbf{F}_1+\omega\mathbf{A}({p},z_{S})\mathbf{F}_2
\end{aligned}\tag{4.62}$$

$\sigma'(z−z_S)$项的存在意味着，虽然强迫项F只出现在式(2.24)-式(2.25)中应力单元的方程中，但由于通过A矩阵的耦合，b向量的不连续也扩展到位移项。应力-位移矢量b在zS上的分量的跳跃很大程度上取决于角顺序:

$$
\begin{array}{ll}
{[U]_{-}^{+}=M_{z z}\left(\rho \alpha^2\right)^{-1}} & m=0 \\
{[V]_{-}^{+}=\frac{1}{2}\left[ \pm M_{x z}-i M_{y z}\right]\left(\rho \beta^2\right)^{-1}} & m= \pm 1 \\
{[W]_{-}^{+}=\frac{1}{2}\left[ \pm M_{y z}-i M_{x z}\right]\left(\rho \beta^2\right)^{-1}} & m= \pm 1
\end{array}
$$



$$
\begin{array}{rll}
{[P]_{-}^{+}} & =-\omega^{-1} \mathcal{E}_z &  m=0 \\
& =\frac{1}{2} p\left[i\left(M_{z y}-M_{y z}\right) \pm\left(M_{x z}-M_{z x}\right)\right] &  m= \pm 1 \\
{[S]_{-}^{+}} & =\frac{1}{2} p\left(M_{x x}+M_{y y}\right)-p M_{z z}\left(1-2 \beta^2 / \alpha^2\right) &  m=0 \\
& =\frac{1}{2} \omega^{-1}\left(\mp \mathcal{E}_x+i \mathcal{E}_y\right) & m= \pm 1 \\
& =\frac{1}{4} p\left[\left(M_{y y}-M_{x x}\right) \pm i\left(M_{x y}+M_{y x}\right)\right] &  m= \pm 2 \\
{[T]_{-}^{+}} & =\frac{1}{2} p\left(M_{x y}-M_{y x}\right),  & m=0 \\
& =\frac{1}{2} \omega^{-1}\left(i \mathcal{E}_x \pm \mathcal{E}_y\right)  & m= \pm 1 \\
& =\frac{1}{4} p\left[ \pm i\left(M_{x x}-M_{y y}\right)+\left(M_{x y}+M_{y x}\right)\right] & m= \pm 2
\end{array}
$$

对于我们的点等效源(4.58)，位移的矢量谐波展开(2.55)将被限制在方位角阶|m| < 2。一般矩张量的这些结果推广了Hudson (1969a)对任意取向位错的分析

我们记得，对于一个固有源，矩张量是对称的因此只有六个独立的分量，进一步的$\mathcal E$就会消失。这导致了这些结果的显著简化。特别是，对于点源，应力变量P将始终是连续的，$m =\pm 1$时的激励仅限于水平位移项，而T仅在$m =\pm 2$时才会有跳跃。

我们已经注意到，位移和牵引量U、V、W、P、S、T的唯一方位角依赖性来自各向同性介质中源的方位角行为。因此，对于固有源，我们可以将分层中位移场的方位角行为与矩张量元素的某些组合联系起来。

- (a)不随方位角变化
  对于P-SV波场，这是由对角线元素(Mxx + Myy), Mzz控制的，对于SH波完全不存在。
- $\cos\phi， \sin\phi$依赖关系:
  这种角行为源于垂直耦合$M_{xz}, M_{yz}$的存在。术语Mxz导致P-SV的$\cos \phi$依赖性和SH的$\sin \phi$依赖性，而Myz则导致P-SV的$\sin\phi$依赖性和SH的$\cos \phi$依赖性。
- $\cos 2\phi， \sin 2\phi$依赖性:
  这种行为由水平偶极子和偶对Mxx, Myy, Mxy控制。差异(Mxx−Myy)导致P-SV的$\cos 2\phi$行为和SH的$\sin 2\phi$行为。一对Mxy给出了P-SV的$\sin 2\phi$依赖和SH的$cos 2φ$依赖。

这些方位依赖关系并不依赖于对介质传播路径性质的任何假设，因此对体波和面波都适用。

### 4.5 震源的波矢量表示

考虑源平面zS附近的局部均匀区域中的源，我们可以通过逆特征向量矩阵$D^{−1}(z_S)$的运算将源平面上方和下方的应力-位移向量$b(z_{S−})，b(z_{S+})$转换为它们的上下波部分。相应的波矢量v将在源平面$z_S$上遭受不连续$\Sigma$，这是由于b的跳变

$$\begin{aligned}
\boldsymbol{v}(\mathrm{k},{m},z_\mathrm{S}+,\omega)-\boldsymbol{v}(\mathrm{k},{m},z_\mathrm{S}-,\omega)& =\boldsymbol{\Sigma}(\mathrm{k},{m},z_\mathrm{S},\omega)  \\
&=\mathbf{D}^{-1}({p},z_{S})\mathbf{S}({k},{m},z_{S},\omega).
\end{aligned}\tag{4.65}$$

我们可以用向上和向下部分来表示这个跳跃向量$\Sigma$，如( 3.17 )中所述。

$$\Sigma=[-\Sigma_{\mathrm{U}},\Sigma_{\mathrm{D}}]^{{T}}\tag{4.66}$$

符号的选择是为了便于物理解释。关系(4.66)的结构可以通过将S划分为位移和牵引跳跃，然后利用$D^{−1}(z_S)$(3.40)的划分形式来理解。

$$\begin{bmatrix}-\Sigma_\mathrm{U}\\\Sigma_\mathrm{D}\end{bmatrix}=\mathrm{i}\begin{bmatrix}n_\mathrm{DS}^\mathrm{T}&-m_\mathrm{DS}^\mathrm{T}\\-n_\mathrm{US}^\mathrm{T}&-m_\mathrm{US}^\mathrm{T}\end{bmatrix}\begin{bmatrix}S_W\\S_\mathrm{T}\end{bmatrix}\tag{4.67}$$

元素$\Sigma U, \Sigma D$具有较为相似的形式

$$\Sigma_{\mathrm{U}}=\mathrm{i}[\mathbf{m}_{\mathrm{D}S}^\mathrm{T}S_\mathrm{T}-\mathbf{n}_{\mathrm{D}S}^\mathrm{T}S_{W}],\quad\quad\Sigma_\mathrm{D}=\mathrm{i}[\mathbf{m}_\mathrm{US}^\mathrm{T}S_\mathrm{T}-\mathbf{n}_{\mathrm{U}S}^\mathrm{T}S_{W}]\tag{4.68}$$

**如果我们考虑一个嵌入在无界介质中的源，这些项的意义是最容易看到的。在这样的波源上方，我们期望只有上行波，下方只有下行波，这样波矢才具有形式**

$$\begin{aligned}{v}(z)&=[{v}_\mathrm{U}(z),{0}]^{T}\quad&z<z_S\\&=[0,{v}_\mathrm{D}(z)]^{T}\quad&z>z_\mathrm{S}\end{aligned}\tag{4.69}$$

穿过zS有跳跃$\Sigma$

$$\Sigma=[-v_\mathrm{U}(z_\mathrm{S}),v_\mathrm{D}(z_\mathrm{S})]^{T}\tag{4.70}$$

比较S的两个表达式(4.66)和式(4.70)可知，源将向无界介质$\Sigma U$向上和$\Sigma D$向下辐射。对于垂直不均匀区域中的源，如果我们在源水平zS处分割介质，并考虑将分层的每两半都由具有zS性质的均匀半空间扩展，我们仍然可以使用跳跃矢量S。这个过程将与我们对反射和透射问题的处理相对应，辐射分量$\Sigma U \Sigma D$将与反射矩阵一起进入地震波场的紧凑物理描述。我们不会因为假设zS处有一个无穷小的均匀区域而失去一般性，因为我们将为源上下的分层区域构造正确的格林张量。
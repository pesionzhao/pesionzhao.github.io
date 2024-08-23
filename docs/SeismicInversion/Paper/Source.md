# Chapter7层状半空间的响应

## 7.1

我们考虑图7.1所示的半空间

![](https://s2.loli.net/2024/03/26/mUQ4VSCaiqM1k6J.png)

其中zS处有一个源，zL下有一个均匀的半空间。在自由表面，牵引力必须消失，因此，对于任何角阶m，频率ω和慢度p，表面应力-位移矢量必须满足

$$\mathbf{b}(0)=[{w}_0,0]^\mathsf{T}\tag{7.1}$$

我们采用点源表示法，正如第四章所讨论的，因此从(4.62)开始，应力-位移向量在源平面上会有一个跳跃

$$\mathbf{b}(z_\text{S}+)-\mathbf{b}(z_\text{S}-)=\mathbf{S}(z_\text{S})\tag{7.2}$$

在分层的基础上，我们可以利用z > zL的地震场的特征向量分解来建立辐射条件。因此，为了排除上行波，我们取

$$\mathbf{b}(z_\mathrm{L})=\mathbf{D}(z_\mathrm{L})[\mathbf{0},\mathbf{c}_\mathrm{D}]^\mathsf{T}\tag{7.3}$$

用下行波元向量cD表示，后者将根据源的性质和半空间的性质来具体说明。对于出现在D(z)中的根式，选择分支切割(3.8)可确保(7.3)具有正确的性质。从( 7.3 )式开始，我们现在可以用传播矩阵P( zS , zL)来构造震源正下方的应力-位移矢量b(zS+)。

$$\mathbf{b}(z_\text{S}+)=\mathbf{P}(z_{S},z_\mathrm{L}){\mathbf b}(z_\mathrm{L})\tag{7.4}$$

随着b向量在zS平面上跳动，我们发现正好在源点的上方。

$$\mathbf{b}(z_\text{S}-)=\mathbf{P}(z_\text{S},z_\text{L})\mathbf{b}(z_\text{L})-\mathbf{S}(z_{\mathsf{S}})\tag{7.5}$$

我们现在可以使用作用于b(zS−)的传播子P(0, zS)来构造表面位移

$$\begin{gathered}
\mathbf{b}(0) =\mathbf{P}(0,z_S)\{\mathbf{P}(z_S,z_L)\mathbf{b}(z_L)-\mathbf{S}(z_S)\} \\
=\mathbf{P}(0,z_\mathrm{L})\mathbf{b}(z_\mathrm{L})-\mathbf{P}(0,z_\mathrm{S})\mathbf{S}(z_\mathrm{S})
\end{gathered}\tag{7.6}$$

其中我们使用了传播链规则( 2.89 )。向量

$$\mathbf{S}(0)=\mathbf{P}(0,z_S)\mathbf{S}(z_S)=[\mathbf{S}_{W0},\mathbf{S}_{T0}]^\mathsf{T}\tag{7.7}$$

表面位移现在可以用zL处的下行波场来表示

$$[w_0,0]^\mathrm{T}=\mathbf{P}(0,z_\mathrm{L})\mathbf{D}(z_\mathrm{L})[0,c_\mathrm{D}]^\mathrm{T}-\mathbf{S}(0)\tag{7.8}$$

因此，无论源的深度如何，w0和cD的关系都由下式限制

$$B_{VL}(0) = P(0, z_L)D(Z_L)\tag{7.9}$$

我们可以认识到这是一个基本的应力-位移矩阵，其列对应于底层半空间中的上行和下行波。对于BVL(0)的分区，我们将(7.8)写为

$$\begin{bmatrix}w_0\\0\end{bmatrix}=\begin{bmatrix}W_{\mathrm{UL}}(0)&T_{\mathrm{UL}}(0)\\W_{\mathrm{DL}}(0)&T_{\mathrm{DL}}(0)\end{bmatrix}\begin{bmatrix}0\\c_{\mathrm{D}}\end{bmatrix}-\begin{bmatrix}S_{\mathrm{W0}}\\S_{\mathrm{T0}}\end{bmatrix}\tag{7.10}$$

由消失牵引力的表面条件可知zL以下的波场为

$$c_D=[{T}_{\mathrm{DL}}(0)]^{-1}{S}_{\mathrm{T0}}\tag{7.11}$$

我们可以把源等效为被位移场的反作用力影响以满足表面条件的牵引力$S_{T0}$。地表位移为

$$w_0=W_{\mathrm{DL}}(0)[T_{\mathrm{DL}}(0)]^{-1}S_{\mathrm{T0}}-S_{\mathrm{W0}}\tag{7.12}$$

假设半空间的长期函数$det\{T_{DL}(0)\}$ 不消失。分量$W_{DL}(0), T_{DL}(0)$是基本矩阵BVL中“下行”向量的位移分量和牵引分量。(7.12)中分量$W_{DL}(0)[T_{DL}(0)]^{−1}$的组合类似于我们在反射问题中遇到的那些(参见5.43)。因此，对于P-SV波的情况，这个矩阵乘积的元素由矩阵BVL(0)的2 × 2次方的比率组成。

$det\{T_{DL}(0)=0\}$的条件，对应于同时满足自由面和辐射条件的位移场的存在;这只会发生在$|p| \gt \beta^{-1}_L$时，因此p波和S波在底层的半空间中都是消失的。在$p \omega$域，这些波与表面响应中的简单极点相关联(7.12)。当变换反向时，慢度最大的极点将产生表面波列(见第11章)。

全表面响应 $w_0$ (7.12)中仅有的其他奇异点是在$|p| = \alpha^{−1}_L， |p| = \beta^{−1}_L$处的分支点，其中$α_L， β_L$是下半空间弹性波波速，这些分支点源于基本矩阵BVL中D(zL)的存在，分支分量将由(3.8)指定。

$$\mathrm{Im}(\omega{q}_{\alpha{L}})\geq0,\quad\mathrm{Im}(\omega{q}_{\mathcal{\beta}{L}})\geq0\tag{7.13}$$

不存在与传播子矩阵$P(0, z_L)$， $P(0, z_S)$相关的分支点。对于均匀层，$P(z_{j-1}, z_j)$关于$q_{αj}，q_{βj}$对称，因此传播子的连续乘积没有分支点.这一性质转移到连续极限。源向量$S(z_S)$，( 4.62 )没有奇点

一旦我们找到了表面位移，其他任何层次的b矢量都有可能被找到

$$\begin{aligned}\mathbf{b}(z)&=\mathbf{P}(z,0)[\boldsymbol{w}_0,0]^\mathsf{T},&z<z_S\\&=\mathbf{P}(z,z_\mathsf{L})\mathbf{D}(z_\mathsf{L})[0,{c}_\mathrm{D}]^\mathsf{T},&z>z_{S}\end{aligned}\tag{7.14}$$

传播子解可以使我们得到地震波场的一个完整的形式说明，但很难对结果做出任何的物理解释

然而，如果我们回忆位移和牵引矩阵$W_{DL}(0)$， $T_{DL}(0)$在反射和传输矩阵$R^{0L}_D$, $T^{0L}_D$(5.56)中的表示，我们有

$$\begin{gathered}
\mathbf{W}_{\mathrm{DL}}(0)=(\mathbf{m}_{\mathrm{D0}}+\mathbf{m}_{\mathrm{U0}}\mathbf{R}_{\mathrm{DL}}^{\mathrm{0L}})(\mathbf{T}_{\mathrm{DL}}^{\mathrm{0L}})^{-1}, \\
\mathbf{T}_{\mathrm{DL}}(0)=(\mathbf{n}_{\mathrm{D0}}+\mathbf{n}_{\mathrm{U0}}\mathbf{R}_{\mathrm{D}}^{\mathrm{0L}})(\mathbf{T}_{\mathrm{D}}^{\mathrm{0L}})^{-1}, 
\end{gathered}\tag{7.15}$$

这些表达式只涉及自由表面下方整个分层的向下反射和透射矩阵。

代入式( 7.15 )，可将地表位移( 7.12 )改写为

$$w_0=({m}_{\mathrm{D0}}+{m}_{\mathrm{U0}}\mathbf{R}_{\mathrm{D}}^{\mathrm{0L}})({n}_{\mathrm{D0}}+{n}_{\mathrm{U0}}\mathbf{R}_{\mathrm{D}}^{\mathrm{0L}})^{-1}S_{\mathrm{T0}}-S_{\mathrm{W0}}\tag{7.16}$$

我们已经看到，$R^{0L}_D$可以在消失区域中构造，而不需要任何与增长指数项相关的数值精度问题，因此( 7.16 )提供了地表源半空间响应的数值稳定表示。**如果使用表面位移( 7.12 )的原始形式，并通过传播矩阵构造$W_{DL}( 0 )$，$T_{DL}(0)$的元素，即使在中等频率下，在大慢度处也会出现严重的精度损失问题。使用式( 7.16 )的一个替代步骤是根据传播子和特征向量矩阵的子式建立从头算解。这样的方案导致非常高效的计算算法**(伍德豪斯, 1981)，但物理内容被抑制

由式(7.16)可得半空间的长期函数为

$$\det\{{T}_{\mathrm{DL}}(0)\}=\det(\mathbf{n}_{\mathrm{D0}}+\mathbf{n}_{\mathrm{U0}}\mathbf{R}_{\mathrm{D}}^{\mathrm{0L}})/\det\mathbf{T}_{\mathrm{D}}^{\mathrm{0L}}\tag{7.17}$$

这个表达式可以很容易地通过一次遍历分层来计算。

如(7.16)所示，在面源向量S(0)的定义中仍然包含一个传播子项$P(0, z_S)$。这对浅源造成的困难很小，例如在勘探应用中。**对于深源，最好是根据反射和透射矩阵来重新定义整个响应，这一过程将在下一节中讨论。**

在自由表面上，向上反射矩阵的形式来自于满足牵引消失的条件：

$$\mathbf{R_F}=-\mathbf{n_{D0}^{-1}n_{U0}}\tag{7.18}$$

因此，通过从式(7.16)中的矩阵逆中提取一个$n_{D0}$因子，我们可以生成地表位移响应的另一种形式

$$w_0=({m}_{\mathrm{D}0}+{m}_{\mathrm{L}0}\mathbf{R}_{\mathrm{D}}^{\mathrm{0L}})[\mathbf{I}-\mathbf{R}_{\mathrm{F}}\mathbf{R}_{\mathrm{D}}^{\mathrm{0L}}]^{-1}\mathbf{n}_{\mathrm{D}0}^{-1}\mathbf{S}_{\mathrm{T}0}-\mathbf{S}_{\mathrm{W}0}\tag{7.19}$$

现在可以清楚地显示自由表面和分层之间的半空间混响算符$[I−R_F R^{0L}_D]^{−1}$，我们可以开始看到总表面位移效应是如何产生的。

## 7.2 深源

我们刚刚看到了如何从一个等效面源和一个位移场中找到半空间的响应，它恰好满足z = zL处的辐射条件。

Kennett(1981)提出的另一种方案是在半空间中由源级表现为上行波和下行波的元素——基本矩阵$B_{VS}$的位移和牵引分区$W_{US}, W_{DS}$和$T_{US}, T_{DS}$——构建整个位移场。

在源平面上，我们有位移和牵引力的间断(4.62)。

$$\begin{aligned}{W}(z_S+)-{W}(z_S-)&={S}_W(z_S),\\{T}(z_S+)-{T}(z_S-)&={S}_T(z_S).\end{aligned}\tag{7.20}$$

在自由表面z = 0处，我们要求不应有牵引力，而在z = zL处，我们希望只有下行波。现在我们在$z < z_S$和$z > z_S$中构造位移场，它们分别满足上边界和下边界条件，但具有常数向量乘子。然后通过施加源条件(7.20)来确定这些因素。

### 通过自由表面反射矩阵进行处理

由我们定义的自由表面反射矩阵$R^{fS}_U$(5.76)-(5.78)的位移矩阵

$$\mathbf{W_{1S}}(z)=\mathbf{W_{US}}(z)+\mathbf{W_{DS}}(z)\mathbf{R_{U}^{fS}},\tag{7.21}$$

在z = 0处没有相关的牵引力。在源上方$z \lt z_S$区域，取位移场即可满足自由曲面边界条件

$$\boldsymbol{W}(z)=\boldsymbol{W}_{1\boldsymbol{S}}(z)\boldsymbol{v}_{1},\quad z<z_{S}\tag{7.22}$$

其中$v_1$是一个常数向量。对于源下方的区域，为位移矩阵

$$\mathbf{W}_{2\mathbf{S}}(z)=\mathbf{W}_{\mathrm{DS}}(z)+\mathbf{W}_{\mathrm{US}}\mathbf{R}_{\mathrm{D}}^{\mathrm{SL}}\tag{7.23}$$

由( 5.90 )可知，满足辐射条件；因此，我们选择一个位移场

$$\boldsymbol{W}(z)=\boldsymbol{W}_{2\boldsymbol{S}}(z)\boldsymbol{v}_{2},\quad z>z_{{S}}\tag{7.24}$$

$v_2$是另一个常数向量。当我们将源级上下位移的表示式(7.22)和式(7.24)插入到源条件(7.20)中时，我们得到以下用$v_1$和$v_2$表示的联立方程

$$\begin{gathered}
{W_{2S}}(z_{\mathcal{S}}){v_{2}}-{W_{1S}}(z_{\mathcal{S}}){v_{1}}={S_{W}}(z_{{S}})\\
{T}_{2S}(z_{S}){v}_{2}-{T}_{1S}(z_{S}){v}_{1}={S}_{{T}}(z_{S})
\end{gathered}\tag{7.25}$$

现在我们可以利用矩阵不变量的性质来消除位移方程和牵引方程之间的变量，就像在(5.63)-(5.69)中处理反射和透射一样。因此我们发现

$$\begin{aligned}v_1&=<\boldsymbol{W}_{1S},\boldsymbol{W}_{2S}>^{-\mathbf{T}}[\boldsymbol{W}_{2S}^\mathrm{T}(z_\mathrm{S})\boldsymbol{S}_\mathrm{T}(z_\mathrm{S})-\boldsymbol{T}_{2S}^\mathrm{T}(z_\mathrm{S}){S}_{\mathrm{W}}(z_\mathrm{S})]\\v_2&=<\boldsymbol{W}_{1S},\boldsymbol{W}_{2S}>^{-1}[\boldsymbol{W}_{1S}^\mathrm{T}(z_\mathrm{S}){S}_\mathrm{T}(z_\mathrm{S})-\boldsymbol{T}_{1S}^\mathrm{T}(z_\mathrm{S}){S}_\mathrm{W}(z_\mathrm{S})]\end{aligned}\tag{7.26}$$

由( 2.72 )

$$<W_{1S},W_{1S}>=<W_{2S},W_{2S}>=0\tag{7.27}$$

因此，满足源和边界条件的位移场为:在z < zS

$$\boldsymbol{W}(z)=\boldsymbol{W}_{\boldsymbol{1S}}(z){<}\boldsymbol{W}_{\boldsymbol{1S}},\boldsymbol{W}_{\boldsymbol{2S}}{>}^{-\mathbf{T}}[\boldsymbol{W}_{\boldsymbol{2S}}^{\mathbf{T}}(z_{\boldsymbol{S}})\boldsymbol{S}_{\mathbf{T}}(z_{\boldsymbol{S}})-\boldsymbol{T}_{\boldsymbol{2S}}^{\mathbf{T}}(z_{\boldsymbol{S}})\boldsymbol{S}_{\boldsymbol{W}}(z_{\boldsymbol{S}})]\tag{7.28}$$

并且对于z > zS

$$W(z)=W_{2S}(z)<W_{1S},W_{2S}>^{-1}[W_{1S}^{{T}}(z_{S})S_{{T}}(z_{S})-{T}_{{S}}^{{T}}(z_{S}){S}_{{W}}(z_{S})]\tag{7.29}$$

有了$W_{1S}、W_{2S}$的表达式，我们可以用源上下区域的反射特性来表示(7.28)。我们可以在源级最容易地计算不变量$<W_{1S}, W_{2S}>$。从定义(7.21)，(7.23)

$$\begin{aligned}<W_{1S},W_{2S}>&=[\mathbf{R}_{U}^\mathrm{fS}]^\mathrm{T}<W_{DS},W_{DS}>+<W_{US},W_{DS}>\mathbf{R}_{D}^\mathrm{SL}\\&+<W_{US},W_{DS}>+[\mathbf{R}_{U}^\mathrm{fS}]^\mathrm{T}<W_{DS},W_{US}>\mathbf{R}_{D}^\mathrm{SL}\end{aligned}\tag{7.30}$$

反射矩阵中的线性项完全消失，并且从(5.60)$<W_{US}, W_{DS}> = iI$。由于反射矩阵是对称的，我们发现

$$<{W}_{1S},{W}_{2S}>=\mathrm{i}[\mathbf{I}-\mathbf{R}_{\mathsf U}^{\mathrm{fS}}\mathbf{R}_{\mathrm{D}}^{\mathrm{SL}}]\tag{7.31}$$

因此位移解(7.28)中出现的逆不变量$<W_{1S}, W_{2S}>^{−1}$是整个半空间的混响算符，包括自由面反射的影响。因此，表面位移可以表示为

$$w_{0}=-\mathrm{i}W_{1S}(0)[\mathbf{I}-\mathbf{R}_{D}^{\mathrm{SL}}\mathbf{R}_{{U}}^{\mathrm{fS}}]^{-1}[W_{2S}^{\mathrm{T}}(z_{\mathsf{S}})\mathbf{S}_{{T}}(z_{{S}})-\mathbf{T}_{2S}^{\mathrm{T}}(z_{{S}})\mathbf{S}_{{W}}(z_{{S}})]\tag{7.32}$$

其中，$W_{1S}$为满足自由面条件的位移矩阵，$W_{2S}$满足下边界条件。当我们在第11章讨论模态求和技术时，表达式(7.32)将被证明是非常方便的。

源层的贡献

$$\begin{aligned}&{W}_{2S}^{{T}}(z_{{S}}){S}_{{T}}(z_{{S}})-{T}_{2S}^{{T}}(z_{{S}}){S}_{{W}}(z_{{S}})\\&=\{{m}_{{DS}}^{{T}}{S}_{{T}}-\mathsf{n}_{{DS}}^{{T}}{S}_{{W}}\}+\mathsf{R}_{{D}}^{{SL}}\{{m}_{{US}}^{{T}}{S}_{{T}}-{n}_{{US}}^{{T}}{S}_{{W}}\}\end{aligned}\tag{7.33}$$

括号中的表达式只是4.5节中介绍的用于描述源向上和向下辐射的数量$\Sigma D(z_S)、\Sigma U(z_S)$的倍数。因此

$$\begin{aligned}&\mathbf{W}_{2S}^\mathrm{T}(z_S)\mathbf{S}_\mathrm{T}(z_S)-\mathbf{T}_{2S}^\mathrm{T}(z_S)\mathbf{S}_\mathrm{W}(z_S)=-\mathrm{i}[\mathbf{\Sigma}_\mathrm{U}(z_S)+\mathbf{R}_\mathrm{D}^\mathrm{SL}\mathbf{\Sigma}_\mathrm{D}(z_S)]\\&\mathbf{W}_\mathrm{IS}^\mathrm{T}(z_S)\mathbf{S}_\mathrm{T}(z_S)-\mathbf{T}_\mathrm{IS}^\mathrm{T}(z_S)\mathbf{S}_\mathrm{W}(z_S)=-\mathrm{i}[\mathbf{\Sigma}_\mathrm{D}(z_S)+\mathbf{R}_\mathrm{U}^\mathrm{fS}\mathbf{\Sigma}_\mathrm{U}(z_S)]\end{aligned}\tag{7.34}$$

当我们将结果(7.31)、(7.34)结合在一起时，我们得到了任意接收平面zR:对于zR < zS的位移场的紧凑而有用的表示

$$\boldsymbol{W}(z_{\mathrm{R}})=[\boldsymbol{W}_{\mathrm{US}}(z_{\mathrm{R}})+\boldsymbol{W}_{\mathrm{DS}}(z_{\mathrm{R}})\mathbf{R}_{\mathrm{U}}^{fS}][\mathbf{I}-\mathbf{R}_{\mathrm{D}}^{\mathrm{SL}}\mathbf{R}_{\mathrm{U}}^{fS}]^{-1}[\mathbf{\Sigma}_{\mathrm{U}}(z_{S})+\mathbf{R}_{\mathrm{D}}^{\mathrm{SL}}\mathbf{\Sigma}_{\mathrm{D}}(z_{S})]\tag{7.35}$$

对于zR > zS，

$$\boldsymbol{W}(z_{\mathrm{R}})=[\boldsymbol{W}_{\mathrm{DS}}(z_{\mathrm{R}})+\boldsymbol{W}_{\mathrm{LS}}(z_{\mathrm{R}})\mathbf{R}_{\mathrm{D}}^{\mathrm{SL}}][\mathbf{I}-\mathbf{R}_{\mathrm{U}}^{\mathrm{fS}}\mathbf{R}_{\mathrm{D}}^{\mathrm{SL}}]^{-\mathbf{1}}[\mathbf{\Sigma}_{\mathrm{D}}(z_{\mathrm{S}})+\mathbf{R}_{\mathrm{U}}^{\mathrm{fS}}\mathbf{\Sigma}_{\mathrm{U}}(z_{\mathrm{S}})]\tag{7.36}$$

这种位移表示分为三个贡献，我们将通过考虑源上方的接收器来说明(参见图7.2)。

![](https://s2.loli.net/2024/03/29/UkXoB6h2xeKFZ94.png)

**首先是源贡献**

$$\Sigma_{{U}}(z_{{S}})+\mathbf{R}_{{D}}^{{SL}}{\Sigma}_{{D}}(z_{{S}})\tag{7.37}$$

这相当于在z = zS能级上与源相关的全部向上辐射。这部分是由直接向上的辐射${\Sigma U(zS)}$产生的，部分是由最初向下的波产生的，但已被反射回源水平以下${R^{SL}_D \Sigma D(z_S)}$。**这个激励矢量，对于各向同性介质，将取决于方位角的m阶，而反射矩阵与m无关。**

**第二个贡献**来自逆不变量$<W1S, W2S>^{-1}$，它只是一个混响算符

$$[\mathbf{I}-\mathbf{R}_\mathrm{D}^\mathrm{SL}\mathbf{R}_\mathrm{U}^\mathrm{fS}]^{-1}\tag{7.38}$$

将半空间的上半部分(包括自由表面反射的影响)耦合到源级的下半部分。半空间的长期函数是

$$\det[\mathbf{I}-\mathbf{R}_{D}^{\mathrm{SL}}\mathbf{R}_{\mathrm{U}}^{\mathrm{fS}}]=0\tag{7.39}$$

式(7.39)满足时，式(7.35)中的极奇点对应于同时满足上下边界条件的位移场的存在性。对于传播波，我们可以对混响算子进行级数展开，如式6.17所示。

$$[\mathbf{I}-\mathbf{R}_D^{\mathrm{SL}}\mathbf{R}_\mathrm{U}^{\mathrm{fS}}]^{-1}=\mathbf{I}+\mathbf{R}_D^{\mathrm{SL}}\mathbf{R}_\mathrm{U}^{\mathrm{fS}}+\mathbf{R}_D^{\mathrm{SL}}\mathbf{R}_\mathrm{U}^{\mathrm{fS}}\mathbf{R}_\mathrm{D}^{\mathrm{SL}}\mathbf{R}_\mathrm{U}^{\mathrm{fS}}+...\tag{7.40}$$

那么我们就可以认识到组合后的作用

$$[\mathbf{I}-\mathbf{R}_\mathrm{D}^\mathrm{SL}\mathbf{R}_\mathrm{U}^\mathrm{fS}]^{-1}[\mathbf{\Sigma}_\mathrm{U}(z_\mathrm{S})+\mathbf{R}_\mathrm{D}^\mathrm{SL}\mathbf{\Sigma}_\mathrm{D}(z_\mathrm{S})]={v}_\mathrm{U}(z_\mathrm{S})\tag{7.41}$$

是在源级产生一系列的上行波群，这些上行波群对应于源的辐射在半空间内受到连续高阶多次混响的影响。如图7.2a所示。

**第三是响应中与接收器位置相对应的部分**

$$\mathbf{W_{US}(z_{R})}+\mathbf{W_{DS}(z_{R})R_{U}^{fS}}\tag{7.42}$$

在它的物理意义被接受之前，需要从反射和传输的角度来重新塑造。我们将位移矩阵$W_{US}(zR)$和$W_{DS}(z_R)$分别表示为$z_R$处的上、下场，以及区域$R_S$的性质，如(6.7)所示，并利用自由表面反射的加法规则(6.10)给出

$$\begin{aligned}
&W_{\mathrm{US}}(z_{\mathrm{R}}) +{W_{DS}}(z_{\mathrm{R}})\mathrm{R_{U}^{fS}}  \\
&=m_{\mathrm{UR}}T_{\mathrm{U}}^{\mathrm{RS}}+[m_{\mathrm{DR}}+m_{\mathrm{UR}}R_{\mathrm{D}}^{\mathrm{RS}}]R_{\mathrm{U}}^{\mathrm{fR}}[\mathrm{I}-R_{\mathrm{D}}^{\mathrm{RS}}R_{\mathrm{U}}^{\mathrm{fR}}]^{-1}T_{\mathrm{U}}^{\mathrm{RS}}
\end{aligned}\tag{7.43}$$

这个表达式可以简化成更紧凑的形式, 它的结构类似于式(6.11)

$$\begin{aligned}
W_{\mathrm{US}}(z_{\mathrm{R}})& +{W_{DS}}(z_{\mathrm{R}})\mathbf{R}_{U}^{\mathrm{fS}}  \\
&=\{\mathbf{m}_{\mathrm{UR}}+\mathbf{m}_{\mathrm{DR}}\mathbf{R}_{\mathrm{U}}^{\mathrm{fR}}\}[\mathbf{I}-\mathbf{R}_{\mathrm{D}}^{\mathrm{RS}}\mathbf{R}_{\mathrm{U}}^{\mathrm{fR}}]^{-1}\mathbf{T}_{\mathrm{U}}^{\mathrm{RS}}
\end{aligned}\tag{7.44}$$

当我们从(7.44)和式(7.41)构造接收端的位移时，对于zR < zS

$${W}(z_{\mathrm{R}})=\{\mathbf{m_{UR}}+\mathbf{m_{DR}R_{U}^{fR}}\}[\mathbf{I}-\mathbf{R_{D}^{RS}R_{U}^{fR}}]^{-1}\mathbf{T}_{U}^{RS}v_{U}(z_{S})\tag{7.45}$$

它首先由Kennett & Kerry(1979)提出，并有一个相当不同的推导。$v_U(z_S)$中的每一波组在传输矩阵$T^{RS}_U$的作用下被投射到接收端(图7 2b)。在源和表面之间的区域内，接收器附近的混响用运算符 $[I−R^{RS}_D R^{fR}_U]^{−1}$ 表示。最后，通过使用适当的变换矩阵 ${m_{UR} + m_{DR}R^{fR}_U}$ 将先前在自由表面反射的上行波的作用加到下行波上，从而产生位移。

我们不仅对(7.35)的贡献有一个现成的物理解释，而且它们都可以用反射和传输矩阵来表示，这些矩阵可以在没有精度损失问题的情况下构造。因此，如果我们使用(7.45)作为接收项，则表示(7.35)对于高频深源在数值上是稳定的, 对于表面接收器(7.45)，减少到$W^{fS}_U$，因此地面位移可以从下式得到

$$w_0=\mathbf{W_F[I-R_D^{0S}R_F]^{-1}T_U^{0S}[I-R_D^{SL}R_U^{fS}]^{-1}[\Sigma_U(z_S)+R_D^{SL}\Sigma_D(z_S)]}\tag{7.46}$$

式(5.86)中，表面放大系数$W_F = (m_{U0} + m_{D0}R^F)$。**该表达式对于构造半空间中任意深度源的地表位移是非常方便的。**

**当接受器在源下面时**

$$v_{\mathrm{D}}(z_{\mathrm{S}})=[\mathbf{I}-\mathbf{R}_{\mathrm{U}}^{\mathrm{fS}}\mathbf{R}_{\mathrm{D}}^{\mathrm{SL}}]^{-1}[\mathbf{\Sigma}_{\mathrm{D}}(z_{\mathrm{S}})+\mathbf{R}_{\mathrm{U}}^{\mathrm{fS}}\mathbf{\Sigma}_{\mathrm{U}}(z_{\mathrm{S}})]\tag{7.47}$$

给出了在整个半空间内混响后源的净向下辐射。接收器的贡献可以用类似于上面描述的方法来计算，得到zR > zS时的位移场为

$${W}(z_{\mathrm{R}})=\{\boldsymbol{m_{DR}}+\boldsymbol{m_{UR}}\mathbf{R_{D}^{RL}}\}[\mathbf{I}-\mathbf{R_{U}^{RS}}\mathbf{R_{D}^{RL}}]^{-1}\mathbf{T_{D}^{RS}}{v_{D}}(z_{S})\tag{7.48}$$

这样就可以在声源以下的区域产生传播和混响效应。

**如果我们专门研究一个表面源，在表面下有一个接收器，那么我们可以把(7.47)和(7.48)结合起来给出**

$$W(0+)=(m_{D0}+m_{U0}R_{D}^{0L})[I-R_{F}R_{D}^{0L}]^{-1}[\Sigma_{D}(0)+R_{F}\Sigma_{U}(0)]\tag{7.49}$$

由于$R^F =−n^{−1}_{D0}n_{U0}$ 所以我们可以写

$$W(0+)=(m_{D0}+m_{u0}R_{D}^{0L})(n_{D0}+n_{u0}R_{D}^{0L})^{-1}[n_{D0}\Sigma_{D}(0)-n_{u0}\Sigma_{{U}}(0)]\tag{7.50}$$

由上下源分量的定义(4.68)可知$[n_{D0}\Sigma D(0)−n_{U0}\Sigma U(0)]$为源矢量S(0)的牵引分量ST0。表面处的位移包含了S ( 0 )的位移分量。故

$$w_0=W(0+)-S_{W0}\tag{7.51}$$

由式(7.50)和式(7.51)给出的地表位移恰好具有式(7.16)的形式，因此我们看到第7.1节中使用的初值技术和本节中开发的两点边界值方法是等价的。

**我们刚才讨论的构造位移场的方法是通用的，并不局限于自由表面和向下辐射的条件(Kennett, 1981)**。更一般的情况可以通过将$R^{fS}_U$替换为适用于z = 0的新上界条件的反射矩阵$R^S_1$，将$R^{SL}_D$替换为适用于z = zL的新下界条件的$R^S_2$来构造。位移解中会出现逆不变量$<W_{1S}, W_{2S}>^{−1}$，一般情况下

$$<W_{1S},W_{2S}>^{-1}=-\mathrm{i}[\mathbf{I}-\mathbf{R}_1^S\mathbf{R}_2^S]^{-1}\tag{7.52}$$

因此，一个特定问题的长期函数可以用相应的反射矩阵$R^S_1, R^S_2$表示为$det[I−R^S_1 R^S_2]$。对于不同的源深度，我们得到的长期函数只相差一个因子，且具有相同的零点。长期行列式的消失代表了在层位zS以上和以下连续反射的波的一种建设性干涉条件。

对于波在球形分层中的传播，我们可以利用上述结果求出位移场$W(l, m, R， ω)$。在这种情况下，我们选择位移矩阵$W_{1S}$满足R = re处的自由曲面边界条件，$W_{2S}$满足原点处的正则性条件。位移场的表达式(7.28)可以直接使用，只要我们记得z和R在相反方向上增加。对于球形情况，我们也可以使用反射矩阵(7.46)表示表面位移，前提是我们将RSL D解释为具有原点正则条件的区域R < RS的反射矩阵(参见第5.2.4节)。结果的物理解释与水平分层不同，因此对两种情况的完全响应的近似将并行运行。

### 自由表面反射的显式表示

在前面的处理中，我们选择在满足上下边界条件的位移矩阵中工作。然而，如果我们构造

$${W_{3S}}(z)=\boldsymbol{W_{\mathrm{US}}}(z)+\boldsymbol{W_{\mathrm{DS}}}(z)\mathbf{R_{U}^{0S}}\tag{7.53}$$

这对应于z = 0处的辐射边界条件。为了满足实际的自由面条件，我们选择位移矩阵$W_{3S}，W_{2S}$在z < zS内的线性组合

$$\mathbf{W}(z)=\mathbf{W}_{3\mathbf{S}}(z)\mathbf{u}_{3}+\mathbf{W}_{2\mathbf{S}}(z)\mathbf{u}_{1}\tag{7.54}$$

并通过要求在z = 0处牵引力为零来确定u1和u3之间的关系

$${T}_{3S}(0)\mathbf{u}_3+{T}_{2S}(0)\mathbf{u}_1=0\tag{7.55}$$

在源下面

$$\boldsymbol{W}(z)=\boldsymbol{W}_{2\boldsymbol{S}}(z)\boldsymbol{u}_2\tag{7.56}$$

它满足下边界条件。源条件(7.20)现在导致(u2 - u1)和u3的联立方程:

$$\begin{aligned}\mathbf{W}_{2\mathbf{S}}(z_{\mathbf{S}})(\mathbf{u}_{2}-\mathbf{u}_{1})-\mathbf{W}_{3\mathbf{S}}(z_{\mathbf{S}})\mathbf{u}_{3}&=\mathbf{S}_{\mathsf{W}}(z_{\mathbf{S}})\\\mathbf{T}_{2\mathbf{S}}(z_{\mathbf{S}})(\mathbf{u}_{2}-\mathbf{u}_{1})-\mathbf{T}_{{3S}}(z_{\mathbf{S}})\mathbf{u}_{3}&=\mathbf{S}_{{T}}(z_{\mathbf{S}})\end{aligned}\tag{7.57}$$

由于我们将希望专注于表面位移，我们求解u3。

$$u_{3}=<W_{3S},W_{2S}>^{-{T}}[W_{2S}^{{T}}(z_{{S}})\mathbf{S}_{{T}}(z_{{S}})-\mathbf{T}_{2S}^{{T}}(z_{{S}})\mathbf{S}_{{W}}(z_{{S}})]\tag{7.58}$$

逆不变量可以通过类比式(7.31)找到，我们之前已经计算了(7.34)中的源项，因此

$$\mathbf{u}_{3}=[\mathbf{I}-\mathbf{R}_{\mathrm{D}}^{\mathrm{SL}}\mathbf{R}_{\mathrm{D}}^{\mathrm{0S}}]^{-1}[\mathbf{\Sigma}_{\mathrm{U}}(z_{\mathrm{S}})+\mathbf{R}_{\mathrm{D}}^{\mathrm{SL}}\mathbf{\Sigma}_{\mathrm{D}}(z_{\mathrm{S}})]\tag{7.59}$$

则震源上方区域的位移由下式给出

$$\mathbf{W}(z)=\{\mathbf{W}_{3\mathbf{S}}(z)-\mathbf{W}_{2\mathbf{S}}(z)\mathbf{T}_{2\mathbf{S}}^{-1}(0)\mathbf{T}_{3\mathbf{S}}(0)\}\mathbf{u}_{3}\tag{7.60}$$

其中在z = 0处有特别简单的形式。由式( 5.57 )可得

$$W_{3S}(0)={m}_{{U}0}\mathbf{T}_{{U}}^\mathrm{0S},\quad{T}_{{S}S}(0)={n}_{{U}0}\mathbf{T}_{{U}}^\mathrm{0S}\tag{7.61}$$

满足下边界条件的位移和牵引力矩阵具有更复杂的形式

$$\begin{gathered}
\mathbf{W_{2S}}(0)=(\mathrm{m_{D0}}+\mathrm{m_{u0}}\mathbf{R_{D}^{0L}})(\mathbf{T_{D}^{0S}})^{-1}(\mathbf{I}-\mathbf{R_{U}^{0S}}\mathbf{R_{D}^{SL}}) \\
T_{2S}(0)=(n_{D0}+n_{u0}R_{D}^{0L})(T_{D}^{0S})^{-1}(I-R_{U}^{0S}R_{D}^{SL}) 
\end{gathered}\tag{7.62}$$

但$W_{2S}(0)T^{−1}_{2S}(0)$比较简单

$$W_{2S}(0)T_{2S}^{-1}(0)=(m_{D0}+m_{U0}R_{D}^{0L})(n_{D0}+n_{U0}R_{D}^{0L})^{-1}\tag{7.63}$$

这种术语组合已经出现在我们的表面源表示中( 7.16 )。表面位移现在形式为

$$w_0=\{{m}_{{U}0}-({m}_{{D}0}+{m}_{{U}0}\mathbf{R}_{{D}}^{{0}\mathbf{L}})({n}_{{D}0}+{n}_{{U}0}\mathbf{R}_{{D}}^{{0}{L}})^{-1}{n}_{{U}0}\}\boldsymbol{\sigma}(z_{{S}})\tag{7.64}$$

其中我们引入了上行波矢量

$$\mathbf{\sigma}(z_{{S}})=\mathbf{T}_{{U}}^{\mathrm{0S}}[\mathbf{I}-\mathbf{R}_{{D}}^{\mathrm{SL}}\mathbf{R}_{{U}}^{\mathrm{0S}}]^{-1}[\mathbf{\Sigma}_{{U}}(z_{{S}})+\mathbf{R}_{{D}}^{\mathrm{SL}}\mathbf{\Sigma}_{{D}}(z_{{S}})]\tag{7.65}$$

矢量σ包括源与源上下层状结构的所有相互作用，但与(7.41)不同的是，σ不允许在自由表面产生反射。在(7.64)中，这样的反射包含在大括号中的项中。这种依赖关系可以通过将(7.64)写成自由面反射矩阵RF并重新排列得到来强调

$$w_0=({m}_{{U}0}+{m}_{{D}0}\mathbf{R}_{{F}})[\mathbf{I}-\mathbf{R}_{{D}}^{{0}\mathbf{L}}\mathbf{R}_{{F}}]^{-1}\mathbf{\sigma}(z_{{S}})\tag{7.66}$$

这样就可以清楚地显示半空间的混响算符。当源仅在表面时，σ(zS)的形式特别简单，因为$T^{0S}_U$成为单位矩阵，$R^{0S}_D$消失，因此

$$\sigma(0+)=[\Sigma_{{U}}(0+)+R_{{D}}^{{0L}}\Sigma_{{D}}(0+)]\tag{7.67}$$
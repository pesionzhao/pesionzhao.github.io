当我们将傅里叶-汉克尔变换算子(2.19)应用于方程集(2.16)和(2.18)时，我们得到了新的位移和应力量U(k, m, z， ω)， P(k, m, z， ω)等的耦合常微分方程集。如果我们以水平慢度p = k/ω为单位，而不是以水平波数k为单位，那么这些转换后的方程就会有一个非常方便的形式,对于P-SV波

$$\frac\partial{\partial z}\begin{bmatrix}
\mathrm{U}\\\mathrm{V}\\\mathrm{P}\\\mathrm{S}\end{bmatrix}
=\omega\begin{bmatrix}
0&{p}(1-2\mathbf{\beta}^2/\alpha^2)&(\mathbf{\rho}\alpha^2)^{-1}&0\\
-{p}&0&0&(\mathbf{\rho}\beta^2)^{-1}\\
-\mathbf{\rho}&0&0&{p}\\
0&\rho[v{p}^2-1]&-{p}(1-2\mathbf{\beta}^2/\alpha^2)&0
\end{bmatrix}\begin{bmatrix}\mathrm{U}\\\mathrm{V}\\\mathrm{P}\\\mathrm{S}\end{bmatrix}-\begin{bmatrix}0\\0\\\mathrm{F}_z\\\mathrm{F}_V\end{bmatrix}\tag{2.24}$$

其中$v=4\beta^2(1-\beta^2/\alpha^2)$

对于SH波

$$\frac\partial{\partial z}\begin{bmatrix}W\\{T}\end{bmatrix}=\omega\begin{bmatrix}0&(\rho\beta^2)^{-1}\\\rho[\beta^2p^2-1]&0\end{bmatrix}\begin{bmatrix}W\\{T}\end{bmatrix}=\begin{bmatrix}0\\{F}_{{H}}\end{bmatrix}\tag{2.25}$$

对于各向同性介质，(2.24)、(2.25)中出现的系数与方位角m阶无关，而U(k, m, z， ω)等的方位角依赖性将仅由力系统f的性质产生。耦合矩阵的元素仅涉及深度z处的弹性参数，而不涉及其垂直导数。这个理想的性质是由Alterman, Jarosch & Pekeris(1959)在一个球体的类似发展中首先指出的，这使得(2.24)，(2.25)非常适合于数值解，因为在插值弹性参数a时涉及的误差

每一组耦合方程(2.24)和式(2.25)都可以写成

$$\partial_z\mathbf{b}({k},{m},z,\omega)=\omega\mathbf{A}({p},z){b}({k},{m},z,\omega)+\mathbf{F}({k},{m},z,\omega)\tag{2.26}$$

用列向量b表示它的分量是位移和应力量。对于P-SV波

$$\mathbf{b}_{{P}}({k},{m},z,\omega)=[{U},{V},P,S]^{\mathsf{T}}\tag{2.27}$$

对于SH波

$$\mathbf{b}_{{H}}({k},{m},z,\omega)=[{W},{T}]^{\mathsf{T}}\tag{2.28}$$

当我们想看一下结果的一般结构时，我们将把一般应力-位移向量b写成下面的形式

$$\mathbf{b}_({k},{m},z,\omega)=[w,t]^{\mathsf{T}}\tag{2.29}$$

### 2.2.2
一旦在层结中存在某种形式的源，我们就必须求解非齐次方程( 2.26 )。

$$\partial_z\mathbf{b}(z)-\omega\mathbf{A}({p},z)\mathbf{b}(z)=\mathbf{F}(z)\tag{2.94}$$

在应力-位移向量b上受到一些初始条件的限制。由于传播算子$P^{-1}( z , z0)$的逆满足
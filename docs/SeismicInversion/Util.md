## SEGY地震数据

### 简介

SEGY（Seismic Exploration Geophysical Data）格式是一种常见的用于存储地震勘探数据的标准格式。SEGY格式的数据通常存储在磁盘文件中，文件通常以“.sgy”或“.segy”作为扩展名。

SEGY文件由三部分组成：文件头（File Header）、道头（Trace Header）和数据部分（Data Section）。

文件头包含了所有道的通用信息，例如数据的采样率、数据类型、地震仪器的参数等。文件头通常包含3200个字节的信息。

道头包含了单个地震道的信息，例如该道的坐标位置、时间信息、道号等。道头的长度通常为240个字节。

数据部分包含了所有的地震数据，每个数据点都是一个16位或32位的整数。在SEGY格式中，数据是以“道”（Trace）的形式组织的，每个道代表了一次地震记录的结果，道的数量取决于勘探过程中所采集的数据点数和勘探面积的大小。

总之，SEGY格式是一种灵活的地震数据存储方式，它可以方便地存储大量的地震勘探数据，并且能够提供高效的数据读取和处理能力。

SEGY格式可以分为IBM格式和IEEE格式两种存储方式。

IBM格式是指IBM公司的二进制浮点数格式，也称为"big-endian"（大端）格式。在IBM格式中，浮点数存储为四个字节的二进制数，其中第一个字节表示符号位和指数部分，而后三个字节表示小数部分。在IBM格式中，数据的字节顺序是从左到右的，即最高位字节存储在最左边，最低位字节存储在最右边。因此，IBM格式也称为"大端字节序"（big-endian byte order）。

IEEE格式是指基于IEEE标准的二进制浮点数格式，也称为"little-endian"（小端）格式。在IEEE格式中，浮点数存储为四个字节的二进制数，其中第一个字节表示小数部分，而后三个字节表示符号位和指数部分。在IEEE格式中，数据的字节顺序是从右到左的，即最低位字节存储在最左边，最高位字节存储在最右边。因此，IEEE格式也称为"小端字节序"（little-endian byte order）。

### 存储格式

具体存储格式见下文件附录

[用 C 语言读写 SGY 格式的地震数据](https://www.doc88.com/p-6857792235992.html)



可以通过将地震数据读取到hampson russell中，点击header editor查看文件头与道头各个位置对应的数据，便于调试

### 具体代码

``` python
import struct

# 打开SEGY文件
with open(datapath, 'rb') as f:

    # 读取二进制文件头
    bin_header = f.read(3600)

    # 获取文件头中的相关信息
    sample_interval = struct.unpack_from(">h", bin_header, 3216)[0] # >h表示大端short
    num_samples = struct.unpack_from(">h", bin_header, 3220)[0] # 采样点

    # 读取所有道数据
    traces = [] # 某一CDP的道集
    offset = [] # 同一CDP道集不同数据对应的偏移距
    cdp = 330
    while True:
        trace_header = f.read(240) # 读取道头
        if not trace_header:
            break
        cdp_cur = struct.unpack_from(">i", trace_header, 20)[0] # 道头CDP
        if cdp_cur == cdp:
            amp = []
            for _ in range(num_samples):
                amp.append(struct.unpack_from(">h", f.read(2))[0]) # 这里的地震数据是两字节的short
            traces.append(amp)
            offset.append(struct.unpack_from(">i", trace_header, 36)[0]) # 道头偏移距
        else:
            data = f.read(num_samples * 2) # 更改文件指针
            continue
```

### SU数据

与segy数据类似，只不过没有3600字节的卷头

[USING SeismicUnix](https://www.geo.uib.no/eworkshop/index.php?n=Main.SeismicUnix)

```python
def read_custom_SU(datapath, savepath):
    cdp = 200
    traces = []
    n = 0
    with open(datapath, 'rb') as f:
        while True:
            trace_header = f.read(240) # 道头数据
            n = n+1
            if not trace_header:
                break
            cdp_cur = struct.unpack_from("<i", trace_header, 20)[0] # CDP
            num_samples = struct.unpack_from("<h", trace_header, 114)[0] # 采样点数
            dt = struct.unpack_from("<h", trace_header, 116)[0] # 采样间隔 μs
            if cdp_cur >= 0:
                amp = []
                for _ in range(num_samples):
                    amp.append(struct.unpack_from("<f", f.read(4))[0])
                traces.append(np.array(amp))
            else:
                data = f.read(num_samples * 4)
                continue

        traces = np.array(traces)
    np.save(savepath, traces)
    print(f'traces = {n}, layers = {num_sample}')
```


## 子波

常用的为Ricker子波

$$ f(t) = (1 - 2\pi^2f_0^2t^2)e^{-\pi^2f_0^2t^2}$$

生成Ricker子波代码为

```python
def z_ricker(t, f0):
    """

    Args:
        t: 时域抽样点,都为正值
        f0: 主频

    Returns:
        Ricker子波时域幅值

    """

    t = np.concatenate((np.flipud(-t[1:]), t), axis=0)  # 将时域映射至整个实轴
    w = (1 - 2 * (np.pi * f0 * t) ** 2) * np.exp(-((np.pi * f0 * t) ** 2))  # Ricker子波计算公式
    return w
```

## AVO建模

### 两项AVO方程(AB method)

$$
R(\theta) = R_0+B\sin^2(\theta)
$$

其中$R_0$代表零偏移距反射系数，即截距，$B$代表梯度,可以在很大程度上简化分析
,对于角度小于 40 度的情况, 第三项不是非常重要

### Aki-Richards方程
Aki-Richards 方程是Zoeppritz 方程的线性近似模拟，代码实现见[Aki-Richards非线性近似](#aki-richards-equation)

$$
\begin{array}{ll}
R_P(\theta)=a \frac{\Delta V_P}{2 V_P}+b \frac{\Delta V_S}{2 V_S}+c \frac{\Delta \rho}{2 \rho}, & \text { where }: \\
a=\frac{1}{\cos ^2 \theta}, & \rho=\frac{\rho_2+\rho_1}{2}, \Delta \rho=\rho_2-\rho_1, \\
b=-8 K \sin ^2 \theta, & V_P=\frac{V_{P 2}+V_{P 1}}{2}, \Delta V_P=V_{P 2}-V_{P 1}, \\
c=1-4 K \sin ^2 \theta, & V_S=\frac{V_{S 2}+V_{S 1}}{2}, \Delta V_S=V_{S 2}-V_{S 1}, \\
K=\left(\frac{V_S}{V_P}\right)^2, & \text { and } \theta=\frac{\theta_1+\theta_2}{2} .
\end{array}
$$

### Aki-Richards方程的Wiggins形式(ABC method)
Wiggins(1983)等人发展了一种近似的，重写的方程形式。他们将方程分为三个
反射系数项, 每一个都比前面的项更简单

$$
R_p(\theta) =  A+B\sin^2(\theta)+C\tan^2(\theta)\sin^2(\theta) 
$$

where:

$$
\begin{aligned}
& A=\frac{1}{2}\left[\frac{\Delta V_P}{V_p}+\frac{\Delta \rho}{\rho}\right], \\
& B=\frac{1}{2} \frac{\Delta V_P}{V_p}-4\left[\frac{V_S}{V_P}\right]^2 \frac{\Delta V_S}{V_S}-2\left[\frac{V_S}{V_P}\right]^2 \frac{\Delta \rho}{\rho}, \\
& C=\frac{1}{2} \frac{\Delta V_P}{V_p} .
\end{aligned}
$$


A是截距, B是梯度, 以及C是曲率,我们把这个方程称为ABC方程

### Aki-Richards方程的Fatti形式
Fatti 等人 (Geophysics, September, 1994) 发展了第三个Aki-Richards方程的
近似公式，被写成:

$$
\begin{gathered}
R_P(\theta)=c_1 R_P\left(0^{\circ}\right)+c_2 R_S\left(0^{\circ}\right)+c_3 R_D, \text { 其中: } \\
c_1=1+\tan ^2 \theta, c_2=-8 K \sin ^2 \theta, c_3=2 K \sin ^2 \theta-\frac{1}{2} \tan ^2 \theta, K=\left(\frac{V_S}{V_P}\right)^2, \\
R_P\left(0^{\circ}\right)=\frac{1}{2}\left[\frac{\Delta V_P}{V_P}+\frac{\Delta \rho}{\rho}\right], R_S\left(0^{\circ}\right)=\frac{1}{2}\left[\frac{\Delta V_S}{V_S}+\frac{\Delta \rho}{\rho}\right], \text { and } R_D=\frac{\Delta \rho}{\rho} .
\end{gathered}
$$

注意到上面的$R_p(0^\circ)$项和前面方程中的 A 项是相同的，并且前面两项的系数和原方程是相同的。

### 对于Aki-Richards方程小结

| 方程名称              | 权系数   | 弹性参数      |
| :--------------------| -------  | ------------- |
| Aki-Richards         | $a,b,c$  | $\frac{\Delta V_P}{2 V_P}, \frac{\Delta V_S}{2 V_S}, \frac{\Delta \rho}{2 \rho}$ |
| Wiggins et al        | $1,\sin^2(\theta),\tan^2(\theta)\sin^2(\theta)$  | $A,B,C$    |
| Fatti et al          |$c_1,c_2,c_3$| $R_P\left(0^{\circ}\right), R_S\left(0^{\circ}\right) R_D$ |

注意到权系数 b, c 和 c2, c3 包含着 VS/VP 比的平方以及$\theta$的三角函数. 但是, 在Wiggins等人的方程中, 这个项包含在弹性参数 B 中

下面是对于三个方程的物理学解释:
- 因为地震道包含着波阻抗的变化，而不是只随速度或者只随密度而变化，所以Aki-Richards方程的原始形式很少被使用
- Aki-Richards方程的A, B, C项对于提取有关 AVO 效应的经验信息 (比如 A, 被称为截距, B, 被称为梯度, 以及 C, 被称为曲率)是非常有用的，这些经验信息可以被单独显示分析或者进行交会分析。权系数并不需要有关VP/VS 比的明确信息
- Fatti 等人发展的方程让我们可以提取有关 P 和 S 反射率的定量信息，这可以用来进行叠前反演。RP0 和 RS0 是线性化后的零角度 P和 S 波反射系数

### Shuey两项AVO方程
Shuey (1985) 使用VP, $\rho$ 和 $\sigma$ 重写了ABC方程. 仅仅梯度和ABC 方程有所不同:

$$
B=A\left[D-2(1+D)\frac{1-2\sigma}{1-\sigma}\right]+\frac{\Delta\sigma}{(1-\sigma)^2}
$$

where:

$$
D=\frac{\Delta V_p /V_p}{\Delta V_p /V_p+\Delta \rho /\rho}, \sigma = \frac{\sigma_1+\sigma_2}{2},  \Delta\sigma = \sigma_2-\sigma_1
$$

上面的方程比较复杂，但是通过假设$\sigma$ = 1/3 (即Vp/Vs=2)，可以被极大地简化成
以下形式:

$$
B=A\left[D-2(1+D)\frac{1}{2}\right]+\frac{9\Delta\sigma}{4}=2.25\Delta\sigma-A
$$

这可以很直观地写成两项AVO方程:

$$
R_p(\theta) = A + (2.25\Delta\sigma-A)\sin^2\theta
$$

## AVO分析（估计截距与梯度）

### 从偏移距道集数据转换到角度道集数据
可以使用下面的方式将偏移距道集转换到角度道集: TODO代码实现 

- 直射线假设 (常速度)
- 射线参数假设 (变速度假设)
- 射线追踪 (变速度)

GPT的回答??

1. 读取地震数据

使用NumPy的loadtxt函数读取地震数据，数据应该以文本文件的形式存储。

2. 将地震数据转换为二维数组

将读取的文本数据转换为二维数组，其中每一行代表一个道集。

3. 对每个道集进行傅里叶变换

使用NumPy的fft函数对每个道集进行傅里叶变换。

4. 计算每个道集的偏移距

对每个道集进行偏移距计算，将偏移距作为数组的第一维，道数作为数组的第二维，重新组织数组。

5. 对每个道集进行反傅里叶变换

使用NumPy的ifft函数对每个道集进行反傅里叶变换。

6. 将每个道集转换为角度域

对每个道集进行Hilbert变换，然后计算相位角度。

7. 将每个道集转换为角度道集

将每个道集按照角度值重新排列，即可得到角度道集。

### 拟合一个回归曲线来匹配振幅拾取值，这是一个角度平方的正弦函数关系.

属性A和B很少直接被利用。通常，利用的是从这两个属性中计算出其他的 AVO 属
性。

- AVO积：A*B 的剖面显示了在气藏位置的顶底都为正的反射特征
- 放大的泊松比变化: A+B 正比于泊松比的变化
- 剪切反射率: A-B 正比于剪切反射率
- 流体因子

## 正演相关代码

### Aki-Richards线性近似

$$R_{pp}=\zeta(\theta)\varDelta\ln(\alpha)+\eta(\theta)\varDelta\ln(\beta)+\xi(\theta)\varDelta\ln(\rho)$$

其中

$$\zeta(\theta)=\frac{1}{\cos ^2 \theta}, b=-8 K \sin ^2 \theta,c=1-4 K \sin ^2 \theta \\
其中 K=\frac12\left(\frac{\beta_1^2}{\alpha_1^2}+\frac{\beta_2^2}{\alpha_2^2}\right)\approx\frac{\beta^2}{\alpha^2}=\left(\frac{V_S}{V_P}\right)^2$$

```python
def createOp(self, theta):
    """

    Args:
        theta: 入射角

    Returns:
        np.array([G1, G2, G3]): aki-richards方程的系数

    """
    theta = np.deg2rad(theta)

    theta = theta[:, np.newaxis] if self.vsvp.size > 1 else theta
    vsvp = self.vsvp[:, np.newaxis].T if self.vsvp.size > 1 else self.vsvp

    G1 = 1.0 / (2.0 * cos(theta) ** 2) + 0 * vsvp
    G2 = -4.0 * vsvp ** 2 * np.sin(theta) ** 2
    G3 = 0.5 - 2.0 * vsvp ** 2 * sin(theta) ** 2
    return np.array([G1, G2, G3])

def forward(self, wav_mtx, theta, nt0, show=False, t0=None):
    """

    Args:
        wav_mtx: 子波矩阵
        theta: 入射角
        nt0: 采样点
        show: 是否显示合成后的地震数据
        t0: 采样时间

    Returns:
        cal_data: 正演得到的地震数据

    """
    # 单道正演
    if self.ntrace == 1:
        m = self.m.T.ravel()  # 注意是按行展平化，所以要先进行转置
        ntheta = len(theta)
        G = self.createOp(theta)
        # G= np.hstack([Gi for Gi in G])
        D = np.diag(np.ones(nt0 - 1), k=1) - np.diag(np.ones(nt0), k=0)  # 差分矩阵
        D[-1] = 0
        D = block_diag(*[D for i in range(3)])  # 用列表批量化diag
        G = np.vstack(np.hstack([np.diag(G_[itheta] * np.ones(nt0)) for G_ in G])
                        for itheta in range(ntheta))  # 生成可以通过矩阵乘法进行卷积的矩阵
        self.W = block_diag(*[wav_mtx for _ in range(ntheta)])  # 子波矩阵, * 表示遍历
        self.G = G @ D  # G in "WGm = d"
        self.D = D
        self.op = np.dot(self.W, np.dot(G, D))  # 得到"Ax = b"中的A
        self.obs_data = np.dot(self.op, m)  # 计算"Ax"得到合成地震数据/仿真观测数据
        self.Rpp = np.dot(np.dot(G, D), m)  # 反射系数
        cal_data = self.obs_data.reshape(ntheta, -1).T  # 同样要注意rehsape的次序，所以要先reshpe再转置而不能直接reshape成指定维度
        if show:
            plt.figure()
            plt.imshow(cal_data, cmap="gray", extent=(theta[0], theta[-1], t0[-1], t0[0]),
                        vmin=-np.abs(cal_data).max(), vmax=np.abs(cal_data).max())
            plt.axis("tight")
            plt.xlabel('incident angle')
            plt.ylabel('time')
            plt.title('seismic data from Aki-Ri')

    # 多道正演 仅仅比单道正演多了一维，需要stack
    else:
        m = np.vstack([self.m[..., i].T.ravel() for i in range(self.ntrace)])
        ntheta = len(theta)
        G = self.createOp(theta)
        # G= np.hstack([Gi for Gi in G])
        D = np.diag(np.ones(nt0 - 1), k=1) - np.diag(np.ones(nt0), k=0)
        D[-1] = 0
        D = block_diag(*[D for i in range(3)])  # 用列表批量化diag
        Garray = []
        for i in range(self.ntrace):
            Gi = np.vstack(np.hstack([np.diag(G_[i, itheta] * np.ones(nt0)) for G_ in G])
                            for itheta in range(ntheta))
            Garray.append(Gi)
        G = np.array(Garray)
        self.W = block_diag(*[wav_mtx for i in range(ntheta)])  # * 表示遍历
        self.G = np.stack([G_ @ D for G_ in G])
        self.D = D
        self.op = np.stack([self.W @ G_ for G_ in self.G])
        self.obs_data = np.stack([np.dot(self.op[i], m[i]) for i in range(self.ntrace)])
        # self.Rpp = np.dot(np.dot(G, D), m)
        cal_data = np.stack([data.reshape(ntheta, -1).T for data in
                                self.obs_data])  # 同样要注意rehsape的次序，所以要先reshpe再转置而不能直接reshape成指定维度
        if show:  # 只显示一个剖面
            plt.figure()
            plt.imshow(cal_data[1], cmap="gray", extent=(theta[0], theta[-1], t0[-1], t0[0]),
                        vmin=-np.abs(cal_data).max(), vmax=np.abs(cal_data).max())
            plt.axis("tight")
            plt.xlabel('incident angle')
            plt.ylabel('time')
            plt.title('seismic data from Aki-Ri')

    return cal_data
    
```

### Zoeppritz方程

精确Zoeppritz方程

$$
\begin{align}
\left[\begin{matrix}
R_{pp}\\
R_{ps}\\
T_{pp}\\
T_{ps}
\end{matrix}\right]
=&\left[ \begin{matrix}
\cos(\theta)&-\sin(\delta_r)&\cos(\theta_t)&\sin(\delta_t)\\
\sin(\theta)&\cos(\delta_r)&-\sin(\theta_t)&\cos(\delta_t)\\
-\cos(2\delta_r)&\frac{\beta_1}{\alpha_1}\sin(2\delta_r)&\frac{\rho_2\alpha_2}{\rho_1\alpha_1}\cos(2\delta_t)&\frac{\rho_2\beta_2}{\rho_1\alpha_1}\sin(2\delta_t)\\
\sin(2\theta)&\frac{\alpha_1}{\beta_1}\cos(2\delta_r)&\frac{\rho_2\beta_2^2\alpha_2}{\rho_1\beta_1^2\alpha_1}\sin(2\theta_t)&-\frac{\rho_2\beta_2\alpha_1}{\rho_1\beta_1\beta_1}\cos(2\delta_t)
\end{matrix}\right]^{-1}\times\left[\begin{matrix}
\cos(\theta)\\
\sin(\theta)\\
\cos(2\theta)\\
\sin(2\delta_r)
\end{matrix}\right]\notag\end{align}
$$

这里我们计算反射系数只用到$R_{pp}$

代码实现

```python
def forward(self, a1, b1, rho1, a2, b2, rho2, theta1):  # 单层的反射系数正演
    """

    Args:
        a1: vp1
        b1: vs1
        rho1:
        a2: vp2
        b2: vs2
        rho2:
        irfwav:
        ipol:
        theta1: 入射角度集

    Returns:
        当前层反射系数
        [
        ["PdPu", "SdPu", "PuPu", "SuPu"],
        ["PdSu", "SdSu", "PuSu", "SuSu"],
        ["PdPd", "SdPd", "PuPd", "SuPd"],
        ["PdSd", "SdSd", "PuSd", "SuSd"],
        ]

    """

    # Create theta1 array of angles in radiants
    theta1 = np.radians(theta1)  # 转弧度

    p = np.sin(theta1) / a1
    theta2 = np.arcsin(p * a2)  # Trans. angle of P-wave
    phi1 = np.arcsin(p * b1)  # Refl. angle of converted S-wave
    phi2 = np.arcsin(p * b2)  # Trans. angle of converted S-wave

    # Matrix form of Zoeppritz equation

    M = np.array(  # shape:[4,4,theta]
        [
            [-sin(theta1), -cos(phi1), sin(theta2), cos(phi2)],
            [cos(theta1), -sin(phi1), cos(theta2), -sin(phi2)],
            [
                2 * rho1 * b1 * sin(phi1) * cos(theta1),
                rho1 * b1 * (1 - 2 * sin(phi1) ** 2),
                2 * rho2 * b2 * sin(phi2) * cos(theta2),
                rho2 * b2 * (1 - 2 * sin(phi2) ** 2),
            ],
            [
                (-rho1) * a1 * (1 - 2 * sin(phi1) ** 2),
                rho1 * b1 * sin(2 * phi1),
                rho2 * a2 * (1 - 2 * sin(phi2) ** 2),
                (-rho2) * b2 * sin(2 * phi2),
            ],
        ],
        dtype="float",
    )
    N = np.array(  # shape:[4,1,theta]
        [
            [sin(theta1), cos(phi1), -sin(theta2), -cos(phi2)],
            [cos(theta1), -sin(phi1), cos(theta2), -sin(phi2)],
            [
                2 * rho1 * b1 * sin(phi1) * cos(theta1),
                rho1 * b1 * (1 - 2 * sin(phi1) ** 2),
                2 * rho2 * b2 * sin(phi2) * cos(theta2),
                rho2 * b2 * (1 - 2 * sin(phi2) ** 2),
            ],
            [
                (rho1) * a1 * (1 - 2 * sin(phi1) ** 2),
                -rho1 * b1 * sin(2 * phi1),
                -rho2 * a2 * (1 - 2 * sin(phi2) ** 2),
                (rho2) * b2 * sin(2 * phi2),
            ],
        ],
        dtype="float",
    )

    # Create Zoeppritz coefficient for all angles
    coef = np.zeros((4, 4, M.shape[-1]))  # [4,4,theta]
    for i in range(M.shape[-1]):  # 按角度数遍历
        Mi = M[..., i]  # 第i个角度的所有
        Ni = N[..., i]
        icoef = np.dot(np.linalg.inv(Mi), Ni)  # M^-1 点乘 N
        coef[..., i] = icoef

    return coef
def forward_mutil(self, x0, element="PdPu", show=False):  # 多层反射系数正演
    """

    Args:
        x0: 输入三参数模型
        element: str
        ["PdPu", "SdPu", "PuPu", "SuPu"],
        ["PdSu", "SdSu", "PuSu", "SuSu"],
        ["PdPd", "SdPd", "PuPd", "SuPd"],
        ["PdSd", "SdSd", "PuSd", "SuSd"],
        layer: 层数，即抽样点数

    Returns:
        所有层反射系数

    """
    theta1 = self.theta
    layer = self.layer
    vp = x0[:layer+1]
    vs = x0[layer+1:2*layer+2]
    rho = x0[2*layer+2:]
    ntheta = len(theta1)
    elements = np.array(
        [
            ["PdPu", "SdPu", "PuPu", "SuPu"],
            ["PdSu", "SdSu", "PuSu", "SuSu"],
            ["PdPd", "SdPd", "PuPd", "SuPd"],
            ["PdSd", "SdSd", "PuSd", "SuSd"],
        ]
    )
    coef = np.zeros((layer+1, ntheta))
    for i in range(layer):
        a1 = vp[i]
        a2 = vp[i + 1]
        b1 = vs[i]
        b2 = vs[i + 1]
        rho1 = rho[i]
        rho2 = rho[i + 1]

        icoef = self.forward(a1, b1, rho1, a2, b2, rho2, theta1)
        index = np.where(elements == element)
        coef[i, ...] = np.squeeze(icoef[index])
    self.ref = coef
    return coef
```

### Aki-Richards-equation非线性近似

公式见上面的[aki-richards方程](#aki-richards方程)

```python
def forward_single_layer(self, vp1, vp2, vs1, vs2, rho1, rho2, Theta_r):
    """
    Args:
        Theta_r: 入射角
    Returns:
        Rpp: 单层反射系数
    Notes:
        通过单层上下界面参数，计算单层反射系数

    """
    # 计算平均与残差
    vp = (vp1 + vp2) / 2
    vs = (vs1 + vs2) / 2
    rho = (rho1 + rho2) / 2
    dvp = vp2 - vp1
    dvs = vs2 - vs1
    drho = rho2 - rho1

    # 计算平均角
    Const = np.sin(Theta_r) / vp1  # np中的三角函数必须为弧度制！
    Theta_t = np.arcsin(Const * vp2)
    theta = (Theta_r + Theta_t) / 2

    # Aki-Richards方程的系数
    G1 = (0.5 * ((1 / np.cos(theta)) ** 2))
    G2 = -(4 * (vs / vp * np.sin(theta)) ** 2)
    G3 = 0.5 * (1 - 4 * (vs / vp * np.sin(theta)) ** 2)

    G = np.stack((G1, G2, G3)).T
    m = np.array([dvp / vp, dvs / vs, drho / rho])

    Rpp = G @ m
    return Rpp

def forward(self, wav_mtx, theta, nt0, show=False, t0=None):  # 目前只做单道
    """

    Args:
        wav_mtx: 子波矩阵
        theta: 入射角集
        nt0: 采样点
        show: 是否显示合成地震数据
        t0: 采样时间，用来限定横坐标

    Returns:
        Rpp: 整道反射系数

    """
    thetar = np.radians(theta)  # 转弧度
    Rpp = []
    for i in range(self.layer):
        a1 = self.vp[i]
        a2 = self.vp[i + 1]
        b1 = self.vs[i]
        b2 = self.vs[i + 1]
        rho1 = self.rho[i]
        rho2 = self.rho[i + 1]
        iRpp = self.forward_single_layer(a1, a2, b1, b2, rho1, rho2, thetar)
        Rpp.append(iRpp)

    Rpp = np.array(Rpp)
    Rpp = np.vstack((Rpp, np.zeros(Rpp.shape[-1])))  # 添加全零行，行数补到和采样点数一样
    cal_data = wav_mtx@Rpp
    if show:
        plt.figure()
        plt.imshow(cal_data, cmap="gray", extent=(theta[0], theta[-1], t0[-1], t0[0]),
                    vmin=-np.abs(cal_data).max(), vmax=np.abs(cal_data).max())
        plt.axis("tight")
        plt.xlabel('incident angle')
        plt.ylabel('time')
        plt.title('seismic data from Aki-Ri-Nonliner')

    return Rpp
```

偏导数求解为

$$
\frac{\partial R}{\partial\alpha_i} = \frac{8\sin^2(\theta) (\rho_{i+1} - \rho_{i}) (\beta_{i} + \beta_{i+1})^2}{((\rho_{i} + \rho_{i+1})(\alpha_{i} + \alpha_{i+1})^3)}+\frac{16\sin^2(\theta)(\beta_{i+1}^2 - \beta_{i}^2)}{(\alpha_{i}+ \alpha_{i+1})^3} -\frac{2\alpha_{i+1}} {(\alpha_{i}+ \alpha_{i+1})^2\cos^2(\theta)}
$$

$$
\frac{\partial R}{\partial\beta_i}=\frac{-8\sin^2(\theta)(\rho_{i+1}-\rho_i)(\beta_i+\beta_{i+1})}{(\rho_i+\rho_{i+1})(\alpha_i+\alpha_{i+1})^2}+\frac{16\sin^2(\theta)\beta_i}{(\alpha_i+\alpha_{i+1})^2}
$$




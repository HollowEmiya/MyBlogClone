---
title: Water And FFT 1
math: true
tags: [Math,Graphics]
index_img: 
banner_img: 
data: 2025--12--12 22:09:48
typora-root-url: ./..
---

快速傅里叶变换渲染水第一部分  
关于傅里叶变换
<!--more-->

# Water And FFT 1

## 傅里叶级数和傅里叶变换

傅里叶级数的本质是将一个周期的信号分解为无限多离散的正弦波。

傅里叶级数是，在时域是一个周期且连续的函数，而在频域是一个非周期并离散的函数。  
傅里叶变换是，将一个时域非周期的连续信号，转换为在频域非周期的连续信号。  
![](/imgs/WaterAndFFT/FSandFT.jpg)  
上面是时域周期且连续的信号，经过傅里叶级数变成在频域非周期并离散的函数。  
下面是时域一非周期的连续信号，经傅里叶变换，变成在频域非周期并连续的信号。

## 傅里叶变换公式

从时域(或者称之为空域)的信号 $f(x)$ 求频谱 $F(\omega)$ 即傅里叶变换。

我们在计算机上处理离散数据，所以关注离散傅里叶变换(Discrete Fourier Transform)。

设，有长度为 $N$ 的数字序列，则通过离散傅里叶变换得到的频域函数为：
$$
\begin{aligned}
&F(\mu)=\sum_{x=0}^{N-1}f(x)e^{-i\frac{2\pi\mu x}{N}}\\
&F(\mu):转换后的频域函数\\
&\mu:频率\\
&f(x):时域的函数(数字序列)\\
&e^{-i\frac{2\pi\mu x}{N}}:一个复数
\end{aligned}
$$

根据欧拉公式：
$$
F(\mu)=\sum_{x=0}^{N-1}f(x)\cos(\frac{2\pi\mu x}{N})-i\sin(\frac{2\pi\mu x}{N})
$$

### 三角函数的正交性

模拟周期函数的傅里叶级数所用的正交函数系：
$$
1,\cos\omega t,\sin\omega t,\cos2\omega t,\sin2\omega t,...,\cos n\omega t,\sin n\omega t,...
$$
其中任意两个不同的函数之积，在他们的公共周期内的积分等于 0。

## 海面和傅里叶逆变换

将水面波浪高度视为在空域的信号 $h(x,z)$，此信号天然接近大量正弦信号叠加。  
如果知道其频谱 $\widetilde{h}(\omega_x,\omega_z)$，使用傅里叶逆变换就可以求出海面高度 $h(x,z)$。

### 海面的 IDFT

$$
\begin{aligned}
&h(\vec{x},t)=\sum\nolimits_\vec{k}\widetilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}\\\\
&\vec{x}=(x,z),空域坐标。\\
&\vec{k}=(k_x,k_z),频域坐标,k_z,k_x均为频率，相当于前面的\omega_x,\omega_z\\
&\widetilde{h}(\vec{k},t),频谱。\\
&多出的\widetilde{h}(\vec{k},t)参数t表示频谱会随时间变化，而高度也会随之变化。\\
&e^{i\vec{k}\cdot\vec{x}}=e^{i(k_xx+k_zz)}
\end{aligned}
$$

求合是对所有频域坐标点 $\vec{k}$ 进行。  
$\vec{k}$ 在频域平面上以原点为中心，每隔 $\frac{2\pi}{L}$ 取一个点，$L$ 为海面 patch 尺寸，共 $N\times N$ 个点：
$$
k_x=\frac{2\pi n}{L},n\in\{-\frac{N}{2},-\frac{N}{2}+1,...,\frac{N}{2}-1\}\\
k_z=\frac{2\pi m}{L},m\in\{-\frac{N}{2},-\frac{N}{2}+1,...,\frac{N}{2}-1\}\\
$$
$\vec{x}$ 在 $xz$ 平面上以原点为中心，每隔 $\frac{L}{N}$ 取一个点，也是 $N\times N$ 个点：
$$
x=\frac{uL}{N},u\in\{-\frac{N}{2},-\frac{N}{2}+1,...,\frac{N}{2}-1\}\\
z=\frac{vL}{N},v\in\{-\frac{N}{2},-\frac{N}{2}+1,...,\frac{N}{2}-1\}\\
$$
在 $N$ 为偶数的情况下，所有频率值 $k$ 对应的周期 $T=\frac{2\pi}{k}$ 均是 $L$ 的约数，  
所以 $h(\vec{x},t)=\sum_\vec{k}\widetilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}$ 横向和纵向的均以 $L$ 为周期，则水面 patch 可以无缝 tiling。

### 菲利普频谱(Phillips spectrum)

频谱 $\widetilde{h}(\vec{k},t)$，通常采用菲利普频谱：
$$
\begin{aligned}
&\widetilde{h}(\vec{k},t)=\widetilde{h}_0(\vec{k})e^{i\omega(k)t}+\widetilde{h}_0^*(-\vec{k})e^{-i\omega(k)t}
\end{aligned}
$$
$\widetilde{h}_0^*$ 表示 $\widetilde{h}_0$ 的共轭复数，$k$ 表示 $\vec{k}$ 的模。
$$
\omega(k)=\sqrt{gk}
$$
$g$ 是重力加速度。
$$
\widetilde{h}_0(\vec{k})=\frac{1}{\sqrt{2}}(\xi_r+i\xi_i)\sqrt{P_h(\vec{k})}
$$
其中 $\xi_r,\xi_i$ 是相互独立的随机数，均服从均值为 0，标准差为 1 的正态分布。
$$
P_h(\vec{k})=A\frac{e^{\frac{-1}{(kL)^2}}}{k^4}|\vec{k}\cdot\vec{\omega}|
$$
$\vec{\omega}$ 是风向，这里的 $L$ 不是海面 patch 尺寸 $L$，$L=\frac{V^2}{g}$，$V$ 是风速。

### 法线

**这里原文计算的就不对。**

经过 IDFT 得到海面高度后，可以使用差分方法计算法线，但是这样求解不精确。  
最精确的方式是求解法线的解析式。

因有高度：
$$
h(\vec{x},t)=\sum\nolimits_{\vec{k}}\widetilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}
$$

#### 计算法线

如果想计算法线，对于曲面有 $F(x,y,z)=h_x(x,t)+h_z(z,t)-y$，求其梯度就是法线……
$$
\vec{N}=normalize(
-\frac{\partial h(\vec{x},t)}{\partial{x}},1,
-\frac{\partial h(\vec{x},t)}{\partial{z}})
$$

### 尖浪(Choppy Waves)

如果只是根据频谱计算高度，表现就是面重复播放周期函数。  
在 Gerstner Wave 中通过移动 x,z 来表示波在水平面的运动，见下图位置 P 中x,y 变换  
![](/imgs/WaterAndFFT/013equ01.jpg)  
同样的操作我们也可以用在 IDFT 海面：
$$
\begin{aligned}
&For\;a\;point\;pos\;P,\\
&P(\vec{x},t)=(\vec{x}+\lambda\vec{D}(\vec{x},t),h(\vec{x},t))\\
&\vec{D}(\vec{x},t)=\sum\nolimits_\vec{k}
-i\frac{\vec{k}}{k}\widetilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}\\
&\vec{x}'=\vec{x}+\lambda\vec{D}(\vec{x},t)
\end{aligned}
$$

$$
\begin{aligned}
-i\frac{\vec{k}}{k}\widetilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}
&=-i\frac{\vec{k}}{k}\widetilde{h}(\vec{k},t)*
(\cos(\vec{k}\cdot\vec{x})+i\sin(\vec{k}\cdot\vec{x}))\\
&=-i\frac{\vec{k}}{k}\widetilde{h}(\vec{k},t)\cos(\vec{k}\cdot\vec{x})-
\frac{\vec{k}}{k}\widetilde{h}(\vec{k},t)\sin(\vec{k}\cdot\vec{x})\\
e^{i\theta}&=\cos\theta+i\sin\theta\\
ie^{i\theta}&=i\cos\theta-\sin\theta\\
&=\sin(\theta+\pi)+i\sin(\theta+\frac{\pi}{2})\\
&=\cos(\theta+\frac{\pi}{2})+i\sin(\theta+\frac{\pi}{2})
\end{aligned}
$$

所以对 IDFT 海面的挤压和 Gerstner Wave 类似，也是在坐标加上波动。  
暂时先这样理解，并非主要部分。  
不过和 Gerstner Wave 一样当挤压过大时，面会产生穿刺。  
![](/imgs/WaterAndFFT/dACross.jpg)  
当发生穿刺时，局部反转，在数学上就是，面元有向面积变为负值。(这是怎么联想到的，真厉害)  
因为 $x',z'$ 都是以 $x,z$ 为变元的二元函数，则有 $x'=x'(x,z),z'=z'(x,z)$，根据二重积分换元法得变换后面积元为：
$$
\begin{aligned}
\mathrm{d}A=\vec{\mathrm{d}x'}\times\vec{\mathrm{d}z'}=
\begin{vmatrix}
\frac{\partial x'}{\partial{x}} \; \frac{\partial x'}{\partial{z}}\\
\frac{\partial x'}{\partial{z}} \; \frac{\partial z'}{\partial{z}}
\end{vmatrix}\mathrm{d}x\mathrm{d}z
\end{aligned}
$$
由于 $\mathrm{d}x\mathrm{d}z$ 必定为正，则 $\mathrm{d}A$ 的正负则取决于 雅可比行列式：
$$
\begin{aligned}
J(\vec{x})&=\begin{vmatrix}
J_{xx} \; J_{xz}\\
J_{zx} \; J_{zz}
\end{vmatrix}=\begin{vmatrix}
\frac{\partial x'}{\partial{x}} \; \frac{\partial x'}{\partial{z}}\\
\frac{\partial x'}{\partial{z}} \; \frac{\partial z'}{\partial{z}}
\end{vmatrix}\\
\vec{x}'&=\vec{x}+\lambda\vec{D}(\vec{x},t)\\
J_{xx}&=\frac{\partial x'}{\partial{x}}=
1+\lambda\frac{\partial{D_x(\vec{x},t)}}{\partial{x}}\\
J_{zz}&=\frac{\partial z'}{\partial{z}}=
1+\lambda\frac{\partial{D_z(\vec{x},t)}}{\partial{z}}\\
J_{xz}&=\frac{\partial{x'}}{\partial{z}}=
\lambda\frac{\partial{D_x}(\vec{x},t)}{\partial{z}}\\
J_{zx}&=\frac{\partial{z'}}{\partial{x}}=
\lambda\frac{\partial{D_z}(\vec{x},t)}{\partial{x}}\\
\vec{D}(\vec{x},t)&=\sum\nolimits_\vec{k}
-i\frac{\vec{k}}{k}\widetilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}\\
D_{x}(\vec{x},t)&=\sum\nolimits_\vec{k}-i\frac{k_x}{k}
\widetilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}\\
J_{xz}&=\frac{\partial{D_x}(\vec{x},t)}{\partial{z}}=
\sum\nolimits_\vec{k}-i\frac{k_x}{k}
\widetilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}ik_z\\
&=\sum\nolimits_\vec{k}\frac{k_x}{k}
\widetilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}k_z=
\sum\nolimits_\vec{k}\frac{k_xk_z}{k}
\widetilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}\\
J_{zx}&=\sum\nolimits_\vec{k}\frac{k_xk_z}{k}
\widetilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}
\end{aligned}
$$


## Reference

[【技术美术】GerstnerWave原理详解(文末源码) - 知乎](https://zhuanlan.zhihu.com/p/623569022)

[[学习笔记]Unity 基于GPU FFT海洋的实现-理论篇 - 知乎](https://zhuanlan.zhihu.com/p/95482541)  
[[学习笔记]Unity 基于GPU FFT海洋的实现-实践篇 - 知乎](https://zhuanlan.zhihu.com/p/96811613)  

[FFT海洋学习笔记（一） - 知乎](https://zhuanlan.zhihu.com/p/335045713)  
[FFT海洋水体渲染学习笔记（二） - 知乎](https://zhuanlan.zhihu.com/p/335946333)

[生成服从标准正态分布的随机数 - 知乎](https://zhuanlan.zhihu.com/p/67776340)  
[fft海面模拟(一) - 知乎](https://zhuanlan.zhihu.com/p/64414956)

[傅里叶分析之掐死教程（完整版）更新于2014.06.06 - 知乎](https://zhuanlan.zhihu.com/p/19763358)  
Stockham 算法: [详尽的快速傅里叶变换推导与Unity中的实现 - 知乎](https://zhuanlan.zhihu.com/p/208511211)

[超强换元法，二重积分计算的核武器！（雅可比行列式超通俗讲解） - 知乎](https://zhuanlan.zhihu.com/p/382301310)
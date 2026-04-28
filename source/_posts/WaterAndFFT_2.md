---
title: Water And FFT 2
math: true
tags: [Math,Graphics]
index_img: 
banner_img: 
data: 2026--01--11 16:22:32
typora-root-url: ./..
---

快速傅里叶变换渲染水第二部分  
快速傅里叶变换和蝶形网络
<!--more-->

# Water And FFT 2

## 递归形式的 FFT 算法及复杂度

标准 DFT：
$$
\begin{aligned}
X(k)=\sum\nolimits^{N-1}_{n=0}x(n)e^{-i\frac{2\pi k}{N}n},k\in{0,1,...,N-1}
\end{aligned}
$$
为了书写方便，常有：$W_N^k=e^{-i\frac{2\pi k}{N}}$。  
因 求合和 $k$ 的取值，所以可以看作为 N 个输入以及 N 个输出的电器元件(N point DFT calculator)：  
![](/imgs/WaterAndFFT/N_point_DFT_Calculator.jpg)  
如果直接暴力计算，就是 $O(N*N)$ 复杂度爆炸了。

**FFT 是利用分治思想对 DFT 进行计算，降低复杂度，FFT 只用于计算 N 为 $2^m$ 数量(2的幂)的计算。**  
因为是不断的二分过程计算的，所以 N 为 2 的幂才行。

考虑用两个 N/2 point DFT calculator 构造一个 N point DFT calculator (N 为 2 的幂)。  
如果将原序号为偶数的给第一个 N/2 point DFT calculator，序号奇数的给第二个，如下图：  
![](/imgs/WaterAndFFT/Nf2_point_DFT_Calculator.jpg)  
可以得到：
$$
\begin{aligned}
G(k_1)&=\sum\nolimits^{\frac{N}{2}-1}_{n=0}
g(n)e^{-i\frac{2\pi k}{\frac{N}{2}}n}=
\sum\nolimits^{\frac{N}{2}-1}_{n=0}
x(2n)e^{-i\frac{2\pi k}{\frac{N}{2}}n}&k_1\in\{0,1,...,\frac{N}{2}-1\}\\
H(k_2)&=\sum\nolimits^{\frac{N}{2}-1}_{n=0}
h(n)e^{-i\frac{2\pi k}{\frac{N}{2}}n}=
\sum\nolimits^{\frac{N}{2}-1}_{n=0}
x(2n+1)e^{-i\frac{2\pi k}{\frac{N}{2}}n}&k_2\in\{0,1,...,\frac{N}{2}-1\}
\end{aligned}
$$
这里 $-i\frac{2\pi k}{\frac{N}{2}}n$ 的 $\frac{n}{\frac{N}{2}}$ 这样是只和求合的取值范围有关，表示 $[0,1)$ 范围的“离散取样”。  
用 $G(k),H(x)$  计算 $X(k)$
$$
\begin{aligned}
X(k)&=
\left\{
	\begin{aligned}
	&G(k)+W_N^kH(k)
	&k&\in\{0,1,...,\frac{N}{2}-1\}\\
	&G(k-\frac{N}{2})+W_N^kH(k-\frac{N}{2})
	&k&\in\{\frac{N}{2},\frac{N}{2}+1,...,N-1\}
	\end{aligned}
\right.
\end{aligned}
$$

推导过程：
$$
\begin{aligned}
when&\;k\in\{0,1,...,\frac{N}{2}-1\},\\

X(k)&=\sum\nolimits_{n=0}^{N-1}x(n)e^{-i\frac{2\pi kn}{N}}\\

&=\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n)e^{-i\frac{2\pi k(2n)}{N}}+
\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n+1)e^{-i\frac{2\pi k(2n+1)}{N}}\\

&=\sum\nolimits_{n=0}^{\frac{N}{2}-1}
x(2n)e^{-i\frac{2\pi kn}{\frac{N}{2}}}+
e^{-i\frac{2\pi kn}{N}}\sum\nolimits_{n=0}^{\frac{N}{2}-1}
x(2n+1)e^{ -i\frac{2\pi kn}{\frac{N}{2} } }\\
&=G(k)+W_N^kH(k)\\
\end{aligned}
$$

$$
\begin{aligned}
when&\;k\in\{\frac{N}{2},\frac{N}{2}+1,...,N-1\},set\;K=k-\frac{N}{2}\\
&K\in\{0,1,...,\frac{N}{2}-1\},e^{i\pi}=-1\\
e^{-i\pi 2}&=1,e^{-i\pi 2n}=1^n=1\\
X(k)&=\sum\nolimits_{n=0}^{N-1}x(n)e^{-i\frac{2\pi kn}{N}}\\

&=\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n)e^{-i\frac{2\pi k(2n)}{N}}+
\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n+1)e^{-i\frac{2\pi k(2n+1)}{N}}\\

&=\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n)
e^{-i\frac{2\pi (K+\frac{N}{2})(2n)}{N}}+
\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n+1)
e^{-i\frac{2\pi (K+\frac{N}{2})(2n+1)}{N}}\\

&=\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n)
e^{-i\frac{2\pi K(2n)}{N}-i\pi 2n}+
\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n+1)
e^{-i\frac{2\pi K(2n+1)}{N}-i\pi(2n+1)}\\

&=\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n)
e^{-i\frac{2\pi K(2n)}{N}}-
\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n+1)
e^{-i\frac{2\pi K(2n+1)}{N}}\\

&=\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n)
e^{-i\frac{2\pi Kn}{\frac{N}{2} }}-
e^{-i\frac{2\pi K}{N}}
\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n+1)
e^{-i\frac{2\pi Kn}{\frac{N}{2} }}\\

&=\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n)
e^{-i\frac{2\pi Kn}{\frac{N}{2} }}-
e^{-i\frac{2\pi k}{N}+i\pi}
\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n+1)
e^{-i\frac{2\pi Kn}{\frac{N}{2} }}\\

&=\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n)
e^{-i\frac{2\pi Kn}{\frac{N}{2} }}+
e^{-i\frac{2\pi k}{N}}
\sum\nolimits_{n=0}^{\frac{N}{2}-1}x(2n+1)
e^{-i\frac{2\pi Kn}{\frac{N}{2} }}\\
&=G(K)+W_N^kH(K)\\
&=G(k-\frac{N}{2})+W_N^kH(k-\frac{N}{2})
\end{aligned}
$$

补全的电路图：  
![](/imgs/WaterAndFFT/Nf2_point_DFT_Calculator_2.jpg)  
图中左侧的 $x[n]$ 对应的是 $\sum\nolimits_{n=0}^{N-1}x(n)e^{-i\frac{2\pi kn}{N}}$ 的 $x(n)$，  
中间 $g[n],h[n]$ 对应 $\sum\nolimits^{\frac{N}{2}-1}_{n=0}
g(n)e^{-i\frac{2\pi k}{\frac{N}{2}}n},\sum\nolimits^{\frac{N}{2}-1}_{n=0}
h(n)e^{-i\frac{2\pi k}{\frac{N}{2}}n}$ 的 $g(n),h(n)$  
右侧的 $G[n],H[n],X[n]$ 就对应 $X(k)=G(k)+W_N^kH(k)$ 的几项。

以上为递归形式的 FFT，但是不适合 GPU 计算，应采用展平的蝶形网络。  
无论是递归还是蝶形，复杂度是相同的。  
**递归在哪里？**每一个 N/2 point DFT calculator 都是一个 N point DFT calculator。  
也就是上面的 G(k),H(k) 其实都可以接着递归写成和 X(k) 一样的格式，也就是递归的。

复杂度计算，设上图 N point DFT calculator 的乘法次数为 $C(N)$，  
则其内两个 N/2 point DFT calculator 的乘法次数为 $C(N/2)$，  
而根据 G(k),H(k) 计算 X(k) 需要 N 次乘法( $W_N^kH(k),W_N^kH(k-\frac{N}{2})$ )，能够得到：
$$
\begin{aligned}
C(N)&=2*C(N/2)+N\\
Set\;N&=2^m\\
There\;is&\;\\
D(0)&=C(1)=1\\
D(m)&=2D(m-1)+2^m\\
D(m-1)&=2D(m-2)+2^{m-1}\\
...\\
D(2)&=2D(1)+2^2\\
D(1)&=2D(0)+2^1\\
D(m)&=2^mD(0)+m*2^m=(m+1)2^m\\
C(N)&=N*(\log_2N+1)
\end{aligned}
$$

## 蝶形网络(Butterfly Diagram)

先来看一个 4 point DFT 的完全展开，  
![](/imgs/WaterAndFFT/ButterflyDiagram4Point.jpg)  
calculator 中间的 $W_2^0,W_2^1$ 是和 $g[x],h[x]$ 相乘计算 $G[x],H[x]$ 的。  
简化：  
![](/imgs/WaterAndFFT/ButterflyDiagram4PointSimple.jpg)
$$
\begin{aligned}
W_N^k&=e^{-i\frac{2\pi k}{N}}\\
W_N^{k+\frac{N}{2}}&=e^{-i\frac{2\pi k}{N}-i\pi}=-e^{-i\frac{2\pi k}{N}}\\
W_N^{k+\frac{N}{2}}&=W_N^k
\end{aligned}
$$
将 $W_4^2,W_4^3$ 换为 $-W_4^0,-W_4^1$，得：  
![](/imgs/WaterAndFFT/ButterflyDiagram4PointSimple2.jpg)

对于给定的 point 数，蝶形网络是固定的，可以通过预计算处理。  
![](/imgs/WaterAndFFT/ButterflyDiagram8Point.jpg)  
蝶形网络还给出了 stage，蝶形网络计算 FFT 是按照 stage 推进的：  
N 个输入经过第一个 stage 计算得到 N 个中间结果，再输入第二个 stage……直到得到最终 N 个输出。N point FFT 有 $\log_2N$ 个 stage。

## bitreveres 算法

蝶形网络的 N 个输入的顺序是乱序的，如上述 8 point 蝶形网络。  
对于一般的 N point 情况，我们可以使用 bitreverse 算法计算该顺序。

对于 N point 蝶形网络，求 $x(k)$ 在几号位置，只需要将 $k$ 先转化为 $\log_2N$ 位的二进制数，  
然后将 bit 反序，再转回十进制，即可求得 $x(k)$ 所在位号。  
以 8 point 蝶形网络为例子，求 $x(3)$ 在几号位置，将 3 转化为 3 ($\log_28=3$) 位二进制数——011，  
反序得到 110，将 110 转为十进制得到 6，即 $x(3)$ 在 6 号位置。

> 推导过程就看原文吧，就不搬运了。

## IFFT

但是海面模型是 IDFT 而非 DFT。  
DFT：
$$
\begin{aligned}
X(k)&=\sum\nolimits_{n=0}^{N-1}x(n)e^{-i\frac{2\pi kn}{N}},k\in\{0,1,...,N-1\}
\end{aligned}
$$
IDFT:
$$
\begin{aligned}
x(n)=\frac{1}{N}\sum\nolimits_{k=0}^{N-1}X(k)e^{i\frac{2\pi kn}{N}},n\in\{0,1,...,N-1\}
\end{aligned}
$$
两者表达式相似但是无法直接套用，按照相同的思路重新推到可以得到 IFFT 算法：

用两个 N/2 point IDFT calculator 去构造 N point IDFT calculator。  
将序号为偶数的输入给第一个 N/2 point IDFT calculator，  
序号为奇数的输入给第二个 N/2 point IDFT calculator:  
![](/imgs/WaterAndFFT/Nf2_point_IDFT_Calculator.jpg)  
则有:
$$
\begin{aligned}
G(n)&=\frac{1}{N}\sum\nolimits_{k=0}^{\frac{N}{2}-1}g(k)
e^{i\frac{2\pi kn}{\frac{N}{2}}}=
\frac{1}{N}\sum\nolimits_{k=0}^{\frac{N}{2}-1}x(2k)
e^{i\frac{2\pi kn}{\frac{N}{2}}},n\in\{0,1,...,\frac{N}{2}-1\}\\
H(n)&=\frac{1}{N}\sum\nolimits_{k=0}^{\frac{N}{2}-1}h(k)
e^{i\frac{2\pi kn}{\frac{N}{2}}}=
\frac{1}{N}\sum\nolimits_{k=0}^{\frac{N}{2}-1}x(2k+1)
e^{i\frac{2\pi kn}{\frac{N}{2}}},n\in\{0,1,...,\frac{N}{2}-1\}
\end{aligned}
$$
根据前面类似的推导得到：
$$
\begin{aligned}
x(n)=\left\{
	\begin{aligned}
	&G(n)+W_N^{-n}H(n)&&n\in\{0,1,...,\frac{N}{2}-1\}\\
	&G(n-\frac{N}{2})+W_N^{-n}H(n-\frac{N}{2})
	&&n\in\{\frac{N}{2},\frac{N}{2}+1,...,N-1\}
	\end{aligned}
\right.
\end{aligned}
$$
补全电路图如下:  
![](/imgs/WaterAndFFT/Nf2_point_IDFT_Calculator_2.jpg)  
和前文类似，针对 $N=4$，对电路展开，得到 4 point IFFT 蝶形网络:  
![](/imgs/WaterAndFFT/ButterflyInverseDiagram4PointSimple.jpg)  
利用 $W_N^{-n-\frac{n}{2}}=-W_N^{-n}$ 能够得到第二张形式:  
![](/imgs/WaterAndFFT/ButterflyInverseDiagram4PointSimple2.jpg)  
IFFT 的 bitreverse 与 FFT 相同。  
由于 DFT/IDFT 是线性的，所以常数因子不会影响算法。  
故而适用于标准的 IDFT: $x(n)=\frac{1}{N}\sum\nolimits_{k=0}^{N-1}X(k)e^{i\frac{2\pi kn}{N}},n\in\{0,1,...N-1\}$  
的 IFFT 算法，可以不添加任何修改地应用于未归一化的 IDFT:
$$
\begin{aligned}
x(n)=\sum\nolimits_{k=0}^{N-1}X(k)e^{i\frac{2\pi kn}{N}},n\in\{0,1,...,N-1\}
\end{aligned}
$$
海面的 IDFT 更接近后者。

## Refrence

[fft海面模拟（二） - 知乎](https://zhuanlan.zhihu.com/p/64726720)
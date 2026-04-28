---
title: Water And FFT 2
math: true
tags: [Math,Graphics]
index_img: 
banner_img: 
data: 2026--01--20 10:34:26
typora-root-url: ./..
---

快速傅里叶变换渲染水第三部分  
海洋渲染和傅里叶变换
<!--more-->

# Water And FFT 3

将海洋 IDFT 模型  
$h(\vec{x},t)=\sum\nolimits_\vec{k}\widetilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}$  
写为标量形式:
$$
\begin{aligned}
h(x,z,t)=\sum\nolimits_{m=-\frac{N}{2}}^{\frac{N}{2}-1}
\sum\nolimits_{n=-\frac{N}{2}}^{\frac{N}{2}-1}
\widetilde{h}(k_x,k_z,t)e^{i(k_xx+k_zz)}
\end{aligned}
$$
将 $k_x,k_z$ 展开：
$$
\begin{aligned}
k_x&=\frac{2\pi n}{L},
n\in\{-\frac{N}{2},-\frac{N}{2}+1,...,\frac{N}{2}-1\}\\
k_z&=\frac{2\pi m}{L},
m\in\{-\frac{N}{2},-\frac{N}{2}+1,...,\frac{N}{2}-1\}
\end{aligned}
$$
即：
$$
\begin{aligned}
h(x,z,t)=\sum\nolimits_{m=-\frac{N}{2}}^{\frac{N}{2}-1}
\sum\nolimits_{n=-\frac{N}{2}}^{\frac{N}{2}-1}
\widetilde{h}(\frac{2\pi n}{L},\frac{2\pi m}{L},t)
e^{i(\frac{2\pi n}{L}x+\frac{2\pi m}{L}z)}
\end{aligned}
$$
为使下标从零开始，令 $n'=n+\frac{N}{2},m'=m+\frac{N}{2}$，则 $n',m'\in\{0,1,...,N-1\}$，得：
$$
\begin{aligned}
h(x,z,t)=\sum\nolimits_{n'=0}^{N-1}\sum\nolimits_{m'=0}^{N-1}
\widetilde{h}(\frac{2\pi(n'-\frac{N}{2})}{L},
\frac{2\pi(m'-\frac{N}{2})}{L},t)
e^{i(\frac{2\pi(n'-\frac{N}{2})}{L}x+
\frac{2\pi(m'-\frac{N}{2})}{L}z)}
\end{aligned}
$$
令 $\widetilde{h}'(n',m',t)=\widetilde{h}(\frac{2\pi(n'-\frac{N}{2})}{L},\frac{2\pi(m'-\frac{N}{2})}{L},t)$ 并将 $e^{i\frac{2\pi(n'-\frac{N}{2})}{L}x}$ 从内部求合提到外面的求合：
$$
\begin{aligned}
h(x,z,t)=\sum\nolimits_{n'=0}^{N-1}e^{i\frac{2\pi(n'-\frac{N}{2})}{L}x}
\sum\nolimits_{m'=0}^{N-1}
\widetilde{h}'(n',m',t)
e^{i(\frac{2\pi(m'-\frac{N}{2})}{L}z)}
\end{aligned}
$$
我们拆解上式:
$$
\begin{aligned}
\widetilde{h}''(n',z,t)&=\sum\nolimits_{m'=0}^{N-1}\widetilde{h}'(n',m',t)
e^{i\frac{2\pi(m'-\frac{N}{2})}{L}z}\\
h(x,z,t)&=\sum\nolimits_{n'=0}^{N-1}\widetilde{h}''(n',z,t)
e^{i\frac{2\pi(n'-\frac{N}{2})}{L}x}
\end{aligned}
$$
由于长度 $L$ 任意取值，为向 IDFT 靠拢，取 $L=N$:
$$
\begin{aligned}
h(x,z,t)&=(-1)^x\sum\nolimits_{n=0}^{N-1}
\widetilde{h}''(n',z,t)e^{i\frac{2\pi n'x}{N}}
\end{aligned}\tag{a}
$$

$$
\begin{aligned}
\widetilde{h}''(n',z,t)=(-1)^z\sum\nolimits_{m'=0}^{N-1}
\widetilde{h}(n',m',t)e^{i\frac{2\pi m'z}{N}}
\end{aligned}\tag{b}
$$

将 $x,z$ 展开:
$$
\begin{aligned}
&x=\frac{u L}{N},u\in\{-\frac{N}{2},-\frac{N}{2}+1,...,\frac{N}{2}-1\}\\
&z=\frac{v L}{N},v\in\{-\frac{N}{2},-\frac{N}{2}+1,...,\frac{N}{2}-1\}
\end{aligned}
$$
因为前面取 $L=N$，且为了下标从零开始，设 $u'=u+\frac{N}{2},v'=v+\frac{N}{2}$，能得到:
$$
\begin{aligned}
&x=u'-\frac{N}{2},u'\in\{0,1,...,N-1\}\\
&z=v'-\frac{N}{2},v'\in\{0,1,...,N-1\}
\end{aligned}
$$
带入 $a,b$ 式得到:
$$
\begin{aligned}
h(u'-\frac{N}{2},v'-\frac{N}{2},t)&=
(-1)^{u'-\frac{N}{2}}\sum_\nolimits{n'=0}^{N-1}
\widetilde{h}''(n',v'-\frac{N}{2},t)
e^{i\frac{2\pi n'u'}{N}}(-1)^{n'}
\end{aligned}\tag{c}
$$

$$
\begin{aligned}
\widetilde{h}''(n',v'-\frac{N}{2},t)=(-1)^{v'-\frac{N}
{2}}\sum\nolimits_{m'=0}^{N-1}\widetilde{h}(n',m',t)
e^{i\frac{2\pi m'v'}{N}}(-1)^{m'}
\end{aligned}\tag{d}
$$

令：
$$
\begin{aligned}
A(u',v',t)&=h(u'-\frac{N}{2},v'-\frac{N}{2},t)/(-1)^{u'-\frac{N}{2}}\\
B(n',v',t)&=\widetilde{h}''(n',v'-\frac{N}{2},t)(-1)^{n'}\\
C(n',v',t)&=\widetilde{h}''(n',v'-\frac{N}{2},t)/(-1)^{v'-\frac{N}{2}}\\
D(n',m',t)&=\widetilde{h}(n',m',t)(-1)^{m'}
\end{aligned}
$$
则 $c,d$ 可化为：
$$
\begin{aligned}
A(u',v',t)&=\sum\nolimits_{n'=0}^{N-1}B(n',v',t)e^{\frac{2\pi n'u'}{N}}
\end{aligned}\tag{1}
$$

$$
\begin{aligned}
C(n',v',t)&=\sum\nolimits_{m'=0}^{N-1}D(n',m',t)e^{\frac{2\pi m'v'}{N}}
\end{aligned}\tag{2}
$$

现在可以套用 IFFT。

## Refrence

[fft海面模拟（三） - 知乎](https://zhuanlan.zhihu.com/p/65156063)
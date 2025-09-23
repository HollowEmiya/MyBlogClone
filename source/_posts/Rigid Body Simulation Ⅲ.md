---
title: Rigid Body Simulation Ⅲ
math: true
tags: [Math]
index_img: 
banner_img: 
data: 2025--09--04 22:09:48
typora-root-url: ./..
---

刚体模拟附录部分。  
<!--more-->

# Rigid Body Simulation Ⅲ

## Appendix

### Appendix A Motion Equation Derivations

在这部分，我们将补充在第2节中缺失的内容，关于方程 $\dot{P}(t)=F(t),\dot{L}(t)=\tau(t),L(t)=I(t)\omega(t)$。这里的推导方法并非标准方法，是由 Andy Witkin 提出的。本附录推导(原作者认为)比传统推导来自 Goldstein[^10]的简洁。  
用 $F_i(t)$ 来描述作用在刚体上的外力，这里 $F_i(t)$ 指的是作用在第 i 个粒子上的外力。然而，对于一个钢铁而言，要保持刚体的形状，就必须有一些“内部”的约束作用作用在同一物体的粒子之间。我们假设这些约束力被动地作用于系统，并且不会对任何网格作用。设 $F_{ci}(t)$ 表示作用于第 i 个粒子的净内力。在 $t_0\rightarrow t_1$ 间 $F_{ci}$ 对第 i 个粒子所做的功为：  
$$
\int_{t_0}^{t_1}F_{ci}(t)\cdot\dot{r}_i(t)\mathrm{d}t
$$
这里 $\dot{r}_i(t)$ 是第 i 个粒子的速度。所有粒子的净做功是总和：  
$$
\sum_i\int_{t_0}^{t_1}F_{ci}(t)\cdot\dot{r}_i(t)\mathrm{d}t=
\int_{t_0}^{t_1}\sum_iF_{ci}(t)\cdot\dot{r}_i(t)\mathrm{d}t
$$
无论任何 $t_0\rightarrow t_1$ 的时间段这个总和必须为零，也就是说  
$$
\sum_iF_{ci}(t)\cdot\dot{r}_i(t)\tag{A-1}
$$
式子(A-1)在任何时候都必须为零。  
所以我们可以使用这个来从我们的推到中消除任何关于约束力 $F_{ci}$ 的部分。  
首先，在 2.3 节内定义的 “$^*$” 操作，因$a^*b=a\times b,a\times b=-b\times a$，则有：  
$$
-a^*b=b\times a=b^*a.\tag{A-2} 
$$
因为 $a^*$ 是反对称矩阵，  
$$
(a^*)^T=-a^*.\tag{A-3}
$$
最后因为$^*$ 是线性操作，  
$$
(\dot{a})^*=\dot{(a^*)}=\frac{\mathrm{d}}{\mathrm{d}t}(a^*)\tag{A-4}
$$
对于多个向量 $a_i$ 的集合  
$$
\sum a_i^*=(\sum a_i)^*.\tag{A-5}
$$

回想一下，我们可以把速度 $\dot{r}_i$ 写作 $\dot{r}_i=v+\omega\times(r_i-x)$ 这里 $r_i$ 是粒子位置，
$x$ 是质心位置，$v,\omega$ 分别是线速度和角速度。令 $r'_i=r_i-x$，并使用 $^*$ 运算符。  
$$
\dot{r}_i=v+\omega^*r_i'=v-r_i'^*\omega.\tag{A-6}
$$
将其带入$\sum F_{ci}\cdot \dot{r}_i$，该式恒为零  
$$
\sum F_{ci}\cdot(v-r_i'^*\omega)=0\tag{A-7}
$$
注意该等式对于任何的 $v,\omega$ 都成立。因为 $v,\omega$ 二者相独立，如果我们将 $\omega$ 设为零，则对于任何 $v$ 的取值，我们都必须满足 $\sum F_{ci}\cdot v=0$，那实际上就是 $\sum F_{ci}=\textbf{0}$。这意味着约束力不会产生合力。同样的，假如我们令 $v$ 取零，则$\sum-F_{ci}\cdot(r_i'^*\omega)=0$ 对于任何 $\omega$ 取值都成立。重写为 $F_{ci}\cdot(r_i'^*\omega)\rightarrow F_{ci}^T(r_i'^*\omega)$，能够得到：  
$$
\sum-F_{ci}^Tr_i'^*\omega=(\sum-F_{ci}^Tr_i'^*)\omega=0\tag{A-8}
$$
对于任何 $\omega$ 取值，都存在 $\sum-F_{ci}^Tr_i'^*=\textbf0^T$。转置后得：  
$$
\sum-(r_i'^*)^TF_{ci}=\sum(r_i')^*F_{ci}=\sum r_i'\times F_{ci}=\textbf{0}\tag{A-9}
$$
这意味着内力不会产生扭矩。  
可以使用上述推导出刚体的运动方程。作用在每个粒子上的合力是内力 $F_{ci}$ 和外力 $F_i$ 之合。第 i 个粒子的加速度 $\ddot{r}_i$ :  
$$
\ddot{r}_i=\frac{\mathrm{d}}{\mathrm{d}t}\dot{r}_i=
\frac{\mathrm{d}}{\mathrm{d}t}(v-r_i'^*\omega)=\dot{v}-\dot{r}_i'^*\omega-r_i'^*\dot\omega\tag{A-10}
$$
因为每个粒子都要遵循牛顿定律 $F=ma$，或者 $ma-f=\textbf0$  
$$
m_i\ddot{r}_i-F_i-F_{ci}=m_i(\dot{v}-\dot{r}_i'^*\omega-r_i'^*\dot\omega)-F_i-F_{ci}=\textbf0\tag{A-11}
$$
为了计算 $\dot{P}=F=\sum F_i$，使用等式 A-11 来对所有粒子求合，  
$$
\sum m_i(\dot{v}-\dot{r}_i'^*\omega-r_i'^*\dot\omega)-F_i-F_{ci}=\textbf0\tag{A-12}
$$
拆分这个大的求合运算为多个小的求合：  
$$
\begin{aligned}
\sum m_i(\dot{v}-\dot{r}_i'^*\omega-r_i'^*\dot\omega)-F_i-F_{ci}&=\\
\sum m_i\dot{v}-
\sum m_i\dot{r}_i'^*\omega-
\sum m_ir_i'^*\dot\omega -
\sum F_i-\sum F_{ci} &=\\
\sum m_i\dot{v}-
(\sum m_i\dot{r}_i')^*\omega-
(\sum m_ir_i')^*\dot\omega-
\sum F_i-\sum F_{ci}&=\\
\sum m_i\dot{v}-
(\frac{\mathrm{d}}{\mathrm{d}t}\sum m_ir_i')^*\omega-
(\sum m_ir_i')^*\dot\omega-
\sum F_i-\sum F_{ci}
&=\textbf0
\end{aligned}\tag{A-13}
$$

因为我们在以质心为原点的坐标系中，通过 2.6 节的等式 (2-20) 我们能知道 $\sum m_ir_i'=\textbf0$，这也就是说 $\frac{\mathrm{d}}{\mathrm{d}t}\sum m_ir_i'=\textbf0$。去除带有 $\sum m_ir_i'$ 和 $\sum F_{ci}$ 的项，得到下面式子：  
$$
\sum m_i\dot{v}-\sum F_i=\textbf{0}\tag{A-14}
$$
简而言之，就是 $M\dot{v}=\dot{P}=\sum F_i$。  
注意到对角动量求导有 $\dot{L}=\tau=\sum r_i'\times F_i$，对于式 (A-11) 两侧都乘上 $r_i'^*$  
$$
r_i'^*m_i(\dot{v}-\dot{r}_i'^*\omega-r_i'^*\dot\omega)-r_i'^*F_i-r_i'^*F_{ci}=r_i'^*\textbf0=\textbf0\tag{A-15}
$$
对所有粒子求合：  
$$
(\sum m_ir_i')^*\dot{v}-(\sum m_ir_i'^*\dot{r}_i'^*)\omega
-(\sum m_ir_i'^*r_i'^*)\dot\omega-
\sum r_i'^*F_i-\sum r_i'^*F_{ci}=\textbf0\tag{A-16}
$$
因为 $\sum r_i'^*F_{ci}=\textbf0$ 能够得到  
$$
(\sum m_ir_i')^*\dot{v}-(\sum m_ir_i'^*\dot{r}_i'^*)\omega-
(\sum m_ir_i'^*r_i'^*)\dot{\omega}-\sum r_i'^*F_i=\textbf0\tag{A-17}
$$
使用 $\sum m_ir_i'=\textbf0$ 得  
$$
-(\sum m_ir_i'^*\dot{r}_i'^*)\omega-
(\sum m_ir_i'^*r_i'^*)\dot{\omega}-\sum r_i'^*F_i=\textbf0\tag{A-18}
$$
因 $\sum r_i'^*F_i=\sum r_i'\times F_i=\tau$  
$$
-(\sum m_ir_i'^*\dot{r}_i'^*)\omega-
(\sum m_ir_i'^*r_i'^*)\dot{\omega}=\tau\tag{A-19}
$$
我们的化简差不多了，如果还记得 $^*$ 运算的定义，能够很简单的得到矩阵 $-a^*a^*$ 是等价于 $(a^Ta)\textbf{1}-aa^T$ 这里 $\textbf1$ 是3x3 的单位矩阵。(这个规则等价于向量规则：$a\times(b\times c)=ba^Tc-ca^Tb.$) 所以：  
$$
\sum -m_ir_i'^*r_i'^*=
\sum m_i\big((r_i'^Tr_i')\textbf1-r_i'r_i'^T\big)=
I(t)\tag{A-20}
$$
带入 (A-19)  
$$
(\sum-m_ir_i'^*\dot{r}_i'^*)\omega+I(t)\dot\omega=\tau\tag{A-21}
$$
上式基本计算完成，给出了角加速度 $\dot\omega$ 关于扭矩 $\tau$ 参数的表达式，只是我们需要先计算出矩阵 $\sum m_ir_i'^*\dot{r}_i'^*$，该矩阵计算复杂程度不亚于计算惯性张量。我们使用最后一个 trick 方法来解决。因为 $\dot{r}_i'=\omega\times r_i',r_i'^*\omega=-\omega\times r_i'$，能得到：  
$$
\sum m_i\dot{r}_i'^*r_i'^*\omega=
\sum m_i(\omega\times r_i')^*(-\omega\times r_i')=
\sum -m_i(\omega\times r_i')\times(\omega\times r_i')=\textbf0
\tag{A-22}
$$
所以我们可以把 $-\sum m_i\dot{r}_i'^*r_i'^*\omega=\textbf0$ 添加到式 A-21  
$$
(\sum-m_ir_i'^*\dot{r}_i'^*-m_i\dot{r}_i'^*r_i'^*)\omega+
I(t)\dot\omega=\tau\tag{A-24}
$$

$$
\dot{I}(t)=\frac{\mathrm{d}}{\mathrm{d}t}\sum-m_ir_i'^*r_i'^*=
\sum -m_ir_i'^*\dot{r}_i'^*-m_i\dot{r}_i'^*r_i'^*\tag{A-24}
$$

$$
\dot{I}(t)\omega+I(t)\dot\omega=
\frac{\mathrm{d}}{\mathrm{d}t}\big(I(t)\omega\big)=\tau.
\tag{A-25}
$$

因为 角动量等于 $L(t)=I(t)\omega(t)$，所以能得到的就是：  
$$
\dot{L}(t)=\tau\tag{A-26}
$$

### Appendix B Quaternion Derivations

对旋转求导 $\dot{q}(t)$ 的推导过程如下。回想刚体角速度 $\omega(t)$ 的描述，物体以大小为 $|\omega(t)|$ 的量级绕着某轴旋转。假设一个物体以恒定的角速度 $\omega(t)$ 旋转。然后物体在一段时间 $\Delta t$ 后的旋转用四元数表示为：$[\cos\frac{|\omega(t)|\Delta t}{2},\sin\frac{|\omega(t)|\Delta t}{2}\frac{\omega(t)}{|\omega(t)|}].$  
我们计算在 $t_0$ 某时刻的 $\dot{q}(t)$。在时刻 $t+\Delta t$($\Delta t$ 是小值)，物体的方向是(一阶精度 to within first order) 先由 $q(t_0)$ 的旋转然后再经过角速度 $\omega(t_0)$ 旋转 $\Delta t$ 时间的结果。组合两次旋转，能够列出：  
$$
q(t_0+\Delta t)=
[\cos\frac{|\omega(t_0)|\Delta t}{2},
\sin\frac{|\omega(t_0)|\Delta t}{2}
\frac{\omega(t_0)}{|\omega(t_0)|}]
q(t_0).
\tag{B-1}
$$
我们用 $t=t_0+\Delta t$ 来替换表示：  
$$
q(t)=[\cos\frac{|\omega(t_0)|(t-t_0)}{2},
\sin\frac{|\omega(t_0)(t-t_0)|}{2}
\frac{\omega(t_0)}{|\omega(t_0)|}]
q(t_0).\tag{B-2}
$$
让我们在时刻 $t_0$ 对 $q(t)$ 求导。首先，因为 $q(t_0)$ 是一个常数，所以我们只需要求导  
$[\cos\frac{|\omega(t_0)|(t-t_0)}{2},\sin\frac{|\omega(t_0)(t-t_0)|}{2}\frac{\omega(t_0)}{|\omega(t_0)|}]$  
在时刻 $t=t_0$  
$$
\begin{aligned}
\frac{\mathrm{d}}{\mathrm{d}t}\cos\frac{|\omega(t_0)|(t-t_0)}{2}
&=
-\frac{|\omega(t_0)|}{2}\sin\frac{|\omega(t_0)|(t-t_0)}{2}\\
&=-\frac{|\omega(t_0)|}{2}\sin0=0
\end{aligned}
\tag{B-3}
$$
相似的  
$$
\begin{aligned}
\frac{\mathrm{d}}{\mathrm{d}t}\sin\frac{|\omega(t_0)|(t-t_0)}{2}
&=
\frac{|\omega(t_0)|}{2}\cos\frac{|\omega(t_0)|(t-t_0)}{2}\\
&=\frac{|\omega(t_0)|}{2}\cos0=\frac{|\omega(t_0)|}{2}
\end{aligned}
\tag{B-4}
$$
因此在 $t=t_0$  
$$
\begin{aligned}
\dot{q}(t)&=
\frac{\mathrm{d}}{\mathrm{d}t}
\Big([\cos\frac{|\omega(t_0)|(t-t_0)}{2},
\sin\frac{|\omega(t_0)|(t-t_0)}{2}
\frac{\omega(t_0)}{|\omega(t_0)|}]q(t_0)\Big)\\
&=
\frac{\mathrm{d}}{\mathrm{d}t}
\Big([\cos\frac{|\omega(t_0)|(t-t_0)}{2},
\sin\frac{|\omega(t_0)|(t-t_0)}{2}
\frac{\omega(t_0)}{|\omega(t_0)|}]\Big)q(t_0)\\
&=
[0,\frac{|\omega(t_0)|}{2}\frac{\omega(t_0)}{|\omega(t_0)|}]q(t_0)\\
&=
[0,\frac{1}{2}\omega(t_0)]q(t_0)=
\frac{1}{2}[0,\omega(t_0)]q(t_0)
\end{aligned}
\tag{B-5}
$$
乘积 $[0,\omega(t_0)]q(t_0)$ 可缩写作 $\omega(t_0)q(t_0)$。因此，$\dot{q}(t_0)$ 的一般表达式为：  
$$
\dot{q}(t_0)=\frac{1}{2}\omega(t)q(t)\tag{B-6}
$$

### Appendix C Some Miscellaneous Formulas

#### C.1 Kinetic Energy

物体动能 $T$ 的定义为：  
$$
T=\sum \frac{1}{2}m_i\dot{r}_i^T\dot{r}_i.\tag{C-1}
$$
令 $r_i'=r_i-x$，有 $\dot{r}_i=v(t)+r_i'^*\omega$，因此  
$$
\begin{aligned}
T&=\sum\frac{1}{2}m_i\dot{r}_i^T\dot{r}_i\\
&=\sum\frac{1}{2}m_i(v+r_i'^*\omega)^T(v+r_i'^*\omega)\\
&=\frac{1}{2}\sum m_iv^Tv+
\sum v^Tm_ir_i'^*\omega+
\frac{1}{2}\sum m_i(r_i'^*\omega)^T(r_i'^*\omega)\\
&=\frac{1}{2}v^T(\sum m_i)v+v^T(\sum m_ir_i'^*)\omega
+\frac{1}{2}\omega^T\Big(\sum m_i(r_i'^*)^Tr_i'^*\Big)\omega
\end{aligned}
\tag{C-2}
$$
因 $\sum m_ir_i'=\textbf0,(r_i'^*)^T=-r_i'^*$  
$$
\begin{aligned}
T&=\frac{1}{2}v^TMv+
\frac{1}{2}\omega^T\Big(\sum -m_ir_i'^*r_i'^*\Big)\omega\\
&=\frac{1}{2}(v^TMv+\omega^TI\omega)
\end{aligned}
\tag{C-3}
$$
这里的 $I=\sum -m_ir_i'^*r_i'^*$ 是 Appendix A 的式 (A-20)。所以动能能够由两部分表达：线性部分 $\frac{1}{2}^TMv$ 和 角度部分 $\frac{1}{2}\omega^TI\omega$。

#### C.2 Angular Acceleration

我们经常需要计算 $\omega(t)$。因为 $L(t)=I(t)\omega(t)$，我们知道 $\omega(t)=I^{-1}(t)L(t)$。  
$$
\dot{\omega}(t)=\dot{I}^{-1}(t)L(t)+I^{-1}(t)\dot{L}(t)\tag{C-4}
$$
因 $\dot{L}(t)=\tau(t)$ (A-26) 式，主要处理 $\dot{I}^{-1}(t)$，在式 (2-40) 能得到  
$$
I^{-1}(t)=R(t)I^{-1}_{body}R(t)^T.
$$

$$
\dot{I}^{-1}(t)=\dot{R}(t)I^{-1}_{body}R(t)^T+R(t)I^{-1}_{body}\dot{R}(t)^T.\tag{C-5}
$$

因为式 (2-12) $\dot{R}(t)=\omega(t)^*R(t)$  
$$
\dot{R}(t)^T=(\omega(t)^*R(t))^T=R(t)^T\big(\omega(t)^*\big)^T
\tag{C-6}
$$
因为 $\omega(t)^*$ 是反对称矩阵(antisymmetric matrix)，($\big(\omega(t)^*\big)^T=-\omega(t)^*$),  
$$
\dot{R}(t)^T=-R(t)^T\omega(t)^*\tag{C-7}
$$

$$
\begin{aligned}
\dot{I}^{-1}(t)&=
\dot{R}(t)I^{-1}_{body}R(t)^T+R(t)I^{-1}_{body}\dot{R}(t)^T\\
&=\dot{R}(t)I^{-1}_{body}R(t)^T-
R(t)I^{-1}_{body}R(t)^T\omega(t)^*\\
&=\omega(t)^*R(t)I^{-1}_{body}R(t)^T-I^{-1}(t)\omega(t)^*\\
&=\omega(t)^*I^{-1}(t)-I^{-1}(t)\omega(t)^*
\end{aligned}
\tag{C-8}
$$

$$
\begin{aligned}
\dot\omega(t)&=\dot{I}^{-1}L(t)+I^{-1}(t)\dot{L}(t)\\
&=\Big(\omega(t)^*I^{-1}(t)-I^{-1}(t)\omega(t)^*\Big)L(t)+
I^{-1}(t)\dot{L}(t)\\
&=\omega(t)^*I^{-1}(t)L(t)-I^{-1}(t)\omega(t)^*L(t)+
I^{-1}(t)\dot{L}(t)\\
\end{aligned}
\tag{C-9}
$$

因为我们有 $I^{-1}(t)L(t)=\omega(t)$，对于第一项 $\omega(t)^*I^{-1}(t)L(t)$ 实际上是 $\omega(t)^*\omega(t)=\omega(t)\times\omega(t)=0$，所以能够得到：  
$$
\begin{aligned}
\dot\omega(t)&=
-I^{-1}(t)\omega(t)^*L(t)+
I^{-1}(t)\dot{L}(t)\\
&=-I^{-1}(t)\omega(t)\times L(t)+
I^{-1}(t)\dot{L}(t)\\
&=I^{-1}(t)\Big(L(t)\times\omega(t)\Big)+
I^{-1}(t)\dot{L}(t)\\
&=I^{-1}(t)\Big(L(t)\times\omega(t)+\dot{L}(t)\Big)
\end{aligned}
\tag{C-10}
$$
我们可以由此看出，即使没有外力作用（即 $\dot{L}(t)=0$），角加速度 $\dot\omega(t)$ 仍可能非零。（事实上，**只要角动量与角速度的方向不一致**，就会出现这种情况——而这一现象又源于：刚体的**转动速度轴（rotational velocity axis）不是刚体的对称轴**。）

#### C.3 Acceleration of a Point

给定一点在世界空间的位置 $p(t)$，经常会需要计算 $\ddot{p}(t)$。  
将模型空间的点 $p_0$ 变换到时刻 $t$ 的世界坐标 $p(t)$: $p(t)=R(t)p_0+x(t)$。  
如果令 $r(t)=p(t)-x(t)$，则  
$$
\begin{aligned}
\dot{p}(t)
&=\dot{R}(t)p_0+\dot{x}(t)=\omega(t)^*R(t)p_0+v(t)\\
&=\omega(t)\times\Big(R(t)p_0+x(t)-x(t)\Big)+v(t)\\
&=\omega(t)\times\Big(p(t)-x(t)\Big)+v(t)\\
&=\omega(t)\times r(t)+v(t)
\end{aligned}
\tag{C-11}
$$

$$
\begin{aligned}
\ddot{p}(t)&=
\dot\omega(t)\times r(t)+
\omega(t)\times \dot{r}(t)+\dot{v}(t)\\
&=\dot\omega(t)\times r(t)+
\omega(t)\times\Big(\omega(t)\times r(t)\Big)
+\dot{v}(t)
\end{aligned}
\tag{C-12}
$$

式(C-12) 用了式 (2-7)，对于上式结果我们可以这样理解。  
第一项，$\dot\omega(t)\times r(t)$ 是该点切向加速度(tangential acceleration)，即 $\dot\omega(t)\times r(t)$ 是物体产生角加速度而产生的垂直于位移 $r(t)$ 的加速度。  
第二项，$\omega(t)\times\Big(\omega(t)\times r(t)\Big)$ 是向心加速度，这种向心加速度产生是因为规定物体是刚体，并且物体上各点必须绕质心做圆周运动<font color=#909090>*(这里我是没懂什么意思……角速度引起的向心加速度？)*</font>。  
最后一项，$\dot v(t)$ 点的线性加速度，是因为物体质心线性加速度产生。

### Appendix D Resting Contact Derivations

如果想在模拟器中实现静接触，可能会需要本附录中的派生和代码。这或许并不是一篇易于阅读的附录内容；不过话说回来，写这篇附录的过程本身也不怎么有趣啊！<font color=#909090>*(翻译下来也不是很有趣啊！keso！)*</font>  
这里的推导过程比较简洁，但是搭配原文附录末尾的代码可能能更清晰易懂。  
$$
\ddot{d_i}(t_0)=
a_{i1}f_1+a_{i2}f_2+\cdots+a_{in}f_n+b_i.\tag{D-1}
$$
给定 $i,j$，我们需要知道 $\ddot{d_i}(t_0)$ 如何依赖于 $f_j$，也就是说，需要知道 $a_{ij}$。同时，我们还要计算常数项 $b_i$。  
我们先开始计算 $a_{ij}$ 暂时忽略常数部分 $b_i$。假设第 i 个接触发生在物体 $A,B$ 之间。根据式 (9-4) 对 $\ddot{d}_i(t_0)$ 能得到：  
$$
\ddot{d}(t_0)=
\hat{n}_i(t_0)\cdot\big(
\ddot{p}_a(t_0)-\ddot{p}_b(t_0)\big)+
2\hat{n}_i(t_0)\cdot\big(
\dot{p}_a(t_0)-\dot{p}_b(t_0)\big)
\tag{D-2}
$$
这里的 $p_a(t_0)=p_i=p_b(t_0)$ 是在 $t_0$ 时刻的第 $i$ 个接触点。因此 $2\hat{n}_i(t_0)\cdot\big(\dot{p}_a(t_0)-\dot{p}_b(t_0)\big)$ 项是一个速度相关项 (你可以在不知道所涉及的力作用情况下立刻计算出来)，关于 $b_i$ 的部分，我们暂时忽略。  
现在我们只需要计算 $\ddot{p}_a(t_0),\ddot{p}_b(t_0)$ 如何依赖 $f_i$，$f_i$ 为第 i 个接触力的大小。  
我们来考虑第 i 个接触点的情况。如果物体 A 不是第 i 个接触点中的其中一个物体，说明 $\ddot{p}_a(t_0)$ 和 $f_i$ 无关，因为第 i 个接触点没有力作用在物体 A 上。同样的，如果 B 也不是第 i 个接触点中的其中一个物体，则 $\ddot{p}_b(t_0)$ 和 $f_i$ 是无关的。(就像 figure 26，第1个点的加速度是不会受到第5个接触点的影响的。因此 $\ddot{d}_1(t_0)$ 完全和 $f_5$ 无关。反之同理 $\ddot{d}_5(t_0)$ 和 $f_1$ 也是无关。)  
![](/imgs/Rigid Body Simulation Ⅱ/figure26.png)  
假设在第 i 个接触点中，包括了物体 A。为了明确起见，假设在第 i 个接触点中，一个力 $j\hat{n}_j(t_0)$ 作用在物体 A 上，而不是 $-j\hat{n}_j(t_0)$。我们需要求解作用在物体 A 上的力 $j\hat{n}_j(t_0)$ 如何影响 $\ddot{p}_a(t_0)$。  
根据式 (C-12)，能够得到：
$$
\ddot{p}_a(t)=
\dot{v}_a(t)+
\dot\omega_a(t)\times r(t)_a+
\omega_a(t)\times\Big(\omega_a(t)\times r_a(t)\Big)
\tag{D-3}
$$
这里 $r_a(t)=p_a(t)-x_a(t)$，并且$x_a(t),v_a(t),\omega_a(t)$ 全都是和物体 A 有关的变量。我们知道 $\dot{v}_a(t)$ 是物体的线性加速度，并且等于作用在 A 上的合力除以质量。因此，力 $j\hat{n}_j(t_0)$ 对加速度 $\dot{v}_a(t),\ddot{p}_a(t_0)$的贡献为:  
$$
\frac{f_j\hat{n}_j(t_0)}{m_a}=f_j\frac{\hat{n}_j(t_0)}{m_a}
\tag{D-4}
$$
同样的，根据式 (C-10),(2-13) 能针对 $\dot\omega_a(t)$ 得  
$$
\dot\omega_a(t)=I^{-1}_a(t)\tau_a(t)+
I^{-1}_a(t)\Big(L_a(t)\times\omega_a(t)\Big)
$$
这里的 $\tau_a(t)$ 是作用在物体 A 上的总扭矩。如果第 j 个 接触发生在点 $p_j$，则力 $j\hat{n}_j(t_0)$ 产生的力矩为：  
$$
\big(p_j-x_a(t_0)\big)\times f_j\hat{n}_j(t_0).
$$
因此，对于 $\ddot{p_a}(t_0)$ 的角度贡献为：  
$$
\begin{aligned}
&f_j\Big(I_a^{-1}(t_0)\big((p_j-x_a(t_0))\times\hat{n}_j(t_0)\big)\Big)\times r_a.\\
&完全看不懂为什么\cdots\\
&可能就是\dot\omega_a(t)暂时忽略了后面的陀螺项,然后算加速度?\\
\end{aligned}\tag{D-5}
$$
$\ddot{p}_a(t_0)$ 对 $f_j$ 的总依赖为：  
$$
f_j\bigg(\frac{\hat{n}_j(t_0)}{m_a}+
\Big(I_a^{-1}(t_0)\big((p_j-x_a(t_0))\times
\hat{n}_j(t_0)\big)\Big)\times r_a\bigg).
$$
如果是一个力 $-f_j\hat{n}(t_0)$ 作用在 A 上，得到的关系是一样的，只是要在 $f_j$ 前添加负号。显然 $\ddot{p}_b(t_0)$ 以类似的方式依赖于 $f_j$。一旦我们计算出 $\ddot{p}_a(t_0),\ddot{p}_b(t_0)$ 如何依赖 $f_j$，就将结果组合并和 $\hat{n}_i(t_0)$ 做点乘，观察 $\ddot{d}_i(t_0)$ 如何依赖 $f_j$。这给出了系数 $a_{ij}$。可能有点困惑，看一看后面的代码部分。

我们还需要计算出 $b_i$。我们知道 $\ddot{d_i}(t_0)$ 包含固定项  
$$
2\dot{\hat{n}}_i(t_0)\cdot
\big(\dot{p}_a(t_0)-\dot{p}_b(t_0)\big).
$$
但是我们也必须要考虑已知外力对 $\ddot{p}_a(t_0),\ddot{p}_b(t_0)$ 的影响，比如重力，同时还有和力无关的项 $\omega_a(t_0)\times(\omega_a(t_0)\times r_a),\Big(I^{-1}_a(t_0)\big(L_a(t_0)\times\omega_a(t_0)\big)\Big)\times r_a$。如果令作用在 A 上的合力为 $F_a(t_0)$ 并且合扭矩为 $\tau_a(t_0)$，接着从式 (D-4), (D-5) 中可以得到 $F_a(t_0)$ 贡献为：$\frac{F_a(t_0)}{m_a}$。  
并且 $\tau_a(t_0)$ 贡献为：$\Big(I_a^{-1}(t_0)\tau_a(t_0)\Big)\times r_a.$  
因此，$\ddot{p}_a(t_0)$ 和 $f_j$ 无关的部分为：  
$$
\begin{aligned}
&\frac{F_a(t_0)}{m_a}+
\big(I^{-1}_a(t_0)\tau_a(t_0)\big)\times r_a+
\omega_a(t_0)\times\big(\omega_a(t_0)\times r_a\big)+\\
&\Big(I^{-1}_a(t_0)\big(L_a(t_0)\times\omega_a(t_0)\big)\Big)\times r_a
\end{aligned}
$$
对于 $\ddot{p}_b(t_0)$ 是相似的。为了计算 $b_i$，我们组合 $\ddot{p}_a(t_0),\ddot{p}_b(t_0)$ 和 $\hat{n}_i(t_0)$ 做点乘，并添加项 $2\dot{\hat{n}}_i(t_0)\cdot\big(\dot{p}_a(t_0)-\dot{p}_b(t_0)\big).$

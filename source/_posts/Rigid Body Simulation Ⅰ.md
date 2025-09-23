---
title: Rigid Body Simulation Ⅰ
math: true
tags: [Math]
index_img: 
banner_img: 
data: 2025--08--12 22:09:48
typora-root-url: ./..
---

刚体模拟内容Ⅰ
<!--more-->

# Rigid Body Simulation Ⅰ

## Part I. Unconstrained Rigid Body Dynamics

### 1. Simulation Basics

模拟刚体的运动与模拟单个粒子的运动几乎是一样的，所以让我们先从粒子模拟开始。我们模拟一个粒子的方法如下。我们用函数 $x(t)$ 来表示在时间 $t$ 时该粒子在世界空间（即在模拟过程中所有粒子或物体所在空间）中的位置。函数 $v(t) = \dot{x}(t) = \frac{\mathrm{d}}{\mathrm{d}t}x(t)$ 表示该粒子在时间 $t$ 时的速度。粒子在时间 $t$ 的状态就是其位置和速度。我们通过定义一个状态向量 $\textbf{Y}(t)$ 来对整个系统进行这一概念的推广：对于单个粒子，  
$$
\textbf{Y}(t)=
\begin{pmatrix}
x(t)\\
v(t)
\end{pmatrix}\tag{1-1}
$$
当我们讨论实际的实现时，必须将 $\textbf{Y}(t)$ “展平”成一个数组。对于单个粒子， $\textbf{Y}(t)$ 可以描述为一组六个数字：通常，我们会让数组的前三个元素代表 $x(t)$，最后三个元素代表 $v(t)$。之后，当我们谈论包含矩阵以及向量的状态向量 $Y(t)$ 时，也会采用同样的操作将 $Y(t)$ 展平成一个数组。当然，我们还得反过来进行这个过程，将一组数字转换回状态向量 $Y(t)$。总的来说，这都是相当简单的，所以，我们将假定我们知道如何将任何类型的状态向量 $Y(t)$ 转换为一个适当长度的数组，反之亦然。（对于涉及粒子的简单示例，请查看这些笔记中的“粒子系统动力学”部分。）  
对于一个有着 n 粒子的系统，我们将 $\textbf{Y}(t)$ 拓展为：  
$$
\textbf{Y}(t)=
\begin{pmatrix}
x_1(t)\\
v_1(t)\\
\vdots\\
x_n(t)\\
v_n(t)
\end{pmatrix}\tag{1-2}
$$
$x_i(t),v_i(t)$ 是第 i 个粒子的位置和速度。处理 n 个粒子与处理单个粒子的难度是一样的，所以目前我们暂且将 Y(t) 设为单个粒子的状态向量（稍后当我们讨论单个刚体时，它将代表单个刚体的状态）。  
要真正模拟我们的粒子的运动，我们还需要知道另外一件事——在时刻 $t$ 时作用在该粒子上的力。我们将把作用在粒子上的力在时刻 $t$ 时的大小定义为 $F(t)$。函数 $F(t)$ 就是作用在该粒子上的所有力的总和：重力、风力、弹簧力等等。若该粒子的质量为 $m$ ，那么其随时间变化的 $\textbf{Y}$ 值可由以下公式表示：  
$$
\frac{\mathrm{d}}{\mathrm{d}t}\textbf{Y}(t)=
\frac{\mathrm{d}}{\mathrm{d}t}
\begin{pmatrix}
x(t)\\
v(t)
\end{pmatrix}=
\begin{pmatrix}
v(t)\\
F(t)/m
\end{pmatrix}.
\tag{1-3}
$$
给定任意时刻 $\textbf{Y}(t)$ 值，等式 1-3 描述了 $\textbf{Y}(t)$ 的瞬时变化率。模拟过程以 $Y(0)$ 为初始条件 (即$x(0),v(0)$) 并通过数值微分方程求解器跟踪 $\textbf{Y}$ 随着时间流动。  
求解器使用何种数值方法其实我们不关心，让我们看一下在 C++ 如何实现数组求解器，假设我们访问一个数值求解器，通常该 ODE 函数应该是：  

~~~C++
typedef void (*dydt_func)(double t[], double t[], double ydot[]);
void ode(double y0[], double yend[], int len, double t0,
        double t1, dydt_func dydt);
~~~

我们将初始状态向量数组 y0 传到 ode 函数。求解器对 y 的内部并不关心，由于求解器可以解决任意维度的问题，我们还必须传 y0 的长度 len。对于一个有 n 个粒子的系统，len = 6n。此外要提供开始和结束时间 t0,t1 求解器的目标是计算 t1 时刻的状态向量并返回到 yend 中。  
我们也提供了函数 dydt 给 ode 函数。状态向量 $\textbf{Y}(t)$ 和 时间 $t$ 编码成一个数组 y，dydt 必须计算并返回 $\frac{\mathrm{d}}{\mathrm{d}t}\textbf{Y}(t)$ 于数组 ydot。必须把 t 传递给 dydt 是因为系统中可能会有 随时间变化的力，这种情况下，dydt 必须知道 时间 来确定力。在跟踪从 t0 到 t1 $\textbf{Y}(t)$ 的变化中，求解器可以随时调用 dydt。假设我们有一个 ode 的例子，我们需要做的就是编写 dydt 将它作为参数传给 ode。  
模拟刚体与模拟粒子遵循完全相同的模式。唯一的区别是刚体的状态向量 $\textbf{Y}(t)$ 包含了更多的信息，而导数 $\frac{\mathrm{d}}{\mathrm{d}t}\textbf{Y}(t)$ 则稍微复杂一些。然而，我们将使用完全相同的范例，使用求解器ode来跟踪刚体的运动，同样我们将提供一个函数 dydt。

### 2. Rigid Body Concepts

#### 2.1 Position and Orientation

粒子在 $t$ 时刻在空间中的位置可以被描述为向量 $x(t)$，表示距离原点的位移。刚体比粒子复杂，在位移的基础上要添加旋转。为了定位一个在世界坐标中的刚体，我们用向量 $x(t)$ 表示刚体的位移，同时必须描述刚体的旋转，暂时用一个 3x3 的旋转矩阵 $R(t)$ 表示，$x(t),R(t)$ 为一个刚体的控件变量。  
刚体和粒子不同，其占据一定的空间具有一定的形状。因为刚体不会形变只会位移和旋转，所以我们用一个固定的大小不会变化的空间定义刚体的形状，称之为 **体空间**。给定体空间的几何描述，然后用 $x(t),R(t)$ 变换体空间的描述至世界空间(figure 1)。为了简化使用的一些方程、等式，我们规定刚体在体空间的质心为原点 (0, 0, 0)。后面会更准确地定义质心，这里可以先认为物体的几何中心就是质心。在描述物体形状时，需要保证物体的几何中心位于 物体空间的  (0, 0, 0)。我们定义 $R(t)$ 表示物体绕质心的旋转，一个在 物体空间 固定的向量 $r$ 在时刻 $t$ 将被旋转至 $R(t)r$。则刚体上任意一点在物体空间的 $p_0$ 先绕原点旋转后位移的结果是 $p(t)$：  
$$
p(t)=R(t)p_0+x(t)\tag{2-1}
$$
![](/imgs/Rigid Body Simulation Ⅰ/Figure1.png)  
由于质心位于原点，所以我们可以从 $x(t)$ 直接获得质心的世界坐标，所以 $x(t)$ 是时刻 $t$ 质心的位置。对于旋转矩阵也有类似的物理意义，想象 物体空间的 x 轴，(1, 0, 0) ，在时间 $t$ 有世界空间向量：  
$$
R(t)\begin{pmatrix}
1\\
0\\
0\\
\end{pmatrix}\\
$$
$$
R(t)=\begin{pmatrix}
r_{xx}\;r_{yx}\;r_{zx}\\
r_{xy}\;r_{yy}\;r_{zy}\\
r_{xz}\;r_{yz}\;r_{zz}
\end{pmatrix},\tag{2-2}
$$

$$
\begin{aligned}
R(t)
\begin{pmatrix}
1\\
0\\
0\\
\end{pmatrix}=
\begin{pmatrix}
r_{xx}\\
r_{xy}\\
r_{xz}\\
\end{pmatrix}
\end{aligned}\tag{2-3}
$$

可以看到就是第一列，$R(t)$ 的物理意义即(1, 2, 3)列 为 刚体(x, y, z) 轴在时间 $t$ 变换到世界空间时所指示方向。  
![](/imgs/Rigid Body Simulation Ⅰ/Figure2.png)

#### 2.2 Linear Velocity

为了简化，我们称 $x(t),R(t)$ 为 位置 和 朝向。接下来要做的就是定义 位置 和 朝向 如何随时间变化。我们需要 $\dot{x}(t),\dot{R}(t)$。因为 $x(t)$ 是质心在世界空间的坐标，$\dot{x}(t)$ 即质心在世界空间的速度。

#### 2.3 Angular Velocity

除了为位移外，物体还可以旋转，我们先不考虑位移，将其固定。那么，物体空间各点的任何移动都必定是由于刚体绕着通过质心的某根轴旋转所致。定义该自旋为 $\omega(t)$ ，其方向决定了物体**旋转轴**的方向 (figure 3)。$\omega(t)$ 的大小 $|\omega(t)|$ 表示物体旋转的快慢。 $|\omega(t)|$ 的量纲是 [转/单位时间],因此它关联了物体在**角速度保持恒定**的前提下，给定时间内将转过的角度，物理量 $\omega(t)$ 称之为角速度。  
![](/imgs/Rigid Body Simulation Ⅰ/Figure3.png)  
对于线速度我们可以对 $x(t)$ 在时间上求导 $v(t)=\frac{\mathrm{d}}{\mathrm{d}t}x(t)$。物体旋转 $R(t)$ 如何与角速度 $\omega(t)$ 关联？尤其是 $R(t)$ 是矩阵 而 $\omega(t)$ 是向量。  
我们知道 $R(t)$ 的列表示 物体 x,y,z 轴在时刻 t 变换后的方向。这意味着 $\dot R(t)$ 的列一定分布表示  x,y,z 轴被变换的速度。为了解决 $R(t)$ 和  $\omega(t)$ 的关联，需要观测刚体中任意矢量变化和角速度 $\omega(t)$ 的关系。  
Figure 4 展示了角速度为 $\omega(t)$ 的刚体，思考在世界空间中指定时刻 $t$ 的向量 $r(t)$。假设该向量固定在刚体上，即 $r(t)$ 会随着刚体在世界空间在运动。由于 $r(t)$ 是一个方向向量，是不受平移影响的也就是说 $\dot r(t)$ 和 $v(t)$ 无关。为了研究 $\dot r(t)$，将其分解为向量a, b，其中 a 平行于 $\omega(t)$ ，b 垂直于  $\omega(t)$ 。假设刚体保持恒定的角速度，使 $r(t)$ 的尖端绕 $\omega (t)$ 轴画了一个圆(figure 4)。圆的半径即为 $|b|$，因为向量 $r(t)$ 的尖端是瞬时沿着圆运动的，所以 $r(t)$ 的瞬时变化率即 $\dot{r}(t)$ 是同时垂直于 $b,\omega(t)$。又因为 $r(t)$ 端点做绕半径 $b$ 的圆周运动，$r(t)$ 的瞬时速度大小为 $|b|\cdot|\omega(t)|$，由于 $b,\omega(t)$ 垂直则有  
$$
|\omega(t)\times b|=|\omega(t)|\cdot|b|\tag{2-5}
$$
![](/imgs/Rigid Body Simulation Ⅰ/Figure4.png)  
整理后我们能得到 $\dot{r}(t)=\omega(t)\times(b)$。因为 $r(t)=a+b$ 但是 $a$ 和 $\omega(t)$ 垂直  
$$
\dot r(t)=\omega(t)\times b=\omega(t)\times b+\omega(t)\times a=\omega(t)\times (b+a)\tag{2-6}
$$

$$
\dot r(t)=\omega(t)\times r(t)\tag{2-7}
$$

我们知道时刻 $t$ 刚体在世界空间的 x 轴是 旋转矩阵的第一列，其导数就是该向量的变化率，利用刚才的式子：
$$
\omega(t)\times
\begin{pmatrix}
r_{xx}\\
r_{xy}\\
r_{xz}
\end{pmatrix}
$$
对于其他轴向也有：
$$
\begin{aligned}
\dot R=\begin{pmatrix}
\omega(t)\times\begin{pmatrix}
r_{xx}\\
r_{xy}\\
r_{xz}
\end{pmatrix},
\omega(t)\times\begin{pmatrix}
r_{yx}\\
r_{yy}\\
r_{yz}
\end{pmatrix},
\omega(t)\times\begin{pmatrix}
r_{zx}\\
r_{zy}\\
r_{zz}
\end{pmatrix}
\end{pmatrix}
\end{aligned}\tag{2-8}
$$


这样写很麻烦，我们可以简化，因 a，b 都是三维向量，则 $a\times b$ 亦是：

$$
a\times b=\begin{pmatrix}
a_yb_z-b_ya_z\\
a_zb_x-b_za_x\\
a_xb_y-b_xa_y
\end{pmatrix}
$$
我们定义一个矩阵 $a^*$: 
$$
{
a^*=\begin{pmatrix}
&0 &-a_z &a_y\\
&a_z &0 &-a_x\\
&0 &-a_z &a_y
\end{pmatrix}
}
$$

$$
{
a^*b=\begin{pmatrix}
a_yb_z-b_ya_z\\
a_zb_x-b_za_x\\
a_xb_y-b_xa_y
\end{pmatrix}=a\times b\tag{2-9}
}
$$

$$
\dot{R}(t)=
\begin{pmatrix}
\omega(t)^*
\begin{pmatrix}
r_{xx}\\
r_{xy}\\
r_{xz}
\end{pmatrix}
&\omega(t)^*
\begin{pmatrix}
r_{yx}\\
r_{yy}\\
r_{yz}
\end{pmatrix}
&\omega(t)^*
\begin{pmatrix}
r_{zx}\\
r_{zy}\\
r_{zz}
\end{pmatrix}
\end{pmatrix}\tag{2-10}
$$

$$
\dot{R}(t)=
\omega(t)^*
\begin{pmatrix}
\begin{pmatrix}
r_{xx}\\
r_{xy}\\
r_{xz}
\end{pmatrix}
&
\begin{pmatrix}
r_{yx}\\
r_{yy}\\
r_{yz}
\end{pmatrix}
&
\begin{pmatrix}
r_{zx}\\
r_{zy}\\
r_{zz}
\end{pmatrix}
\end{pmatrix}\tag{2-11}
$$

$$
\dot{R}(t)=
\omega(t)^*{R}(t)\tag{2-12}
$$

#### 2.4 Mass of a Body

为了做一些推导，我们需要对刚体进行一些理论上的积分，我们暂时认为刚体是由很多微小粒子组成的，编号为1 到 N，第 i 个粒子的质量为 $m_i$ 每个粒子在空间中有起始位置 $r_{0i}$ 。在 t 时刻：  
$$
r_i(t)=R(t)r_{0i}+x(t)\tag{2-13}
$$
刚体的质量 $M$ 为：  
$$
M=\sum^N_{i=1}m_i\tag{2-14}
$$

#### 2.5 Velocity of Particle

第 i 个粒子的速度 $\dot{r}_i(t)$ 是对 等式2-13 的求导，记得 $\dot{R}(t)=\omega^*R(t)$：  
$$
\dot{r}_i(t)=\omega^*R(t)r_{0i}+v(t)\tag{2-15}
$$
可以重写为：  
$$
\begin{aligned}
\dot{r}_i(t)&=\omega^*R(t)r_{0i}+v(t)\\
&=\omega^*(R(t)r_{0i}+x(t)-x(t))+v(t)\\
&=\omega^*(r_i(t)-x(t))+v(t)
\end{aligned}\tag{2-16}
$$
这里写成叉乘：  
$$
\dot{r}_i(t)=\omega(t)\times(r_i(t)-x(t))+v(t)\tag{2-17}
$$
所以刚体上一点粒子的速度有两个分量，一个是线性分量 $v(t)$ 及一个角速度分量 $\omega(t)\times(r_i(t)-x(t))$  
![](/imgs/Rigid Body Simulation Ⅰ/Figure5.png)

#### 2.6 Center of Mass

我们对质心的定义将使我们能够同样，地将物体的动力学分解为 线性分量 和 角分量。  
$$
\frac{\sum m_ir_i(t)}{M}\tag{2-18}
$$
其中 $M$ 是物体的质量，当我们说我们正在使用质心坐标系时，意思是在物体空间中，以质心为原点:  
$$
\frac{\sum m_ir_{0i}}{M}=\textbf{0}=
\begin{pmatrix}
0\\0\\0
\end{pmatrix}\tag{2-19}
$$
关于 $x(t)$ 是 $t$ 时刻的质心位置：  
$$
\begin{aligned}
\frac{\sum m_ir_i(t)}{M}&=
\frac{\sum m_i(R(t)r_{0i}+x(t))}{M}\\
&=\frac{R(t)\sum m_ir_{0i}+\sum m_ix(t)}{M}\\
&=R(t)\textbf{0}+x(t)\frac{\sum m_i}{M}\\
&=x(t)
\end{aligned}
$$

$$
\begin{aligned}
&\sum m_i(r_i(t)-x(t))=\\
&\sum m_i(R(t)r_{0i}+x(t)-x(t))=R(t)\sum m_ir_{0i}=\textbf{0}
\end{aligned}\tag{2-20}
$$

#### 2.7 Force and Torque

当我们想象由于外部因素导致一个力施加在刚体上，比如重力、风、或接触力，我们想象这个力恰巧作用在刚体一个特定的粒子上。作用力的作用位置决定了力所施加的位置。我们用 $F_i(t)$ 表示 $t$ 时刻作用在第 i 个粒子上的力。定义第 i 个粒子的外力矩为 $\tau_i(t)$  
*(记住，我们的粒子模型只是概念性质的，我们可以在任何几何位置去想象一个力不论是物体表面上或者物体内部的，因为我们都可以随意想象那恰好有个粒子)*  
$$
\tau_i(t)=(r_i(t)-x(t))\times F_i(t)\tag{2-21}
$$
扭矩与力不同在于，作用于粒子上的扭矩取决于粒子相对质心的位置，我们可以直观的发现 $\tau_i(t)$ 的方向就是当质心被固定时，由于 $F_i(t)$ 作用，物体将会绕其旋转的轴。(figure 6)  
![](/imgs/Rigid Body Simulation Ⅰ/Figure6.png)  
作用在物体上的力为作用在粒子上的总和：  
$$
F(t)=\sum F_i(t)\tag{2-22}
$$
外力矩：  
$$
\tau(t)=\sum\tau_i(t)=\sum(r_i(t)-x(t))\times F_i(t)\tag{2-23}
$$
可以注意到 $F(t)$ 没有包含作用于物体的各个力的位置信息。然而 $\tau(t)$ 的确能告知我们 $F_i(t)$ 在物体上的分布情况。

#### 2.8 Linear Momentum

$$
p=mv\tag{2-24}
$$

$$
P(t)=\sum m_i\dot{r}(t)\tag{2-25}
$$

根据等式 2-17，$\dot{r}_i(t)=\omega(t)\times(r_i(t)-x(t))+v(t)$，因此物体的线性动量为：  
$$
\begin{aligned}
P(t)&=\sum m_i\dot{r}_i(t)\\
&=\sum[m_iv(t)+m_i\omega(t)\times(r_i(t)-x(t))]\\
&=\sum m_iv(t)+\omega(t)\sum m_i(r_i(t)-x(t)).
\end{aligned}\tag{2-26}
$$
因为我们以质心为原点所以能够应用 2-20，得  
$$
P(t)=\sum m_iv(t)=v(t)\sum m_i=Mv(t)\tag{2-27}
$$
刚体的线性动量就像一个粒子的线性动量简单，所以我们能得到  
$$
\dot{v}(t)=\frac{\dot{P}(t)}{M}\tag{2-28}
$$
我们知道 $p=mv,\dot{p}=ma=f$，所以能够有  
$$
\dot{P}(t)=F(t)\tag{2-29}
$$
就是说，线性动量的变换等于作用在刚体上的总力。$P(t)$ 不包含旋转速度，同样 $F(t)$ 也不包含。  
由于 $P(t)$ 和 $v(t)$ 的关系简单，我们使用 $P(t)$ 来表示刚体的状态，而非 $v(t)$。如果使用 $v(t)$ 可以使用下面的关系  
$$
\dot v(t)=\frac{F(t)}{M}\tag{2-30}
$$
不过使用 $P(t)$ 更符合我们处理角速度和加速度的形式。

#### 2.9 Angular Momentum

虽然线性动量非常直观 ( $P(t)=Mv(t)$ )，但角动量不是。在刚体中引入角动量唯一的好处就是能够写成比执着于角速度更简洁的等式。因此或许不必纠结给角动量附加直观的物理解释——毕竟，它本质上是**极为反直觉**的概念。角动量最终能简化方程，源于**自然界中角动量守恒**，而角速度不守恒：若刚体在无外力矩的太空漂浮，其角动量恒定；但即使刚体的角动量恒定，角速度也可能变化！正因如此，刚体的角速度**即便不受力也可能改变**。由此，“选择角动量作为状态变量”比角速度更合理——最终在数学处理上更简便。  
从线性动量我们知道 $P(t)=Mv(t)$，相似的我们定义刚体总角动量 $L(t)=I(t)\omega(t)$，其中 $I(t)$ 是一个 3x3 的矩阵，严格而言是二阶张量，称之为 **惯性张量**。惯性张量表述了刚体相对于其质心的质量分布，该张量决定物体的朝向，但是不依赖物体的平动。  
要注意的是无论线性还是角度，动量都是一个速度的线性函数，区别仅在于角动量的 “缩放因子” 是矩阵——惯性张量，而平动动量的 “缩放因子” 是标量——质量。此外，角动量不受平动影响，平动动量也不受转动影响。  
角动量 $L(t)$ 和 外力矩 $\tau(t)$ 的关系非常简单：  
$$
\dot{L}(t)=\tau(t)\tag{2-31}
$$
类似 $\dot{P}(t)=F(t)$.

#### 2.10 The Inertia Tensor

惯性张量是角速度和角动量之间的缩放系数。在给定时刻 $t$ ，令 $r_i'$ 表示第 i 个粒子距离 $x(t)$ 的位移 $r_i'=r_i(t)-x(t)$……不就是 $r_i'=R(t)r_{0i}$  
$$
\begin{aligned}
I(t)=\sum
\begin{pmatrix}
m_i(r_{iy}^{'2}+r_{iz}^{'2}) 
& -m_ir_{ix}^{'}r_{iy}^{'}
& -m_ir_{iz}^{'}r_{ix}^{'}\\

-m_ir_{ix}^{'}r_{iy}^{'}
& m_i(r_{iz}^{'2}+r_{ix}^{'2})
& -m_ir_{iy}^{'}r_{iz}^{'}\\

-m_ir_{iz}^{'}r_{ix}^{'}
& m_ir_{iy}^{'}r_{iz}^{'}
& m_i(r_{ix}^{'2}+r_{iy}^{'2})
\end{pmatrix}
\end{aligned}\tag{2-32}
$$
在**实际实现**中，我们会将**有限求和**替换为对刚体体积的**积分**（在世界坐标系中进行）。质量项 $m_i$ 会被**密度函数**取代。

乍看之下，似乎每当刚体的朝向 $R(t)$ 变化时，都需要重新计算积分来求解惯性张量 $I(t)$。除非刚体形状极其简单（例如球形或立方体，这类形状的积分可解析求解），否则在仿真过程中执行这种计算会**代价过高**。  
幸运的是，通过使用物体坐标系，我们可以基于**预计算积分**（在体坐标系中完成），低成本地计算任意朝向 $R(t)$ 下的惯性张量。（该积分通常在仿真启动前完成，应被视为描述刚体物理属性的输入参数之一。）使用式子 $r_i^{'T}r_i^{'}=r_{ix}^{'2}+r_{iy}^{'2}+r_{iz}^{'2}$，可以重写
$$
I(t)=\sum m_ir_i^{'T}r_i^{'}
\begin{pmatrix}
1&0&0\\
0&1&0\\
0&0&1
\end{pmatrix}-
\begin{pmatrix}
m_ir_{ix}^{'2}
& m_ir_{ix}^{'}r_{iy}^{'}
& m_ir_{ix}^{'}r_{iz}^{'}\\
m_ir_{iy}^{'}r_{ix}^{'}
& m_ir_{iy}^{'2}
& m_ir_{iy}^{'}r_{iz}^{'}\\
m_ir_{iz}^{'}r_{ix}^{'}
& m_ir_{iz}^{'}r_{iy}^{'}
& m_ir_{iz}^{'2}
\end{pmatrix}\tag{2-33}
$$

$$
\begin{aligned}
r_i^{'}r_i^{'T}&=
\begin{pmatrix}
r_{ix}^{'}\\
r_{iy}^{'}\\
r_{iz}^{'}
\end{pmatrix}
\begin{pmatrix}
r_{ix}^{'}&
r_{iy}^{'}&
r_{iz}^{'}
\end{pmatrix}\\
&=
\begin{pmatrix}
r_{ix}^{'2}
& r_{ix}^{'}r_{iy}^{'}
& r_{ix}^{'}r_{iz}^{'}\\
r_{iy}^{'}r_{ix}^{'}
& r_{iy}^{'2}
& r_{iy}^{'}r_{iz}^{'}\\
r_{iz}^{'}r_{ix}^{'}
& r_{iz}^{'}r_{iy}^{'}
& r_{iz}^{'2}
\end{pmatrix}
\end{aligned}\tag{2-34}
$$

设 $\textbf{1}$ 是一个 3x3 的单位矩阵：
$$
I(t)=\sum m_i(r_i^{'T}r_i^{'}\textbf{1}-r_i^{'}r_i^{'T})\tag{2-35}
$$
因为 $r_i(t)=R(t)r_{0i}+x(t),\;r'_i=R(t)r_{0i},\;R(t)R(t)^T=\textbf{1}$  
$$
\begin{aligned}
I(t)&=
\sum m_i(r_i^{'T}r_i^{'}\textbf{1}-r_i^{'}r_i^{'T})\\
&=\sum m_i
\bigg(\Big(R(t)r_{0i}\Big)^T
\Big(R(t)r_{0i}\Big)\textbf{1}-
\Big(R(t)r_{0i}\Big)\Big(R(t)r_{0i}\Big)^T\bigg)\\
&=\sum m_i\Big(r_{0i}^TR(t)^TR(t)r_{0i}\textbf{1}-R(t)r_{0i}r_{0i}^TR(t)^T\Big)\\
&=\sum m_i(r_{0i}^Tr_{0i}\textbf{1}-R(t)r_{0i}r_{0i}^TR(t)^T)
\end{aligned}\tag{2-36}
$$
由于 $r_{0i}^Tr_{0i}$ 是标量，所以可以有：
$$
\begin{aligned}
I(t)&=\sum m_i(r_{0i}^Tr_{0i}\textbf{1}-R(t)r_{0i}r_{0i}^TR(t)^T)\\
&=\sum m_i(R(t)r_{0i}^Tr_{0i}R(t)^T\textbf{1}-R(t)r_{0i}r_{0i}^TR(t)^T)\\
&=R(t)\bigg(\sum m_i\Big((r_{0i}^Tr_{0i})\textbf{1}-r_{0i}r_{0i}^T)\Big)\bigg)R(t)^T
\end{aligned}\tag{2-37}
$$
定义 $I_{body}$ 矩阵：
$$
I_{body}=\sum m_i((r_{0i}^Tr_{0i}\textbf{1}-r_{0i}r_{0i}^T))\tag{2-38}
$$

$$
I(t)=R(t)I_{body}R(t)^T\tag{2-39}
$$

幸运的是，由于 $I_{body}$ 是在**体坐标系**中定义的，因此在仿真过程中它是**恒定不变**的。正因如此，我们可以在仿真开始前预计算刚体的 $I_{body}$ 再结合当前的**朝向矩阵** $R(t)$，就能轻松计算出任意时刻 *t* 的惯性张量 $I(t)$。第 5.1 节将基于“对刚体在体坐标系中体积的积分”，推导**矩形物体**的体坐标系惯性张量。  
$I(t)$ 的逆由下面给出：
$$
\begin{aligned}
I^{-1}(t)&=(R(t)I_{body}R(t)^T)^{-1}\\
&=(R(t)^T)^{-1}I_{body}^{-1}R(t)^{-1}\\
&=R(t)I_{body}^{-1}R(t)^T
\end{aligned}\tag{2-40}
$$

因为 $R(t)^T=R(t)^{-1},\Big(R(t)\Big)^T=R(t).$ 显然，$I_{body}^{-1}$ 也是固定的。

#### 2.11  Rigid Body Equations of Motion

对于一个刚体我们定义 $\textbf{Y}(t)$:
$$
\textbf{Y}(t)=
\begin{pmatrix}
x(t)\\
R(t)\\
P(t)\\
L(t)
\end{pmatrix}\tag{2-41}
$$
刚体的状态由**空间信息**（位置与朝向）和**速度信息**（线动量与角动量）共同构成。刚体的质量 $M$ 与**体坐标系惯性张量** $I_{body}$ 是常量——我们假设在仿真启动时，这两个物理属性已预先确定。  
在任意给定时刻，辅助量 $I(t)$（世界坐标系惯性张量）、$\omega (t)$（角速度）与 $v(t)$（线速度）需通过计算得到:
$$
\begin{aligned}
& v(t)=\frac{P(t)}{M}\\
& I(t)=R(t)I_{body}R(t)^T\\
& \omega(t)=I(t)^{-1}L(t)
\end{aligned}\tag{2-42}
$$
导数 $\frac{\mathrm{d}}{\mathrm{d}t}\textbf{Y}(t)$:
$$
\begin{aligned}
\frac{\mathrm{d}}{\mathrm{d}t}\textbf{Y}(t)=
\frac{\mathrm{d}}{\mathrm{d}t}\begin{pmatrix}
x(t)\\
R(t)\\
P(t)\\
L(t)
\end{pmatrix}=
\begin{pmatrix}
v(t)\\
w(t)^*R(t)\\
F(t)\\
\tau(t)
\end{pmatrix}
\end{aligned}\tag{2-43}
$$
最后一点：与其用 $\textbf{Y}(t)$ 中的矩阵 $R(t)$ 来表示物体的朝向，不如使用四元数。第 4 节讨论了用四元数代替旋转矩阵。简单来说，四元数是一种包含四个元素的向量，可用于表示旋转。如果我们将 $\textbf{Y}(t)$ 中的 $R(t)$ 替换为四元数 $q(t)$，就可以将 $R(t)$ 视为一个辅助变量，从 $q(t)$ 直接计算 $R(t)$，就像从 $L(t)$ 直接计算 $\omega(t)$ 的方式。第 4 节会推导出一个类似于 $\dot R(t) = \omega(t)^*R(t)$ 的公式，该公式根据 $q(t)$ 和 $\omega(t)$ 表示 $\dot q(t)$。 

### 3. Computeing $\frac{\mathrm{d}}{\mathrm{d}t}\textbf{Y}(t)$

让我们考虑刚体的 dydt 函数的实现。代码是用c++编写的，我们假设我们有称为 matrix 和 triple 的数据类型（类），它们分别实现3 × 3矩阵和3空间中的点。  

~~~c++
/// 3x3
class matrix
{
    
}

/// 3
class triple
{
    
}

class RigidBody
{
    // constant quantities
    double mass;
    matrix Ibody;
    matrix IbodyInv;
    
    // state variables
    triple x;
    matrix R;
    triple P;
    triple L;
    
    // derived quantities
    matrix Iinv;
    triple v;
    triple omega;
    
    // computed quantities
    triple force;
    triple torque;
}

RigidBody Bodies[NBODIES];
~~~

假设在模拟开始之前，已经为数组`Bodies`的每个成员计算了常量`mass`， `Ibody`和`IbodyInv`。同样，每个刚体的初始条件通过赋值给的 `Bodies` 每个成员的状态变量`x`、`R`、`P`和`L`来指定。本节中的实现用旋转矩阵表示方向；第4节描述了用四元数表示方向所必需的更改。我们通过传递实数数组与微分方程求解器进行通信。  
需要几个函数：  

~~~c++
// copy body state information to an array
void State_to_Array(RigidBody *rb, double *y)
{
    *y++ = rb->x[0];
    *y++ = rb->x[1];
    *y++ = rb->x[2];
    
    for(int i = 0; i < 3; i++)
    {
        for(int j = 0; j < 3; j++)
        {
            *y++ = rb->R[i,j];
        }
    }
    *y++ = rb->P[0];
    *y++ = rb->P[1];
    *y++ = rb->P[2];
    
    *y++ = rb->L[0];
    *y++ = rb->L[1];
    *y++ = rb->L[2];
}

// Copy information from an array to RigidBody
void Array_to_State(RigidBody *rb, double *y)
{
    rb->x[0] = *y++;
    rb->x[1] = *y++;
    rb->x[2] = *y++;
    
    for(int i = 0; i < 3; i++)
    {
        for(int j = 0; j < 3; j++)
        {
            rb->R[i,j] = *y++;
        }
    }
    rb->P[0] = *y++;
    rb->P[1] = *y++;
    rb->P[2] = *y++;
    
    rb->L[0] = *y++;
    rb->L[1] = *y++;
    rb->L[2] = *y++;
    
    // 原文就这么写的，我也不知道哪来的，伪代码看一乐吧
    rb->v = rb->P / mass;
    
    rb->Iinv = R * IbodyInv * Transpose(R);
    
    rb->omega = rb->Iinv * rb->L;
}
~~~

检查这些例程，我们看到每个刚体的状态由3+9+3+3 = 18个数字($mass\; and\; I_{body}:\textbf{fxxk}$)表示。body的所有成员和大小为18·NBODIES的数组y之间的传输。  
Compute_Force_and_Torque考虑了所有的力和扭矩：重力，风，与其他物体的相互作用等。

开摆了这部分都是程序设计没什么好说的

### 4. Quaternions vs. Rotation Matrix

对于刚体模拟，避免使用旋转矩阵的最重要原因是数值漂移问题。假设我们根据公式跟踪刚体的方向$\dot R(t)=\omega^*R(t)$  
当我们用这个公式更新 $R(t)$ 时（也就是说，当我们对这个方程积分时），我们将不可避免地遇到数值漂移。数值误差将在R(t)的系数中建立，因此$R(t)$ 将不再是精确的旋转矩阵。从图形上看，效果是将 $R(t)$ 应用于物体会导致扭曲效果。  
在刚体模拟中，使用单位四元数表示旋转可缓解数值漂移问题。四元数仅需四个参数，仅需一个额外变量即可描述旋转的三个自由度。相比之下，旋转矩阵用九个参数描述三个自由度，冗余度显著高于四元数。因此，四元数的漂移程度远低于旋转矩阵。若必须修正四元数的漂移，那是因为它已失去单位幅值特性。可通过将四元数重新归一化为单位长度轻松修正。由于这两大特性，直接用单位四元数 $q(t)$ 表示物体方向最为理想。我们仍将角速度表示为向量 ω(t)。计算惯性张量 $I^{-1}(t)$ 所需的方向矩阵 $R(t)$，将作为辅助变量由 $q(t)$ 直接推导得出。  
我们把四元数 $s+v_xi+v_yj+v_zk$ 写成数对: $[s,v]$  
$$
[s_1,v_1][s_2,v_2]=[s_1s_2-v_1\cdot v_2,s_1v_2+s_2v_1+v_1\times v_2]\tag{4-1}
$$
绕单位轴 $u$ 旋转 $\theta$ 表示为: $[\cos(\frac{\theta}{2}),\sin(\frac{\theta}{2})u].$  
在刚元数表示旋转时，若 $q_1$ 和 $q_2$ 分别表示旋转，则 $q-2q_1$ 表示先执行 $q_1$ 旋转、再执行 $q_2$ 旋转的复合旋转。稍后我们将展示如何修改第 3 节的例程以适配四元数的方向表示。但在修改前，需先推导 $\dot q(t)$ 的公式，附录 B 推导了该公式：  
$$
\dot q(t)=\frac{1}{2}\omega(t)q(t)\tag{4-2}
$$
这里 $\omega(t)q(t)$ 是 四元数 $[0, \omega(t)]$ 和 $q(t)$ 相乘的简写。  
这个等式看着有些眼熟，不是吗？  
$\dot R(t)=\omega(t)^*R(t)$  
使用四元数我们需要重新设计 RigidBody：  

~~~c++
class RigidBody
{
    // constant quantities
    double mass;
    matrix Ibody;
    matrix IbodyInv;
    
    // state variables
    triple x;
    quaternion q;
    triple P;
    triple L;
    
    // derived quantities
    matrix Iinv;
    matrix R;
    triple v;
    triple omega;
    
    // computed quantities
    triple force;
    triple torque;
}

rb->R = quaternion_to_matrix(normalize(rb->q));
~~~

函数 `normalize` 返回被 $q$ 除以 长度 的归一化结果。单位长度的四元数再传给 `quaternion_to_matrix` ，返回 3x3 矩阵，如 $q=[s,v]$ 则返回的矩阵为  
$$
\begin{pmatrix}
1-2v_y^2-2v_z^2
& 2v_xv_y-2sv_z
& 2v_xv_z+2sv_y\\
2v_xv_y+2sv_z
& 1-2v_x^2-2v_z^2
& 2v_yv_z-2sv_x\\
2v_xv_z-2sv_y
& 2v_yv_z+2sv_x
& 1-2v_x^2-2v_y^2
\end{pmatrix}
$$
从旋转矩阵到四元数  
~~~C++
quaternion matrix_to_quaternion(const matrix &m)
{
    quaternion q;
    double tr,s;
    
    tr = m[0,0] + m[1,1]+m[2,2];
    if(tr >= 0)
    {
        s = sqrt(tr + 1);
        q.r = 0.5 * s;
        s = 0.5 / s;
        q.i = (m[2,1] - m[1,2]) * s;
        q.j = (m[0,2] - m[2,0]) * s;
        q.k = (m[1,0] - m[0,1]) * s;
    }
    else
    {
        int i = 0;
        if(m[1,1] > m[0,0])
            i = 1;
        if(m[2,2] > m[i,i])
            i = 2;
        
        switch(i)
        {
            case 0:
                s = sqrt((m[0,0]-(m[1,1] + m[2,2])) + 1);
                q.i = 0.5 * s;
                s = 0.5 / s;
                q.j = (m[0,1] + m[1,0]) * s;
				q.k = (m[2,0] + m[0,2]) * s;
				q.r = (m[2,1] - m[1,2]) * s;
                break;
            case 1:
				s = sqrt((m[1,1] - (m[2,2] + m[0,0])) + 1);
				q.j = 0.5 * s;
				s = 0.5 / s;
				q.k = (m[1,2] + m[2,1]) * s;
				q.i = (m[0,1] + m[1,0]) * s;
				q.r = (m[0,2] - m[2,0]) * s;
				break;
			case 2:
				s = sqrt((m[2,2] - (m[0,0] + m[1,1])) + 1);
                q.k = 0.5 * s;
                s = 0.5 / s;
                q.i = (m[2,0] + m[0,2]) * s;
                q.j = (m[1,2] + m[2,1]) * s;
                q.r = (m[1,0] - m[0,1]) * s;
        }
    }
    return q;
}
~~~

矩阵m的结构使m[0,0], m[0,1]和m[0,2]构成m的第一行（而不是列）。  
例程Array_to_Bodies和Bodies_to_Array根本不需要任何更改，但请注意常量STATE_SIZE从18更改为13，因为四元数需要的元素比旋转矩阵少5个。我们只需要在ddt_State_to_Array中进行其他更改

### 5. Examples

#### 5.1 Inertia Tensor of a Block

![](/imgs/Rigid Body Simulation Ⅰ/Figure7.png)  
计算图中 $x_0\times y_0\times z_0$ 的长方形的惯性张量 $I_{body}$  我们将质心设为原点，因此长方体的 x 轴 $[-\frac{x_0}{2},\frac{x}{2}]$，yz同理。  
我们需要知道质量才能计算，$\rho(x_i,y_i,z_i)dxdydz=m_i$  因为物体密度恒定，所以质量就是 $M=\rho x_0y_0z_0$  
$$
\begin{aligned}
I_{xx}&=\int_{-\frac{x_0}{2}}^{\frac{x_0}{2}}
\int_{-\frac{y_0}{2}}^{\frac{y_0}{2}}
\int_{-\frac{z_0}{2}}^{\frac{z_0}{2}}
\rho(x,y,z)(y^2+z^2)
\mathrm{d}z\mathrm{d}y\mathrm{d}x\\
&=\frac{M}{12}(y_0^2+z_0^2)\\
I_{yy}&=\frac{M}{12}(x_0^2+z_0^2)\\
I_{zz}&=\frac{M}{12}(x_0^2+y_0^2)
\end{aligned}\tag{5-1}
$$
计算 $I_{xy}$(根据 2-32 式)  
$$
\begin{aligned}
I_{xy}&=\int_{-\frac{x_0}{2}}^{\frac{x_0}{2}}
\int_{-\frac{y_0}{2}}^{\frac{y_0}{2}}
\int_{-\frac{z_0}{2}}^{\frac{z_0}{2}}
-\rho(x,y,z)(xy)
\mathrm{d}z\mathrm{d}y\mathrm{d}x\\
&=0
\end{aligned}\tag{5-2}
$$
有  
$$
\begin{aligned}
I_{body}=
\frac{M}{12}
\begin{pmatrix}
y_0^2+z_0^2
& 0
& 0\\
0
& x_0^2+z_0^2
& 0\\
0
& 0
& x_0^2+y_0^2
\end{pmatrix}
\end{aligned}\tag{5-3}
$$

#### 5.2 A Uniform Force Field

假设一个均匀的力作用在物体的每一个粒子上。例如，我们通常将引力场描述为对刚体的每个粒子施加一个力，其中 $g$ 是指向向下的矢量。重力作用在物体上的合力 $F_g$ 是  
$$
F_g=\sum m_ig=Mg\tag{5-4}
$$
扭矩：  
$$
\sum(r_i(t)-x(t))\times m_ig=\textbf{0}\tag{5-5}
$$
这里用了一个式子 2-20。由此可见，均匀的重力场对物体的角动量没有影响。这里引力场可以被视为一个**单一合力**，作用于物体的**质心**处，大小等于 $Mg$(其中 $M$ 是物体质量，$g$ 是重力加速度)。

#### 5.3 Rotation Free Movement of a Body

让我考虑一下图中所示的两个力  
![](/imgs/Rigid Body Simulation Ⅰ/Figure8.png)  外力 $F=(0,0,f)$分别作用在 $x(t)+(-3,0,-2)\;,\;x(t)+(3,0,-2)$  
我们直观的想到这只会有线性加速度，而非角加速度。  
合力为 $(0,0,2f)$，加速度 $\frac{2f}{M}$ 沿着 z 轴  
有两力矩  
$$
\begin{aligned}
&(x(t)+
\begin{pmatrix}
-3\\0\\-2
\end{pmatrix}-x(t))\times F=
\begin{pmatrix}
-3\\0\\-2
\end{pmatrix}\times F\\
&(x(t)+
\begin{pmatrix}
3\\0\\-2
\end{pmatrix}-x(t))\times F=
\begin{pmatrix}
3\\0\\-2
\end{pmatrix}\times F\\
&\tau=
\begin{pmatrix}
-3\\0\\-2
\end{pmatrix}\times F +
\begin{pmatrix}
3\\0\\-2
\end{pmatrix}\times F=
\begin{pmatrix}
0\\0\\-2
\end{pmatrix}\times F\\
&=
\begin{pmatrix}
0\\0\\-2
\end{pmatrix}\times
\begin{pmatrix}
0\\0\\f
\end{pmatrix}
=\textbf{0}
\end{aligned}
$$

#### 5.4 Translation Free Movement of a Body

现在假设外力 $F_1=(0,0,f)$作用在物体的点 $x(t)+(-3,0, -2)$上，外力 $F_2=(0,0,-f)$ 作用在物体的点 $x(t)+(3,0,2)$上。由于F1 =−F2，作用在物体上的合力是 $F_1+F_2=0$​，所以没有质心加速度。  
![](/imgs/Rigid Body Simulation Ⅰ/Figure9.png)  另一方面，扭矩合是  
$$
\begin{aligned}
&(x(t)+
\begin{pmatrix}
-3\\0\\-2
\end{pmatrix}-x(t))\times F_1
+
(x(t)+
\begin{pmatrix}
3\\0\\-2
\end{pmatrix}-x(t))\times F_2
=&\\
&\begin{pmatrix}
-3\\0\\-2
\end{pmatrix}\times
\begin{pmatrix}
0\\0\\f
\end{pmatrix}
+
\begin{pmatrix}
3\\0\\-2
\end{pmatrix}\times 
\begin{pmatrix}
0\\0\\-f
\end{pmatrix}
=\\
&
\begin{pmatrix}
0\\3f\\0
\end{pmatrix}
+
\begin{pmatrix}
0\\3f\\0
\end{pmatrix}=
\begin{pmatrix}
0\\6f\\0
\end{pmatrix}
\end{aligned}
$$
因此，合扭矩为$(0,6f,0)$，与y轴平行。最终的结果是，作用在物体上的力使物体沿 y轴 呈角加速。

#### 5.5 Force vs. Torque Puzzle

在分析一个力作用于物体某点所产生的影响时，有时会觉得这个力**仿佛被重复考量了两次**。也就是说，若力 $F$ 作用于空间中某点 $r+x(t)$ 处的物体，我们会先将其视为**加速质心**的力，再将其视为**赋予物体自旋**的力。  
![](/imgs/Rigid Body Simulation Ⅰ/Figure10.png)  
这催生出一个看似矛盾的情境：观察上图中的长水平木块（初始静止）。假设力 $F$ 作用于木块的**质心**并持续一段时间（比如10秒)。由于力作用于质心，**对物体不产生力矩**。10秒后，物体将获得一定**线速度** *v*；但物体不会获得任何**角速度**，因此木块的**动能**为 $\frac{1}{2}M|v|^2$。  
![](/imgs/Rigid Body Simulation Ⅰ/Figure11.png)  现在假设相同的力 $F$ **偏置**作用于物体（如图11所示）。由于作用于物体的力大小未变，**质心的加速度也相同**。因此，10秒后，物体将再次获得线速度 $v$。然而，由于力 F 偏置作用，此时会对物体**产生力矩**，10秒后，物体将获得一定的**角速度** $\omega$。此时物体的动能（见附录C)为:  
$$
\frac{1}{2}M|v|^2+\frac{1}{2}\omega^TI\omega
$$
木块的动能**比力作用于质心时更高**。但若两次作用的是**完全相同的力**，木块的动能为何会不同？
提示：能量（或功）是**力对位移的积分**（即力沿作用路径累积的效果）。  
![](/imgs/Rigid Body Simulation Ⅰ/Figure12.png)  图12展示了**偏置力为何会产生更高的动能**：木块的动能等于**力所做的功**，而功是**力对位移的积分**（即力沿作用路径累积的效果）。  
在图11中，力**偏置作用于质心外**时，需关注**力作用点的运动轨迹**——该轨迹明显长于图10中**质心的运动轨迹**。因此，当力偏置作用时，**力作用点（记为p点）描出的路径更长**，导致力做的功更多（因为功的本质是力对作用点位移的累积)。

## Reference
[Physically Based Modeling](https://www.cs.cmu.edu/~baraff/sigcourse/index.html)  
[Unconstrained Rigid Body Dynamics](https://www.cs.cmu.edu/~baraff/sigcourse/notesd1.pdf)  


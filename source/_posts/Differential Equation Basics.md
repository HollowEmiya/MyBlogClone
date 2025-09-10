---
title: Differential Equation Basics
math: true
tags: [Math]
index_img: 
banner_img: 
data: 2025--08--12 22:09:48
typora-root-url: ./..
---
微分方程描述的是**未知函数**及**未知函数的导数**之间的关系。
<!--more-->
# Differential Equation Basics

## 1. Initial Value Problems

微分方程描述的是**未知函数**及**未知函数的导数**之间的关系。解微分方程的过程就是找到满足该关系的函数，该函数通常也需要满足一些其他条件。在这个课程中，我们将关注一类特殊问题——**初值问题(Initial Value Problems)**。在经典的初值问题中，系统行为以如下的**常微分方程(**ordinary differential equation - ODE**)**表示
$$
\dot{\textbf{x}}=f(\textbf{x},t)
$$
函数 f 是已知的，可以通过给定 $\textbf{x},t$ 进行计算，$\textbf{x}$ 表示系统的状态，$\dot{\textbf{x}}$ 是 $\textbf{x}$ 在时间上的导数。值得注意的是，$\textbf{x}$ 和 $\dot{\textbf{x}}$ 都是矢量(vector)。正如 初始值问题 的名字一样，我们能得到在 $t=t_0$ 时 $\textbf{x}(t_0)=\textbf{x}_0$，并且追踪 $\textbf{x}$ 随着时间的变化。

一般的初值问题很好可视化。在 2D 平面中，$\textbf{x}(t)$ 绘制了一条描述点 $\textbf{p}$ 在平面上的运动轨迹。在任一点 $\textbf{x}$ 方程 $f$ 可以求得一个二维向量，所以可以认为 $f$ 在该平面定义了一个方向场(figure 1.)。在 $\textbf{x}$ 的向量是 移动点 $\textbf{p}$ 如果经过 $\textbf{x}$ 必须拥有的速度，当然 $\textbf{p}$ 可能经过该点也可能不会。$f$ 就像一股洋流使得 移动点 $\textbf{p}$ 从一点移动到零一点。无论我们最初把 $\textbf{p}$ 存放在何处，该点的“电流”都会将其捕获。$\textbf{p}$ 的位置取决于我们最初把它扔到哪里，但是一旦扔到哪里，所有未来的运动都是由 $f$ 决定的。$f$ 驱动点 $\textbf{p}$ 产生的移动轨迹构成了向量场的积分曲线如 Figure 2 。  
积分曲线也就是方程的解，其上的每一点斜率都是 $f$ 的 向量。

我们将 $f$ 写作 $\textbf{x}$ 和 $t$ 的方程，但是导数函数可能依赖时间也可能不会。如果导数函数真的和时间相关，点 $\textbf{p}$ 不仅会运动，整个向量场都会运动，这样 $\textbf{p}$ 的速度不仅取决于他的位置，还取决于他什么时候到达该位置。在这种情况下，导数 $\dot{\textbf{x}}$ 在两个方面上受时间影响：首先，导数向量本身随时间变化。其次，因为点 $\textbf{p}$ 沿着轨迹 $\textbf{x}(t)$ 运动，会在不同时间表现出不同的导数向量。可以想象一个粒子随着起伏变化的向量场而浮动就像起落的潮水……我的爱如潮水\~爱如潮水将我向你推\~

![](/imgs/Differential Equation Basics/Figure1.png)  
![](/imgs/Differential Equation Basics/Figure2.png)

## 2. Numerical Solutions

初级的微分方程主要关注其解析解，即猜测其未知函数的函数形式。比如一个微分方程为 $\dot{x}=-kx$，解即为$x=e^{-kt}$.

相比之下，我们关注的是数值解，即从初始值 $\textbf{x}(t_0)$ 开始以时间为步进进行计算。为了进行步进，我们需要通过导函数 $f$ 计算在时间间隔 $\Delta t$ 内 $\textbf{x}$ 的变化 $\Delta\textbf{x}$，然后在 $\textbf{x}$ 累加 $\Delta\textbf{x}$ 获得新的函数值。在计算数值解时，导函数 $f$ 视作一个黑盒，我们提供 $\textbf{x},t$ 的数值，然后得到 $\dot{\textbf{x}}$ 的”数值“值。  
数值方法是通过在每个时间步长内进行一次或多次这些导数的计算来运作的。

### 2.1. Euler's Method

最简单的数值方法被称为欧拉法。我们将 $\textbf{x}$ 的初始值记作 $\textbf{x}_0 = \textbf{x}(t0)$，而我们对下一时刻 $t_0 + h$ 时 $\textbf{x}$ 的估计记作 $\textbf{x}(t_0 + h)$，其中 $h$ 是一个步长参数。欧拉法只是通过沿导数方向步进一步来计算 $\textbf{x}(t_0 + h)$。  
$$
\textbf{x}(t_0+h)=x_0+h\dot{\textbf{x}}(t_0)
$$
可以借助二维向量场的图像来直观理解欧拉方法。与真实的积分曲线不同，点 $\textbf{p}$ 会沿着一个多边形路径移动，每条边的长度由在起点处计算向量 $f$ 的值，并乘以步长 $h$ 来确定。请参见 Figure.3 自下而上表示了不同的步进精度。  
![](/imgs/Differential Equation Basics/Figure3.png)  尽管这种方法简单，但并不精确。考虑一个二维函数 $f$ 的情况，其积分曲线是同心圆。由 $f$ 控制的点 $\textbf{p}$ 应该永远在它开始所在的圆上绕圈运动。然而，每次使用欧拉法进行一步运算时，点 $\textbf{p}$ 都会沿着一条直线移动到半径更大的圆上，因此其路径会形成向外的螺旋状。减小步长可以减缓这种向外漂移的速度，但永远无法消除它。  
此外，欧拉方法实际上是不稳定的。以一个一维函数为例 $f=-kx$，应该使得点 $\textbf{p}$ 以指数级速度下降至0。如果步长足够小我们能得到合理的结果，但当 $h>\frac{1}{k}$，则 $\Delta x>|x|$，所以结果会在零附近来回震荡，如果超过 $h=\frac{2}{k}$，震荡将会发散，系统崩溃。  
![](/imgs/Differential Equation Basics/Figure4.png)  
最后，欧拉方法甚至都不算高效。大多数数值求解方法几乎都将全部时间用于进行导数计算，因此每一步的计算成本取决于每一步的计算次数。尽管欧拉方法每一步只需进行一次计算，但方法的实际效率取决于它允许的步长大小（在保证精度和稳定性的同时)以及每一步的计算成本。更为复杂的算法，甚至有些每一步需要多达四到五次计算，都能大大优于欧拉方法，因为它们每一步的高计算成本被它们允许的更大步长所抵消。  
要了解我们如何改进欧拉方法，我们需要更仔细地研究该方法所产生的误差。理解其原理的关键在于泰勒级数：
假设函数 $\textbf{x}(t)$ 是连续的，那么我们可以将其在该步进结束时的值，表示为一个初始时刻的函数值与各阶导数的无穷级数和：  
$\textbf{x}(t_0+h)=\textbf{x}(t_0)+h\dot{\textbf{x}}(t_0)+\frac{h^2}{2!}\ddot{\textbf{x}}(t_0)+\frac{h^3}{3!}\dddot{\textbf{x}}(t_0)+...+\frac{h^n}{n!}\frac{\delta ^n{\textbf{x}}}{\delta t^n}$  
对上式进行截断只保留前两项就得到了欧拉更新公式。也就是说只有除第一个带导函数的子项外，其余导数全都为零欧拉公式才是正确的，也就是说只有在 $\textbf{x}(t)$ 线性时欧拉公式才正确。在欧拉公式和完整的泰勒级数式间的误差，主要由 $\frac{h^2}{2!}\ddot{\textbf{x}}(t_0)$ 决定。因此，我们可以将误差项描述为 $O(h^2)\;\;Order \; h\; squared$ 。假设我们将步长减半即 $\frac{h}{2}$ ，这样的误差会缩小 1/4，但是给定的时间间隔内计算量会达到两倍。这意味着在时间间隔$t_0$ 和 $t_1$ 之间的计算误差和 $h$ 成线性相关。理论上我们可以挑选一个合适大小的 $h$ 来计算 $t_0$ 和 $t_1$ 间的 $\textbf{x}$ 值，并保证较小的误差，可实际计算过程中根据误差和 $f$ 我们可能必须使用较大的步进。

### 2.2.The Midpoint Method

如果我们能求 $\dot{\textbf{x}}$ 和 $\ddot{\textbf{x}}$ ，就能实现 $O(h^3)$ 级别的误差：  
$$
\textbf{x}(t_0+h)=\textbf{x}(t_0)+h\dot{\textbf{x}}(t_0)+\frac{h^2}{2!}\ddot{\textbf{x}}(t_0)+O(h^3)\tag{1}
$$

时间导数 $\dot{\textbf{x}}$ 是函数 $f(\textbf{x}(t),t)$ ，为了简化情况，我们假设函数 $f$ 从 $\textbf{x}$ 间接受到时间影响，所以 $\dot{\textbf{x}}=f(\textbf{x}(t))$，根据链式法则：  
$$
\begin{aligned}
&chain\;rule:f(g(x))'=f'[g(x)]g'(x)\\
&\ddot{\textbf{x}}=f(\textbf{x}(t))'=f'(\textbf{x}(t))\textbf{x}'(t)=\frac{\delta f}{\delta\textbf{x}}\dot{\textbf{x}}=f'f\\
\end{aligned}
$$
为了避免可能计算 $f'$ 出现复杂的计算，我们可以仅用 $f$ 来近似二阶项，并将该近似值带入到 等式1，保留 $O(h^3)$ 的误差，为此我们再次进行泰勒展开：  
$$
f(\textbf{x}_0,\Delta\textbf{x})=f(\textbf{x}_0)+\Delta\textbf{x}f'(\textbf{x}_0)+O(\Delta\textbf{x}^2)\tag{2}
$$
我们引入 $\ddot{\textbf{x}}$ 通过等式:  
$$
\begin{aligned}
&\Delta\textbf{x}=\frac{h}{2}f(\textbf{x}_0)\\
&f(\textbf{x}_0+\frac{h}{2}f(\textbf{x}_0))=
f(\textbf{x}_0)+
\frac{h}{2}f(\textbf{x}_0)f'(\textbf{x}_0)+
O(h^2)\\
&=f(\textbf{x}_0)+
\frac{h}{2}\ddot{\textbf{x}}(t_0)+O(h^2)\\
&这里\;\textbf{x}_0=\textbf{x}(t_0),
我们两边乘上h,使\;O(h^2)\;为\;O(h^3)\\
&h[f(\textbf{x}_0+\frac{h}{2}f(\textbf{x}_0))]=
hf(\textbf{x}_0)+
\frac{h^2}{2}\ddot{\textbf{x}}(t_0)+O(h^3)\\
&\frac{h^2}{2}\ddot{\textbf{x}}+O(h^3)=
h[f(\textbf{x}_0+\frac{h}{2}f(\textbf{x}_0))-f(\textbf{x}_0)]\\
&带入式子1,\\
&\textbf{x}(t_0+h)=
\textbf{x}(t_0)+h\dot{\textbf{x}}(t_0)+
[\frac{h^2}{2!}\ddot{\textbf{x}}(t_0)+O(h^3)]\\
&\textbf{x}(t_0+h)=\textbf{x}(t_0)+h\dot{\textbf{x}}(t_0)+h[f(\textbf{x}_0+\frac{h}{2}f(\textbf{x}_0))-f(\textbf{x}_0)]\\
&\textbf{x}(t_0+h)=\textbf{x}(t_0)+h[f(\textbf{x}_0+\frac{h}{2}f(\textbf{x}_0))]
\end{aligned}
$$
该方法先计算一步欧拉公式，然后在该步进中点进行二阶导计算，用中点值更新 $\textbf{x}$ ，即中点法。  
![Figure5](/imgs/Differential Equation Basics/Figure5.png)   
中点数值方法精度在 $O(h^3)$ 但是需要进行两次 $f$ 的计算。  
我们不必止步于 $O(h^3)$ 这种误差。通过对函数 $f$ 进行更多次的计算，我们可以消除更高阶的导数项。这种做法最常用的方法被称为 4 阶 Runge-Kutta方法，其每一步的误差为 $O(h^5)$。（中点法可以被称为 2 阶 Runge-Kutta 方法。）我们不会推导出4阶 Runge-Kutta 方法，但计算 $\textbf{x}(t_0 + h)$ 的公式如下所示：  
$$
k_1&=&hf(\textbf{x}_0,t_0)\\
k_2&=&hf(\textbf{x}_0+\frac{k_1}{2},t_0+\frac{h}{2})\\
k_3&=&hf(\textbf{x}_0+\frac{k_2}{2},t_0+\frac{h}{2})\\
k_4&=&hf(\textbf{x}_0+k_3,t_0+h)\\
\textbf{x}(t_0+h)&=&\textbf{x}_0+
\frac{1}{6}k_1+\frac{1}{3}k_2+
\frac{1}{3}k_3+\frac{1}{6}k_4
$$
## Adaptive Stepsizes

无论我们使用哪种方法，一个核心问题就是步进大小的选择。理想情况我们希望有一个合理的步进尺寸，尽可能的大来减少计算，但不要太大以免带来额外的误差，或者导致函数不稳定。如果我们选择的步长固定，我们只能受限于 $\textbf{x}(t)$ 最差的区域所允许的步进速度。我们希望的是在随时间变化时动态选择 $h$ 。只要我们能使 $h$ 变大而不产生太多误差，我们就应该这样做。当需要减少 $h$ 以避免过多的误差时，我们也想这样做。这就是自适应步进调整的思想：在求解ODE的过程中改变 $h$ 。  
在这里，我们将介绍欧拉方法的自适应步长。其基本思想如下。假设我们有一个给定的步长 $h$，我们想知道我们可以考虑改变多少步长。  
假设我们计算 $\textbf{x}(t_0 + h)$ 的两个估计值。我们计算一个估计值 $\textbf{x}_a$ ，一个欧拉步进，步进大小为 $h$，范围从 $t_0$ 到 $t_0+h$ ，再计算另一个估计值 $\textbf{x}_b$ ，两次欧拉步进，其步进大小为 $\frac{h}{2}$，范围从 $t_0$ 到 $t_0+h$ 。$\textbf{x}_a$ 和 $\textbf{x}_b$ 和实际的 $\textbf{x}(t_0+h)$ 误差都在 $O(h^2)$，那意味着 $\textbf{x}_a$ 和 $\textbf{x}_b$ 的差值也在  $O(h^2)$。我们能确定这个正确的误差 $e$：$e=|\textbf{x}_a-\textbf{x}_b|$  
这为我们计算欧拉步长为 $h$ 时的误差提供了一个方便的估算值。假设我们希望每步误差为 $10^{−4}$，而当前误差仅为 $10^{−8}$。由于误差是随 $h^2$ 变化，所以我们可以提升步长到: $(\frac{10^{−4}}{10^{−8}})^\frac{1}{2}h=100h$。相反如果我们实际误差为 $10^{-3}$，我们应降低步进为: $(\frac{10^{−4}}{10^{-3}})^\frac{1}{2}\approx.316h$。  
强烈推荐使用自适应步进。

## Implementation

我们试图解决的常微分方程 (ODE) 可能能描述很多问题、情况——比如，一组有质量的物体和弹簧，一些刚体，或者一个可形变的物体。我们希望以一种方式实现 ODE 的求解器和模型，使得求解器和模型之间相互隔离，不会暴露各自的内部细节。这样能够方便的修改求解器并复用求解器的代码。好消息是这种设计并不难以实现，因为所有求解器都能被表示为一小组标准化操作。可以推测，受常微分方程支配的对象系统会以某种数据结构的形式存在。我们的实现思路是：先编写针对特定类型的代码来操作该数据结构、执行标准操作，再基于这些通用操作来实现求解器。  
从求解器的视角来看，它所操作的方程组是一个黑盒函数 $f(\textbf{x},t)$。求解器需要能够在 $\textbf{x}$ 和 $t$ 的任意取值下，按需计算 $f$ 的值；随后在推进时间步时，将更新后的 $\textbf{x}$ 和 $t$ 植入系统。为支持这些操作，代表待求解常微分方程的对象必须能够响应求解器的如下请求：  

* 返回 $dim(\textbf{x})$ 。因为 $\textbf{x}$ 和 $\dot{\textbf{x}}$ 可能是向量，求解器必须知道其长度来分配内存空间，执行向量计算。
* 获取、设置 $\textbf{x},t$。求解器需能在时间步结束时植入新值。此外，**多步法**在开展导数计算过程中，必须将 $\textbf{x}$ 和 $t$ 设置为中间值。
* 能计算在当前 $\textbf{x},t$ 计算函数 $f$ 值。

在**面向对象语言**中，这些操作会天然以**通用函数**的形式实现，并通过**类型专属**的方式处理。在**非面向对象语言**中，“通用函数”会通过以下方式模拟：在结构体槽位中植入指向类型专属函数的指针，或直接将函数指针作为参数传递给求解器。稍后我们将详细探讨，这些操作如何在粒子 - 弹簧系统等特定模型中落地实现。

## Reference

[Physically Based Modeling](https://www.cs.cmu.edu/~baraff/sigcourse/index.html)

[Differential Equation Basics](https://www.cs.cmu.edu/~baraff/sigcourse/notesb.pdf)

[游戏物理引擎(一) 刚体动力学 - 知乎](https://zhuanlan.zhihu.com/p/109532468)

[Video Game Physics Tutorial - Part I: Rigid Body Dynamics | Toptal®](https://www.toptal.com/game/video-game-physics-part-i-an-introduction-to-rigid-body-dynamics)

[Publications :: Box2D](https://box2d.org/publications/)
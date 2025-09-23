---
title: Rigid Body Simulation Ⅱ
math: true
tags: [Math]
index_img: 
banner_img: 
data: 2025--08--19 22:09:48
typora-root-url: ./..
---

刚体模拟 Ⅱ。  
<!--more-->

# Rigid Body Simulation Ⅱ

## Part Ⅱ. Nonpenetration Constraints

### 6. Problems of Nonpenetration Constraints

现在我们知道了如何写出和实现刚体的运动方程，让我们考虑一下如何防止物体移动时他们在环境中互相穿透的问题。为简化讨论，假设我们模拟一个**质点**（即单个粒子）落到固定地面的过程。这里面涉及几个关键问题。  现在我们处理的是**完全不可变形的刚体**，因此当粒子撞击地面时，**绝不允许任何穿透**。（若将地面视为柔性体，可能允许粒子轻微穿透，并将其解释为地面在撞击点附近的形变。但这里地面是刚性的，因此**完全不允许穿透**。）这意味着：当粒子**实际接触地面瞬间**，我们需要**突然改变粒子的速度**——这与柔性体（如橡胶球）的处理方式截然不同。  
对于柔性体，碰撞可视为**渐进过程**：在**较短但不为零的时间跨度**内，球与地面间会产生作用力，使球的速度逐渐变化；同时球因该力发生形变。球越刚性，形变越小，碰撞过程越快。**极限情况**下，球为**绝对刚性**（完全无形变），若速度不能**瞬时**停止，球就会穿透地面。  
因此，在**刚体动力学**中，我们默认将碰撞视为**瞬时事件**（持续时间为零）。

这意味着我们需要处理**两种接触类型**。

* 当两个物体在点 $p$ 处接触，且存在相互靠近的速度（如粒子撞击地面），我们称之为**碰撞接触**。碰撞接触需要速度的**瞬时变化**。每当碰撞发生时，描述物体运动的状态（包含位置和速度）会发生**速度的不连续变化**。而求解常微分方程（ODE）的数值程序，默认基于“状态 $Y(t)$ 始终**平滑变化**”的假设运行。显然，碰撞时要求 $Y(t)$ 突变，直接违反了这一假设。  
  我们通过以下方式解决这个问题：若碰撞发生在时间 $t_c$，我们让常微分方程（ODE）求解器**停止运行**。接着，我们获取该时刻的状态 $Y(t_c)$，并计算碰撞涉及物体的速度需如何改变。我们将反映这些新速度的状态记为 $Y(t_c)^+$。需注意： $Y(t_c)$ 与 $Y(t_c)^+$ 在**所有空间变量**（位置和姿态）上一致，但在 $t_c$ 时刻参与碰撞的物体**速度变量**上存在差异。随后，我们以新状态 重 $Y(t_c)^+$ 启数值求解器，并指示它从时间 $t_c$ 开始继续模拟。

* 当物体在点 $p$ 处**相互静止接触**（例如：粒子以零速度接触地面），我们称物体处于**静息接触**状态。此时，我们需要计算一个**阻止粒子向下加速的力**——本质上，这个力等于粒子受重力的重量（或其他施加于粒子的外部力）。我们将粒子与地面的力称为**接触力**。静息接触显然不需要我们在每一瞬间都停止并重启 ODE 求解；从 ODE 求解器的视角看，接触力只是 `Compute_Force_and_Torque` 函数返回的力的一部分。

![](/imgs/Rigid Body Simulation Ⅱ/figure13.png)  
到目前为止，我们需要处理两个问题：计算碰撞接触时的速度变化，以及计算防止穿透现象的接触力。但在解决这些问题之前，我们必须先处理几何问题，即实际检测物体之间的接触。让我们回到粒子下落至地面的例子。在运行模拟时，我们会计算粒子在特定时间点朝地面下落的位置（见图13）。假设我们考虑粒子在时刻 $t_0,t_0+\Delta t,t_0+2\Delta t$ 等位置，且碰撞时刻 tc（粒子实际撞击地面的时刻)位于 $t_0$ 和 $t_0+\Delta t$ 之间。那么在 $t_0$ 时刻，我们发现粒子位于地面之上；但在下一个时间步 $t_0+\Delta t$ 时刻，我们发现粒子位于地面之下，这意味着发生了穿透现象。   
如果我们希望在碰撞时刻 $t_c$ 停止并重启模拟器，这就需要计算出 $t_c$。我们目前所知道的是 $t_c$ 位于 $t_0$ 和 $t_0+\Delta t$ 之间。通常，精确求解 $t_c$ 是困难的，因此我们通过数值方法在特定容差范围内求解 $t_c$。确定 $t_c$ 的一个简单方法是使用一种称为二分法的数值方法[14]。  
如果在 $t_0+\Delta t$ 时刻检测到相互穿透，我们通知常微分方程（ODE）求解器，希望在 $t_0$ 时刻重启，并模拟到 $t_0+\Delta t/2$ 时刻。如果模拟器到达 $t_0+\Delta t/2$ 时刻时没有遇到相互穿透，我们就知道碰撞时间 $t_c$ 位于 $t_0+\Delta t/2$  和 $t_0+\Delta t$ 之间。否则，$t_c$ 小于 $t_0+\Delta t/2$，我们尝试从 $t_0$ 模拟到 $t_0+\Delta t/4$。最终，碰撞时间  $t_c$ 会在一定的数值容差范围内被计算出来。 $t_c$ 的精度取决于碰撞检测程序。碰撞检测程序有参数 $\epsilon$。当我们计算得到的  $t_c$ “足够好”时——即粒子对地板的穿透深度不超过 $\epsilon$，且粒子在地板以上的高度小于 $\epsilon$——我们就认为粒子与地板接触（图14)。  
![](/imgs/Rigid Body Simulation Ⅱ/figure14.png)  
二分法虽然有点慢，但易于实现且相当稳健。更快捷的方法是通过考察位置 $\textbf{Y}(t_0)$ 和 $\textbf{Y}(t_0 + \Delta t)$ 来实际预测碰撞时间 $t_c$。Baraff[1, 2] 阐述了如何进行此类预测。具体如何实现，取决于你与常微分方程(ODE)求解器的交互方式。你可能需要使用异常处理代码向 ODE 求解器通报各种事件 (碰撞、相互穿透)，或者向 ODE 求解器传递某种消息。我们这里假定你有办法让 ODE 求解器恰好推进到时间 $t_c$ 这个点。 

一旦真正到达碰撞时间点，或在未发生穿透状态的任意时刻 $Y(t)$ 下，都必须进行几何判定以找出所有接触点。（仅仅因为你在检测物体A和B之间的碰撞时间，并不意味着可以忽略其他物体C和D之间的静接触力。无论何时推进模拟，都需要计算物体间的接触点和这些点的接触力。）有关碰撞检测问题的文献非常丰富。例如，近年来涉及该主题的SIGGRAPH论文有Von Herzen、Barr和Zatz[^17]以及Moore和Wilhelms[^12]；在机器人领域，多篇重要论文包括Canny[^4]、Gilbert和Hong[^6]、Meyer[^11]以及Cundall[^5]。Preparata和Shamos[^13]描述了计算几何中解决该问题的多种方法。在下一节中，我们将简要介绍一种针对本课程笔记所关注的模拟类型、能实现高效算法的碰撞检测“思路”。该算法的实际代码虽然易于编写，但过于冗长，无法纳入这些笔记。接下来，我们将继续探讨碰撞接触和静接触问题。 

### 7. Collision Detection

碰撞检测算法首先进行预处理步骤，即为每个刚体计算一个**包围盒**（边界框的边与坐标轴平行）。给定n个这样的包围盒后，算法需要快速确定所有发生重叠的包围盒对。对于包围盒未发生重叠的刚体对，无需进一步考虑。包围盒发生重叠的刚体对需要进一步分析。我们首先将描述如何高效检测**凸多面体**（convex polyhedra）定义的刚体之间是否存在相互穿透或接触点；随后将阐述如何高效执行包围盒检测。

如第1节所述，模拟过程是反复计算状态向量 $Y(t)$ 的导数 $\frac{\mathrm{d}}{\mathrm{d}t}\textbf{Y}(t)$（在多个时刻 *t* 处）。数值ODE求解器负责选择计算导数的时刻 $t$。对于复杂模拟，求解器选择的时刻需保证相邻时间步间状态 $\textbf{Y}$ 变化极小——这使得时间步之间存在极强的几何一致性。在 $t_0+\Delta t$ 时刻，可复用前一时刻 $t_0$ 的碰撞检测结果。

#### 7.1 Convex Polyhedra

我们利用**几何一致性**的核心机制是借助 **“见证信息（witnesses）”**。在我们的场景中，给定两个凸多面体 *A* 和 *B*，**见证信息**是一段可用于快速回答“*A* 和 *B* 是否不相交？”这一“是/否”问题的信息。我们通过**缓存前一时刻的见证信息**来利用时间步间的几何一致性：期望前一时刻的见证信息在当前时刻仍能作为有效依据。

由于我们考虑的是凸多面体，两个多面体不相交**当且仅当**它们之间存在一个**分离平面（separating plane）**。两个多面体之间的分离平面是指：每个多面体都位于该平面的**不同侧**。要验证一个平面是否为分离平面，只需确保多面体A和多面体B的所有顶点都位于平面的**对立两侧**。因此，分离平面是“两个凸多面体不相交”这一事实的**见证信息（witness）**。若不存在分离平面，那么多面体必然相互穿透。

最开始查找 witness (模拟的起始阶段，或者两个物体第一次足够接近以至于不仅需要包围盒检测) 这个消耗是无法避免的。要初始查找分离平面一个简单的方法如下。如果一对凸多面体是分离的或者接触(但不相交)，那么分离平面满足以下特性：要么该平面包含其中一个多面体的某个面，要么该平面包含其中一个多面体的一条边，并且与另一个多面体的一条边平行。（也就是说，分离平面的法线是两条边方向的叉积，而平面本身包含其中一条边。）我们称这些面和边为**特征面**、**特征边**。最初，我们简单地检查所有可能的面和边的组合，看看是否有一个这样的组合形成一个分离平面最初，我们简单地检查所有可能的面和边的组合，看看是否有一个这样的组合形成一个分离平面。(Figure 15)尽管这种方法效率不高，但因执行频率极低，效率问题并不关键。在后续时间步中，只需利用前一时间步找到的**特征面**或**特征边**构造分离平面，再验证该平面是否仍然有效（如图16所示）。  
![](/imgs/Rigid Body Simulation Ⅱ/figure15.png)  
![](/imgs/Rigid Body Simulation Ⅱ/figure16.png)  
在少数情况下，缓存的面或两条边无法构成分离平面，对与之前缓存的面和边相邻进行检测，判断其是否为分离平面。不过，这种情况发生的频率已经足够低，因此直接从头开始重新计算一个新的分离平面(无需依赖任何先验信息)反而可能更简便。  
![](/imgs/Rigid Body Simulation Ⅱ/figure17.png)  
一旦找到分离平面，如果两个几何体不是分离状态，就需要确定他们之间的接触面积。两个多面体的**接触点**仅可能出现在分离平面上。给定分离平面后，只需比较多面体上**与分离平面共面**的面、边和顶点，就能**快速且高效**地确定接触点。  
一旦无法找到分离平面，则两个多面体必然处于**相互穿透**状态。当多面体发生穿透时，几乎总是存在以下两种情况之一：要么一个多面体的**顶点**位于另一个多面体内部，要么一个多面体的**边**与另一个多面体的**面**相交。这种情况下穿透的顶点或者线、面将被缓存为 穿透的见证要素 (witness)。因为这意味着在更早的时刻发生了碰撞，模拟器将回溯并计算更早时刻的 $\frac{\mathrm{d}}{\mathrm{d}t}\textbf{Y}(t)$。在确定碰撞时刻之前，碰撞/接触判定步骤的首要操作是检查缓存的顶点或边与面，判断它们是否指示穿透。因此，因此，在找到碰撞时间之前，以最小的计算开销识别当前仍然存在互穿的状态。

#### 7.2 Bounding Boxes

为了减少成对碰撞/接触的次数，在仿真环境中引入 Bounding Boxs。如果两个 Bounding boxs 没重叠，不需要进一步计算包围盒内部的东西。给 n 个与坐标轴对齐的矩形包围盒，我们想要有效的计算出所有重叠的包围盒对。直接遍历的复杂度是 $O(n^2)$，效率太低了。现有的计算几何算法可以在$O(n\log n+ k)$时间内解决这个问题，其中 k 是成对重叠的数量；一般结果是，对于 d 维边界盒[^13]，问题可以在时间 $O(n\log ^{d−2}n+k)$内解决。使用相干性，我们可以获得更好的性能。

##### 7.2.1 The one-dimensional case

可以先考虑和坐标轴平行的一维包围盒。这样的包围盒可以定义为一个 $[b,e]$ 区间，$b,e$ 是实数。假设有 n 个区间，第 i 个区间为 $[b_i,e_i]$。问题就变成确定区间 $[b_i,e_j]$ 和 $[b_j,e_j]$ 相交的组合。

该问题最初可通过排序-扫描算法(sort and sweep algorithm) 解决。首先，创建一个包含所有区间起点 $b_i$ 和终点 $e_i$ 的有序列表(按值从低到高排序)。接着对列表执行扫描(sweep)，并维护一个初始为空的活动区间列表(active intervals)。当遇到某个区间的起点 $b_i$ 时，将该区间加入活动列表，并将活动列表中所有已有区间输出为“与区间 i 重叠”（如图18a所示）；当遇到某个区间的终点 $e_i$ 时，将该区间从活动列表中移除（如图18b所示）。该过程的时间复杂度为：创建有序列表需 $O(n\log n)$，扫描列表需 $O(n)$，输出每个重叠关系需 $O(k)$ (k 为重叠关系总数)。因此，总时间复杂度为 $O(n\log n+k)$，这是解决该问题的最优算法。  
![](/imgs/Rigid Body Simulation Ⅱ/figure18.png)  
后续比较可以用以下方法优化。首先不必使用 $O(n\log n)$ 算法构建 $b_i,e_i$ 的有序序列。利用前一个时间步进的 $b_i,e_i$ 的已有序列更高效。若时间相干性足够高，当前时间步进的有效序列几乎和前一个时间步进一致。因此可使用插入排序[^15]将“接近有序”的列表排列成有序列表。插入排序的原理是：将元素向列表头部移动，直到遇到比它更小的元素。具体来说，先比较第二个元素与第一个元素(必要时交换)，再将第三个元素向头部移动直到找到合适位置，以此类推；每次元素移动都对应两个值的顺序变化。当列表最后一个元素处理完成后，列表即有序。这种排序的时间复杂度为 $O(n+c)$ (c 为所需的交换操作次数)。例如，图19与图18的唯一区别是“区间4”向右移动了——从图18的有序列表出发，只需**一次交换** 就能得到图19的有序列表。插入排序一般不建议作为通用排序方法(因其最坏情况需要 $O(n^2)$ 次交换)；但在**高时间相干性的场景**（如本算法环境）中，处理“近乎有序”的列表时，它是高效的。要完善算法，需注意：若两个区间 $i$ 和 $j$ 在**前一时间步进重叠**，但在**当前时间步不重叠**，则必然发生了涉及 $b_i$（或 $e_i$）与 $b_j$（或 $e_j$)的一次或多次交换；反之，若区间 $i$ 和 $j$ 从前一时间步“不重叠”变为当前时间步“重叠”，也存在同样的交换逻辑。  
![](/imgs/Rigid Body Simulation Ⅱ/figure19.png)  
因此，如果我们在每个时间步进内维护一张重叠区间表，这张表可以在每个时间步进内以 $O(n+c)$ 的消耗被更新。假设有相干性(前后时刻区序列的差距很小)，那么所必须的交换次数 $c$ 将接近实际有效的交换次数 $k$ (这里的有效是在说，插入删除我们元素前移的过程中可能需要多次交换次数才能让元素到达正确的位置)，而额外的 $O(c-k)$ 可以忽略不计。因此，对于一维边界框问题，连贯性视角产生了一种极其(乃至极致[^18])简单的算法，其随着连贯性的提升而逼近最优解。 

##### 7.2.2 The three-dimensional case

求解三维空间包围盒相交问题的高效几何算法比一维空间的复杂得多。然而这些算法都有共同点，本质上都是在坐标轴上进行排序，就像之前的一维空间一样。每个包围盒被描述成三个独立区间 $[b_i^{(x)},e_i^{(x)}],[b_i^{(y)},e_i^{(y)}],[b_i^{(z)},e_i^{(z)}]$，表示第 i 个包围盒在三维坐标中的区间。因此我们首先能想到的，针对具有空间一致性下的几何算法，提高其效率的优化思路就是对包含 $[b_i^{(x)},e_i^{(x)}]$ 的列表进行排序，对于 y,z 轴也是同理的。同样的这个步骤会提升 $O(n+c)$ 的效率，这里 c 已经是三个列表交换次数的总和了。不过，如果我们注意到检测两个包围盒相交是常数时间复杂度的话，如果我们只是简单的判断相交即无论何时在索引 $i,j$ 对应的数据间发生交换操作(在任意坐标轴上)，我们就能在 $O(n+c)$ 是时间尺度上检测所有的相交变化。  
同样的，我们维护一个重叠包围盒表，并且在每个时间步进内以 $O(n+c)$ 的代价更新。额外交换消耗还是 $O(c-k)$。在三维空间情况下，如果两个包围盒在某一个坐标轴上发生改变，但是实际上他们没有相交，可能会增加额外的计算。在实际中，可以发现额外消耗可以不急，并且算法基本在 $O(n+k)$ 时间内完成。

### 8. Colliding Contact

在笔记剩余部分我们将研究某一特定时刻 $t_0$ 时的物体状态。在这个时间 $t_0$ 我们假设：没有物体相交，并且模拟器已经确定了那些物体接触、在哪个点接触。为了简化情况，我们假设所有物体都是多面体并且物体间所有的接触点都被检测到了。我们将多面体间顶点/面 和 边/边 接触视为接触。一个顶点接触另一个多面体的面时就视为发生 点/面接触。当一对边接触时视为发生 边/边 接触，这种情况下认为两条边不共线。(点/点 和 点/边 情况将被忽略，本篇不考虑这两种情况。)例如放在平面上的立方体可以视为有 4 个 点/面 接触，桌面上每个角各有一个。一个立方体放在桌子上但是底部悬在桌子边缘，仍然存在 4 个触点；两个在桌面的顶点是 点/面触点，两个 一条立方体的边 和 桌面边 形成的 边/边 触点。  
每个触点可以结构化为：  

~~~c++
struct Contact
{
    RigidBody	*a, // body containing vertev
    			*b; // body containing face
    triple		p,  // world-space vertex location
    			n,	// outwards pointing normal of face
    			ea,	// edge direction for A
    			eb; // edge direction for B
    boolean		vf; // TRUE if vertex/face contact
};

int Ncontacts;
Contact *Contacts;
~~~

如果发生 点/面 接触，变量 $a$ 指向接触顶点所属的刚体，$b$ 指向面所属的刚体。我们称这两个物体为 $A,B$，对于点/面 接触，变量 $n$ 被设为刚体接触面指向外侧的单位法线，变量 $ea,eb$ 没有被用到。  
对于 边/边 接触，$ea$ 是一个三维单位向量，指向物体 A(由a指定) 接触边的方向，$eb$ 也是同理。对于边/边接触，$n$ 表示 $ea\times eb$ 方向的单位向量。我们和以前一样，将两个接触体标记为 $A,B$，使得法线方向 $ea\times eb$ 从 B 指向外部，指向 A，就像点/面 接触一样。  
这两种接触类型，接触点的世界空间位置由 $p$ 给出(要么是接触点，要么是两条线的交点)。碰撞检测程序负责查找所有接触点，将 `Ncontacts` 设为接触点数量，并为 `contact` 数组分配空间初始化。  
![](/imgs/Rigid Body Simulation Ⅱ/figure20.png)  
首先我们要做的是检查所有 `Contact` 结构体，检测是否发生了碰撞接触。对于给定的接触点，两个物体在点 `p` 处接触。设 $p_a(t)$ 表示物体 A 的特定一点，满足 $p_a(t_0)=p$。(对于点/面接触而言,这个点就是顶点本身；对于边/边接触而言，是物体A接触边上的一些特定点。)相似的，设 $p_b(t)$ 表示物体 B 上特定一点，满足 $p_b(t_0)=p$ (figure 20)。尽管 $p_a(t_0),p_b(t_0)$ 是相同的，但是两点在 $t_0$ 的速度可不一定是相同的。我们将检测速度观察物体是否会发生碰撞。  
根据 2.5 节的内容我们可以知道 $\dot{p_a}(t_0)$  
$$
\dot{p_a}(t_0)=\omega(t)\times(p_a(t_0)-x_a(t_0))+v_a(t_0)\tag{8-1}
$$

$$
\dot{p_b}(t_0)=\omega(t)\times(p_b(t_0)-x_b(t_0))+v_b(t_0)\tag{8-2}
$$

让我们考察一个标量物理量。  
$$
v_{rel}=\hat n(t_0)\cdot(\dot{p_a}(t_0)-\dot{p_b}(t_0))\tag{8-3}
$$
![](/imgs/Rigid Body Simulation Ⅱ/figure21.png)  
在该式子中，$\hat n(t_0)$ 是每个接触点的表面单位法线，通过变量 `n` 描述。物理量 $v_{rel}$ 给出了 $\dot{p_a}(t_0)-\dot{p_b}(t_0)$ 在 $\hat n(t_0)$ 方向上的相对速度。很明显，若 $v_{rel}$ 是正数，则表示相对速度 $\dot{p_a}(t_0)-\dot{p_b}(t_0)$ 就是 $\hat n(t_0)$ 的正方向，说明两个物体正在相互远离，接触点将在 $t_0$ 时刻后立刻消失，正如 figure 21。我们无需关心这种情况。  
若 $v_{rel}$ 是 0，说明物体既没接近又没远离 $p$ 点(figure 22)。这就是所谓静止接触，在下一节我们会处理这种情况。  
![](/imgs/Rigid Body Simulation Ⅱ/figure22.png)  
还有最后一种情况，$v_{rel}<0$，这意味着相对速度和 $\hat n(t_0)$ 方向相反，会发生接触碰撞。如果物体速度没有立即改变，就会发生互相穿透(figure 23)。  
![](/imgs/Rigid Body Simulation Ⅱ/figure23.png)  
我们如何改变速度？任何力影响点 $p$，都需要至少一小段时间来完全停止两者之间的相对运动，无论这个力有多强。由于我们想物体瞬间改变速度，我们假设一个新的物理量，冲量 $J$ 。冲量是一个矢量，类似力，但是有动量单位。施加冲量可以使物体速度瞬间改变，我们可以想象一个很大的力作用在一个很短的时间 $\Delta t$ 内。如果我们令 $F$ 接近无穷，$\Delta t$ 接近 0，且满足  
$$
F\Delta t=J\tag{8-4}
$$
通过分析力 $F$ 在时间 $\Delta t$ 内对速度的影响，我们可以推导出冲量 $J$ 对物体速度的作用规律。  
例如我们对质量 $M$ 的物体施加冲量 $J$，线速度改变为：  
$$
\Delta v=\frac{J}{M}\tag{8-5}
$$
同样的，线性动量的变化 $\Delta P$ 被简化为 $\Delta P=J$。如果冲量作用于点 $p$，那么就类似力能够产生扭矩一样，$J$ 也可以有冲量扭矩。  
$$
\tau_{impulse}=(p-x(t))\times J\tag{8-6}
$$
可以想象，冲量扭矩 $\tau_{impulse}$ 也会引起角动量的变化 $\Delta L=\tau_{impulse}$。角速度的改变可以简化为 $I^{-1}(t_0)\tau_{impulse}$，假设冲量作用在时间 $t_0$。  
当两个物体碰撞时，我们将在他们之间应用一个冲量来改变他们的速度。对于无摩擦力影响的物体，冲量方向就是法线方向，$\hat n(t_0)$。因此我们简写  
$$
J=j\hat n(t_0)\tag{8-7}
$$
这里的 $j$ 是一个尚未确定的标量，其给出了冲量的大小。我们将采用以下约定，冲量$J$ 以正值作用于物体A，即物体A收到冲量 $+j\hat n(t_0)$，所以物体B受大小相等方向相反的冲量 $-j\hat n(t_0)$(figure 24)。  
![](/imgs/Rigid Body Simulation Ⅱ/figure24.png)  
我们通过碰撞试验的经验法则来计算 $j$。设 $\dot{p}_a^-(t_0)$ 为施加冲量前接触顶点的速度， $\dot{p}_a^+(t_0)$ 为我们施加冲量 $J$ 后的速度。 $\dot{p}_b^-(t_0),\dot{p}_b^+(t_0)$ 也同样这样定义。使用这些符号，得到在法线上的相对速度是  
$$
v_{rel}^-=\hat{n}(t_0)\cdot(\dot{p}_a^-(t_0)-\dot{p}_b^-(t_0))\tag{8-8}
$$
在冲量应用后：  
$$
v_{rel}^+=\hat{n}(t_0)\cdot(\dot{p}_a^+(t_0)-\dot{p}_b^+(t_0))\tag{8-9}
$$
无摩擦的经验定律描述为：  
$$
v_{rel}^+=-\epsilon v_{rel}^-\tag{8-10}
$$
$\epsilon$ 成为恢复系数，必须满足 $0\leq\epsilon\leq1$。  
如果 $\epsilon=1$，则 $v_{rel}^+=v_{rel}^-$，这个碰撞是完全弹性碰撞，且没有动能损失维持动能守恒。  
在另一个端点，$\epsilon=0$，则 $v_{rel}^+=0$ 动能损失达到最大，在碰撞之后两物体在接触点 p 进入静接触模式。(figure 25)  
![](/imgs/Rigid Body Simulation Ⅱ/figure25.png)  
计算冲量 $J=j\hat{n}(t_0)$ 的大小相对简单，尽管推导过程有点麻烦。我们定义位移 $p-x_a(t_0),p-x_b(t_0)$ 为 $r_a,r_b$，如果我们令 $v_a^-(t_0),\omega_a^-(t_0)$ 表示物体 A 碰撞前的速度，$v_a^+(t_0),\omega_a^+(t_0)$ 表示物体 A 碰撞后的速度，则可以列出  
$$
\dot{p}_a^+(t_0)=v_a^+(t_0)+\omega_a^+(t_0)\times r_a\tag{8-11}
$$

$$
\begin{aligned}
v_a^+(t_0)&=v_a^-(t_0)+\frac{j\hat{n}(t_0)}{M_a}\\
\omega_a^+(t_0)&=\omega_a^-(t_0)+I_a^{-1}(t_0)(r_a\times j\hat{n}(t_0))\\
\end{aligned}
\tag{8-12}
$$

$$
\begin{aligned}
\dot{p}_a^+(t_0)&=(v_a^-(t_0)+\frac{j\hat{n}(t_0)}{M_a})+(\omega_a^-(t_0)+I_a^{-1}(t_0)(r_a\times j\hat{n}(t_0)))\times r_a\\
&=v_a^-(t_0)+\omega_a^-(t_0)\times r_a+
(\frac{j\hat{n}(t_0)}{M_a})+
(I_a^{-1}(t_0)(r_a\times j\hat{n}(t_0)))\times r_a\\
&=\dot{p}_a^-+
j(\frac{\hat{n}(t_0)}{M_a}+I_a^{-1}(t_0)(r_a\times\hat{n}(t_0)))\times r_a
\end{aligned}
\tag{}
$$

对于 $\dot{p}_a^+(t_0)$ 我们可以法线，这是一个给关于 j 的线性函数，  
对于物体 B，而言，收到一个相反的冲量$-j\hat{n}(t_0)$  
$$
\dot{p}_b^+(t_0)=\dot{p}_b^--j
(\frac{\hat{n}(t_0)}{M_a}+I_a^{-1}(t_0)(r_a\times\hat{n}(t_0)))\times r_b\tag{8-14}
$$

$$
\begin{aligned}
\dot{p}_a^+(t_0)-\dot{p}_b^+(t_0)&=
(\dot{p}_a^+-\dot{p}_b^+)+
j(\frac{\hat{n}(t_0)}{M_a}+
\frac{\hat{n}(t_0)}{M_b}+\\
&(I_a^{-1}(t_0)(r_a\times\hat{n}(t_0)))\times r_a+
(I_b^{-1}(t_0)(r_b\times\hat{n}(t_0)))\times r_b)
\end{aligned}\tag{8-15}
$$

为了计算 $v_{rel}^+$ 我们使用带 $\hat{n}(t_0)$ 的表达式，由于 $\hat{n}(t_0)$ 是单位矢量， $\hat{n}(t_0)\cdot\hat{n}(t_0)=1$，则：  
$$
\begin{aligned}
v_{rel}^+&=\hat{n}(t_0)\cdot
(\dot{p}_a^+(t_0)-\dot{p}_b^+(t_0))\\
&=\hat{n}(t_0)\cdot
(\dot{p}_a^-(t_0)-\dot{p}_b^-(t_0))+
j\bigg(\frac{1}{M_a}+\frac{1}{M_b}+\\
&\hat{n}(t_0)\cdot
\big(I_a^{-1}(t_0)(r_a\times\hat{n}(t_0))\big)\times r_a+
\hat{n}(t_0)\cdot
\big(I_b^{-1}(t_0)(r_b\times\hat{n}(t_0))\big)\times r_b\bigg)\\
&=v_{rel}^-+j\bigg(\frac{1}{M_a}+\frac{1}{M_b}+\\
&\hat{n}(t_0)\cdot
\big(I_a^{-1}(t_0)(r_a\times\hat{n}(t_0))\big)\times r_a+
\hat{n}(t_0)\cdot
\big(I_b^{-1}(t_0)(r_b\times\hat{n}(t_0))\big)\times r_b\bigg)
\end{aligned}\tag{8-16}
$$
 我们能够使用 $v_{rel}^-$ 表达 $v_{rel}^+$ ，根据 8-10 和 8-16：  
$$
\begin{aligned}
v_{rel}^-+j\bigg(\frac{1}{M_a}+\frac{1}{M_b}+
\hat{n}(t_0)\cdot
\Big(I_a^{-1}(t_0)(r_a\times\hat{n}(t_0))\Big)\times r_a+\\
\hat{n}(t_0)\cdot
\Big(I_b^{-1}(t_0)(r_b\times\hat{n}(t_0))\Big)\times r_b\bigg)=-\epsilon v_{rel}^+
\end{aligned}
\tag{8-17}
$$

$$
\begin{aligned}
j&=\\
&\frac{-(1+\epsilon)v_{rel}^-}
{\frac{1}{M_a}+\frac{1}{M_b}+
\hat{n}(t_0)\cdot
\Big(I_a^{-1}(t_0)
\big(r_a\times\hat{n}(t_0)\big)\Big)\times r_a+
\hat{n}(t_0)\cdot
\Big(I_b^{-1}(t_0)
\big(r_b\times\hat{n}(t_0)\big)\Big)\times r_b}
\end{aligned}
\tag{8-18}
$$

> 这里原文放了一大堆代码，我就不写了……懒了

需注意以下几点：
恢复系数 $\epsilon=0.5$ 是任意选取的。在实际物理引擎实现中，需允许用户根据碰撞物体的材质组合（如“钢球-橡胶垫”与“木块-金属板”的碰撞），灵活设置不同的 $\epsilon$ 值;  
每次检测到碰撞时，必须重新扫描接触点列表：  
因为原本静止的物体可能因碰撞开始运动，且新的碰撞事件也可能同步发生;  
多接触点碰撞的顺序对模拟结果有影响：  
例如“立方体平落至平面”时，四个顶点会同时发生碰撞——此时若按不同顺序处理接触点，模拟结果可能截然不同。虽然存在“同时计算多个接触点的冲量”的进阶方法，但实现更复杂，且其原理基于下一节“静接触”的核心概念。若需深入，可参考 Baraff 的文献[^1]。  
顺便说一下，如果您想要某些物体是“固定的”，不能移动（比如地板或墙壁），您可以使用以下技巧：对于此类物体，将质量设为 0；同时让惯性张量的逆矩阵也为 3×3 的零矩阵。您可以专门编写代码来检查某个物体是否应固定不动，或者重新编写刚体的定义，用 invmass 替代 mass。对于普通物体，invmass 是质量的倒数，而对于固定物体，invmass 为 0。惯性张量也是如此。（请注意，在任何动力学计算（包括下一节）中都不会使用质量或惯性张量，只使用它们的逆矩阵，所以您无需担心除以零的问题。）在下一节关于静止接触的内容中，也可以使用同样的技巧来模拟能够承受任意重量而不移动的物体。

### 9. Resting Contact

静止接触的情况，当物体在接触点既不碰撞也不分离时，是我们将在这些笔记中解决的最后（也是最难的）动力学问题。要实现本节中的内容，必须设计一个相当复杂的数值软件，我们将在下面描述它。  
对于这种情况，我们假设有 n 个接触点的情景。在每个接触点，物体都处于静接触状态，也就是说，按照第 8 节的说法，相对速度 $v_{rel}$ 是零(指在数值容差THRESHOLD 内可以视为零)。我们可以这样认为，因为 `find_all_collisions` 程序能够消除碰撞接触，并且任何相对速度 $v_{rel}$ 大于 THRESHOLD 的接触点都可以安全地忽略，因为此时物体正在分离状态。  
与碰撞接触类似，对于每个接触点，存在一个作用在接触面沿着接触面法线的接触力。对于碰撞接触情况，有冲量 $j\hat{n}(t_0)$，$j$ 是未知的标量。对于静接触情况，每个接触点某个力 $f_i\hat{n}_i(t_0)$，$f_i$ 是一个未知的标量，$\hat{n}_i(t_0)$ 是第 i 个接触点的法线(figure 26)。我们的目标是计算每个 $f_i$ 大小。在计算 $f_i$ 们的时候，他们必须被同时被确定，因为在第 i 个接触点的力可能会影响物体上第 j 个接触点，可能会影响发生接触的两个物体的其中一个，也可能两个物体都会受到影响。在第 8 节我们写过接触点 $p_a(t_0),p_b(t_0)$ 速度如何随 $j$ 变化。在这节我们会做类似的事，不过现在需要描述点 $p_a(t_0),p_b(t_0)$ *加速度*如何根据每个 $f_i$ 变化。  
![](/imgs/Rigid Body Simulation Ⅱ/figure26.png)

对于碰撞接触，有经验规律，将冲量强度 $j$ 与相对速度和恢复系数联系起来。对于静接触，根据三个条件来计算 $f_i$ 们。  
**首先，接触力必须避免相互穿透**，就是说，接触力必须足够大防止接触的两个物体发生“挤压”。  
**其次，我们希望接触力是互相排斥的**，接触力能把物体相互推开，但是不能像“胶水”一样将两个物体粘在一起。  
**最后，要求当物体开始分离的时候，接触点的力必须为零**。例如，当一个方块静置在桌子上，在接触点上可能有一些力防止方块在重力作用下向下加速。然而如果有一个力使得方块向上，那么方块和桌子的接触力在方块离开桌子的那一刻就会为零。

让我们解决第一个条件：**预防相互穿透**。对于每个接触点 $i$，我们构建表达式 $d_i(t)$，该表达式描述了在时刻 $t$ 接触点上两物体的分离距离。距离为正值则表示物体已脱离接触，并且在第 $i$ 点发生分离。而距离值为负，则表示相互穿透。因为在时刻 $t_0$ 物体发生接触，对于每个接触点都有 $d_i(t_0)=0$(在数值容差范围内)。我们的目标是确保**接触力**使得每个接触点的在未来时刻 $t>t_0$，始终满足 $d_i(t)\ge 0$。

对于点/面接触，能够离开构建简单的函数 $d_i(t)$。若点 $p_a(t),p_b(t)$ 是物体 A,B 的第 i 个接触点，则在未来时刻 $t\ge t_0$ 顶点和面的距离为  
$$
d_i(t)=\hat{n}_i(t)\cdot(p_a(t)-p_b(t))\tag{9-1}
$$
在时刻 $t$，函数 $d(t)$ 表示了物体 A,B 之间的分离距离。若 $d_i(t)$ 为零，则在第 i 个接触点物体将会失去接触。然而如果 $d_i(t)<0$，物体就相互穿透了，这是需要避免的(figure 27)。同样的函数也可以用在边/边接触，因为 $\hat{n}_i(t)$ 是指向外侧，从B指向A(通常都这么定义的)，如果两个接触边移动使物体分离，$\hat{n}_i(t)\cdot(p_a(t)-p_b(t))$ 将为正值。  
![](/imgs/Rigid Body Simulation Ⅱ/figure27.png)  
由于 $d_i(t_0)=0$，我们必须保证给在时刻 $t_0$，$d_i(t_0)$ 不会有减小趋势，即必须有 $\dot{d_i}(t_0)\ge 0$。  
$$
\dot{d_i}(t_0)=
\dot{\hat{n}}_i(t)\cdot(p_a(t)-p_b(t))+
\hat{n}_i(t)\cdot(\dot{p}_a(t)-\dot{p}_b(t))
\tag{9-2}
$$
因为 $d_i(t)$ 描述分离距离，$\dot{d_i}(t)$ 描述的是在时刻 $t$ 的分离速度。然而在时刻 $t_0,p_a(t_0)=p_b(t_0)$ 意味着 $\dot{d_i}(t_0)=\hat{n}(t_0)\cdot(\dot{p}_a(t_0)-\dot{p}_b(t_0))$。式子和前面提到的 $v_{rel}$ 比较相似。函数 $\dot{d_i}(t_0)$ 是判断物体如何分离的度量，对于静接触物体，$\dot{d_i}(t_0)=0$，因为物体在接触点上既不会再接近彼此，也不会远离对方。  
在此基础上，可以推断出 $d_i(t_0)=\dot{d_i}(t_0)=0$。来观察 $\ddot{d_i}(t_0)$,  
$$
\tag{9-3}
\begin{aligned}
\ddot{d_i}(t)&=
\Big(\ddot{\hat{n}}_i(t)\cdot
\big(p_a(t)-p_b(t)\big)+
\dot{\hat{n}}_i(t)\cdot\big(\dot{p}_a(t)-\dot{p}_b(t)\big)\Big)+\\
&\Big(\dot{\hat{n}}_i(t)\cdot
\big(\dot{p}_a(t)-\dot{p}_b(t)\big)+
\hat{n}_i(t)\cdot\big(\ddot{p}_a(t)-\ddot{p}_b(t)\big)\Big)\\
&=\ddot{\hat{n}}_i(t)\cdot(p_a(t)-p_b(t))+\\
&2\dot{\hat{n}}_i(t)\cdot
\big(\dot{p}_a(t)-\dot{p}_b(t)\big)+
\hat{n}_i(t)\cdot\big(\ddot{p}_a(t)-\ddot{p}_b(t)\big)
\end{aligned}
$$
有 $p_a(t_0)=p_b(t_0),\ddot{d_i}(t_0):$  
$$
\ddot{d_i}(t_0)=2\dot{\hat{n}}_i(t_0)\cdot
\big(\dot{p}_a(t_0)-\dot{p}_b(t_0)\big)+
\hat{n}_i(t)\cdot\big(\ddot{p}_a(t_0)-\ddot{p}_b(t_0)\big).
\tag{9-4}
$$
物理量 $\ddot{d_i}(t_0)$ 意味着在接触点 $p$ 两个物体相对的加速度如何。  
若 $\ddot{d_i}(t_0)>0$，物体彼此之间有远离的加速度，在时刻 $t_0$ 后，接触立刻分离。  
如果 $\ddot{d_i}(t_0)=0$，则保持接触状态。  
另一种情况 $\ddot{d_i}(t_0)<0$ 必须避免，因为这表示物体相对加速度是向着彼此的。  
注意，如果物体 B 是固定的，则 $\hat{n}_i(t_0)$ 是常数，得 $\dot{\hat{n}}_i(t_0)$ 是零，式子可以进一步简化。

因此，针对每个接触点我们有如下约束，来保证接触力的第一个条件——**预防相互穿透**。  
$$
\ddot{d_i}(t_0)\ge0\tag{9-5}
$$
由于加速度 $\ddot{d_i}(t_0)$ 取决于接触力，所以这实际上就是对接触力的约束。

下面来研究一下第二条和第三条约束。由于**接触力总是表现为相互排斥的**，所以每个接触力的方向的必须向外作用。这意味着每个 $f_i$ 都必须为正，因为作用在物体 A 上的力为 $f_i\hat{n}_i(t_0)$，而 $\hat{n}_i(t_0)$ 是指向外侧朝向 B 的法线，所以针对每个接触点都要满足：  
$$
f_i\ge0\tag{9-6}
$$
第三个约束可以简单的用 $f_i,\ddot{d_i}(t_0)$ 表示。因为如果第 i 个接触点正在分离，接触力 $f_i\hat{n}_i(t_0)$ 必须为零。可以表示为：  
$$
f_i\ddot{d_i}(t_0)=0;\tag{9-7}
$$
如果接触正在分离，$\ddot{d_i}(t_0)>0$ 并且要满足等式(9-7) 则必有 $f_i=0$。如果接触不分离，则 $\ddot{d_i}(t_0)=0$ 并且满足了等式(9-7)不必理会 $f_i$。

为了真的找到满足等式(9-5),(9-6),(9-7)，我们要用未知的 $f_i$ 们描述每一个 $\ddot{d}_i(t_0)$。  
$$
\ddot{d_i}(t_0)=a_{i1}f_1+a_{i2}f_2+\cdots+a_{in}f_n+b_i.\tag{9-8}
$$
矩阵形式为  
$$
\begin{aligned}
\begin{pmatrix}
\ddot{d_1}(t_0)\\
\vdots\\
\ddot{d_n}(t_0)
\end{pmatrix}
=
\textbf{A}

\begin{pmatrix}
f_1\\
\vdots\\
f_n
\end{pmatrix}
+

\begin{pmatrix}
b_i\\
\vdots\\
b_n
\end{pmatrix}
\end{aligned}
\tag{9-9}
$$
$\textbf{A}$ 是式(9-8) 中 $a_{ij}$ 的 nxn 系数矩阵。尽管计算 $a_{ij},b_i$ 的代码并不复杂，但是计算部分代码所依赖的推导过程有些复杂。推导过程我们放在附录D，以及计算 $a_{ij},b_i$ 的代码。  

~~~c++
void compute_a_matrix(Contact contacts[ ], int ncontacts,
bigmatrix &a);
void compute_b_vector(Contact contacts[ ], int ncontacts,
vector &b);
~~~

这里 `matrix,vector` 类型表示任意大小的矩阵和向量。第一个计算 $a_{ij}$， 第二个计算 $b_{i}$。

一旦我们计算完这些，就可以解方程 (9-5),(9-6),(9-7)。这个方程形式就是 二次规划——*quadratic program*(QP)。也就是说满足这个方程的 $f_i$ 是通过叫做 quadratic programming 的算法得到的。不是所有的二次规划问题又能高效的得到对应解，但是我们的接触力都是垂直于接触面的(意味着不涉及摩擦)，事实证明，关于我们这种 QP 问题是可以高效解决的。值得一提的是，QP 代码很容易处理 $\ddot{d_i}(t_0)=0$ 而不是 $\ddot{d_i}(t_0)\ge0$。如果我们希望约束两个物体在接触点永远不分离，我们需要 $\ddot{d_i}(t_0)=0$ (并且丢掉约束 $f_i\ge0$)。这使得我们在仿真过程中能实现铰链(hinges)、销接头(pin-joints)以及非穿透约束。

然而二次规划的代码并不常见，显然他们不像线性方程那么常见，而且实现也更困难。原文的二次规划程序来自the Department of Operations Research at Stanford University，详情参考 Gill et al.[^7][^8][^9]。文中经常使用 Bareff[^3]提供的二次规划代码。如果真的咬定青山不放松，建议直接看这篇论文(除了关于接触和摩擦的部分)。  
无论如何，我们假设有一个可用的 QP 求解器。我们假设将矩阵 $\textbf{A}$ 和向量 $b_i$ 们传递给 QP 求解器，然后就能得到 $f_i$ 们的向量。我们假设  

~~~c++
void qp_solve(bigmatrix &a, vector &b, vector &f);
~~~

来看看如何计算所有的静接触力。下面代码大概在 `find_collisions` 之后，由 `Compute_Force_and_Torque` 调用。  
~~~c++
void compute_contact_forces(Contact contacts[], int ncontacts, double t)
{
    /*
    假设 contacts[] 内每个元素都代表静接触中的一个接触(contact)。
    此外，对于 Bodies[] 的每个元素，'force','torque'字段，
    都被设置为作用于在该物体上的力(force)和扭矩(torque),因为重力、风等，可能通过调用Compute_External_Force_and_Torque_for_all_Bodies(t);
    */
    /*
    申请 nxn 矩阵`amat`, n维向量`fvec`、`bvec`
    */
    bigmatrix amat = new bigmatrix(ncontacts, ncontacts);
    vector bvec = new vector(ncontacts),
		   fvec = new vector(ncontacts);
    /*
    计算 a_ij,b_i 系数
    */
    compute_a_matrix(contacts, ncontacts, amat);
	compute_b_vector(contacts, ncontacts, bvec);
    
    // 解 f_i
    qp_solve(amat, bmat, fvec);
    
    /*
    现在将刚刚计算的静接触力添加到每个刚体的 'force','torque'字段内
    */
    for(int i = 0; i < ncontacts; i++)
    {
        double f = fvec[i]; /* fi */
        triple n = contacts[i]->n; /* nˆi(t0) */
        RigidBody *A = contacts[i]->a, /* body A */
        *B = contacts[i]->b; /* body B */
        /* apply the force `f n' positively to A... */
        A->force += f * n;
        A->torque += (contacts[i].p - A->x) * (f*n);
        /* and negatively to B */
        B->force -= f * n;
        B->torque -= (contacts[i].p - B->x) * (f*n);
    }
}
~~~

差不多就是这样了！现在静息力已经计算出来，并与外力结合在一起，我们将控制权交还给ODE求解器，每个物体都以物理正确的方式愉快地沿着自己的道路前进，没有相互渗透。

### References 

[^1]:D. Baraff. Analytical methods for dynamic simulation of non-penetrating rigid bodies. In Computer Graphics (Proc. SIGGRAPH), volume 23, pages 223–232. ACM, July 1989.]   
[^2]:D. Baraff. Curved surfaces and coherence for non-penetrating rigid body simulation. In Computer Graphics (Proc. SIGGRAPH), volume 24, pages 19–28. ACM, August 1990.  
[^3]:D. Baraff. Fast contact force computation for nonpenetrating rigid bodies. Computer Graphics (Proc. SIGGRAPH), 28:23–34, 1994.  
[^4]:J. Canny. Collision detection for moving polyhedra. IEEE Transactions on Pattern Analysis and Machine Intelligence, 8(2), 1986.  
[^5]:P.A. Cundall. Formulation of a three-dimensional distinct element model—Part I. A scheme to represent contacts in a system composed of many polyhedral blocks. International Journal of Rock Mechanics, Mineral Science and Geomechanics, 25, 1988.  
[^6]:E.G. Gilbert and S.M. Hong. A new algorithm for detecting the collision of moving objects. In International Conference on Robotics and Automation, pages 8–13. IEEE, 1989.  
[^7]:P. Gill, S. Hammarling, W. Murray, M. Saunders, and M. Wright. User’s guide for LSSOL: A Fortran package for constrained linear least-squares and convex quadratic programming. Technical Report Sol 86-1, Systems Optimization Laboratory, Department of Operations Research, Stanford University, 1986.  
[^8]:P. Gill, W. Murray, M. Saunders, and M. Wright. User’s guide for QPSOL: A Fortran package for quadratic programming. Technical Report Sol 84-6, Systems Optimization Laboratory, Department of Operations Research, Stanford University, 1984.  
[^9]:P. Gill, W. Murray, M. Saunders, and M. Wright. User’s guide for NPSOL: A Fortran package for nonlinear programming. Technical Report Sol 86-2, Systems Optimization Laboratory, Department of Operations Research, Stanford University, 1986.  
[^10]:H. Goldstein. Classical Mechanics. Addison-Wesley, Reading, 1983.  
[^11]:W. Meyer. Distance between boxes: Applications to collision detection and clipping. In International Conference on Robotics and Automation, pages 597–602. IEEE, 1986.  
[^12]:P.M. Moore and J. Wilhelms. Collision detection and reponse for computer animation. In Computer Graphics (Proc. SIGGRAPH), volume 22, pages 289–298. ACM, August 1988.  
[^13]:F.P. Preparata and M.I. Shamos. Computational Geometry. Springer-Verlag, New York, 1985.  
[^14]:W.H. Press, B.P. Flannery, S.A. Teukolsky, and W.T. Vetterling. Numerical Recipes. Cambridge University Press, 1986.  
[^15]:R. Sedgewick. Algorithms. Addison-Wesley, 1983.  
[^16]:K. Shoemake. Animating rotation with quaternion curves. In Computer Graphics (Proc. SIGGRAPH), volume 19, pages 245–254. ACM, July 1985.  
[^17]:B. Von Herzen, A. Barr, and H. Zatz. Geometric collisions for time-dependent parametric surfaces. In Computer Graphics (Proc. SIGGRAPH), volume 24, pages 39–48. ACM, August 1990  
[^18]:不会翻译 - if not maximal

[Physically Based Modeling](https://www.cs.cmu.edu/~baraff/sigcourse/index.html)  
[Rigid Body Simulation Ⅱ](https://www.cs.cmu.edu/~baraff/sigcourse/notesd2.pdf)

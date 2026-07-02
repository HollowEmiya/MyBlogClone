---
title: Road Networks
math: true
tags: [Math][DCC]
index_img: 
banner_img: 
data: 2026--06--04 09:09:48
typora-root-url: ./..
---

路网制作相关内容。

<!--more-->

# Procedural Generation of Roads

## 3. Discrete anisotropic shortest path algorithm

### Anisotropic shortest path problem

本文研究**连续域上的加权各向异性最短路径问题**：即计算连接两点的路径，使路径上代价权重函数的线积分最小。

设有界且闭的紧集$\Omega\in\mathbb{R^2}$(其实就是平面), 找到起点a, 终点b 之间的连续路径 $\rho$.  
使得代价函数 $c(p,\dot{p},\ddot{p})$ 的线积分最小. 

* $p$ : 路径位置
* $\dot{p}$ : 位置的一阶导数, 其实速度? 表示路径的切线方向, 描述路径的坡度.  
  找寻坡度更小的路径.
* $\ddot{p}$ : 位置二阶导数, 其实是加速度? 表示路径的曲率, 如果曲率过大, 会产生很大加速度a or 重力g. 过大的曲率是不合理的道路

设 $\mathcal{P}$ 是在 $\Omega$ 从 a 到 b 的二次连续可微分路径的集合.(二次连续可微分保证路径的速度和加速度不会跳变, 是正常的路径), 即 $\mathcal{P}$  表示从 $[0,T]\rightarrow\Omega$ 的连续函数 $\rho$ 的集合, 在 $\rho$ 里, 从时域空间到空域空间: $[0,T]\rightarrow\Omega$, 且 $\rho(0)=a, \rho(T)=b$.  
路径代价的函数 $C$ : $\mathcal{P}\rightarrow[0,+\infty)$:
$$
\begin{aligned}
C(\rho)=\int_0^Tc(p(t),\dot{p}(t),\ddot{p}(t))\mathrm{d}t
\end{aligned}
$$

目的就是找到最短路径 $\rho^*$, 使得 :
$$
\begin{aligned}
C(\rho^*)=\mathop{min}\limits_{\rho\in\mathcal{P}}C(\rho)
\end{aligned}
$$

### Discrete anisotropic shortest path

就是说把区域 $\Omega$ 离散为网格, 将路径变成对网格点的线段拼接.  
最短路径变成有限图 $\mathcal{G}$ 的最短路径问题.

### Grid sampling and graph definition

设 $\boldsymbol{p}_{ij},(i,j)\in[0,n-1]$ 为搜索区域上均匀采样得到的网格点, 这些点对应的是图 $\mathcal{G}$ 上的节点. 因为代价函数的各向异性的, 所以网格点和临近的网格点不一定能够逼近最优路径. 所以, 认为每个网格节点 $\boldsymbol{p}_{ij}$ 在距离 $r$ 内和相邻的网格节点隐性相连通. 定义为: 
$$
\begin{aligned}
\mathcal{M}(\boldsymbol{p}_{ij})\subset\{\boldsymbol{q},||\boldsymbol{p}_{ij}-\boldsymbol{q}||\leq r\}
\end{aligned}
$$

显示存储所有链接关系会浪费大量内存，所以使用路径段掩码 $\mathcal{M}_k$ 来存储网格之间的连通关系。

### Shortest path computation

使用 A* 算法计算最短路径.  
A* 是一种最佳优先图搜索算法，它可以找到从给定起始节点到目标节点的最小代价路径。我们使用代价函数 $c(p)$ 加上可接受的启发式代价估计函数 $e(p)$ 来确定搜索访问节点的顺序。我们将函数 $e(p)$ 定义为到目标 b 的直线路径代价，这样它永远不会高估到达 b 的实际最小代价。启发式函数 $e$ 是单调的，因此 A* 本身是可接受的。

代价函数 $c(p)$ 表示 A* 已走过路径的代价，启发式代价估计函数 $e(p)$ 表示预估的剩余路径代价。启发式函数 $e$ 是单调的，指的是如果有任意相邻节点 $p,p'$，一定有 $e(p)<e(p')+p到p'的实际成本$，不会出现跳变。

关于生成算法，对于每个网格节点，定义代价值函数表示为 $c(p_{ij})$ 以及前驱节点的函数表示为 $p(p_{ij})$。所有节点的代价值 $c(p_{ij})$ 首先初始化为 $\infty$，起始节点的 $c(a)$ 设置为 $h(b)$。用初始点 a 初始化优先队列 $\mathcal{Q}$。算法的主循环如下：

1. 当 $\mathcal{Q}$ 不空时，从优先队列中选择代价值 $c(p_{ij})$ 最小的节点 $p_{ij}$
2. 如果选择的 $p_{ij}=b$，停止算法
3. 对于所有的点 $q\in\mathcal{M}_k(p_{ij})$，计算线段 $[p_{ij},q]$ 的代价 $c(p_{ij},q)$；如果 $[p_{ij},q]$ 段的代价 $c(p_{ij},q)<c(q)$，说明节点 $q$ 从 $p_{ij}$ 路径相连代价更小，我们将 $p_{ij}$ 设为 $q$ 的前驱节点。

该算法最后生成的是一组网格节点序列组成的离散路径，定义为 $\rho^*={p_k},k\in[0,n]$。

算法第3步涉及计算网格节点集合 $\mathcal{M}_k(p_{ij})$ 来确定和哪些节点相连。还需要评估连接两个网格节点线段上的代价函数。该代价按照如下方式进行即时运算。设 $[p_k,p_{k+1}]$ 表示路径的一段，$t_k,t_{k+1}$ 分别是对应于 $p_k,p_{k+1}$ 的参数。线积分的定义为：
$$
\begin{aligned}
c(p_k,p_{k+1})=\int_{t_k}^{t_{k+1}}c(p(t),\dot{p}(t),\ddot{p}(t))\mathrm{d}t
\end{aligned}
$$
通过将积分域离散为 n 个区间，将曲线积分近似为有限和。

### Cost functions definition

上述定义了一组参数化的代价函数 $c(p(t),\dot{p}(t),\ddot{p}(t))$，用以评估代价函数的线积分。这些函数由用户定义，并通过<u>约束最短路径搜索</u>间接地控制路径的轨迹。  
下一节讨论代价函数的计算。

## 4. Cost functions

本节中介绍一类成本函数，该函数定义了地形坡度和自然障碍对最短路径计算的影响。我们将全局代价函数 $c$ 定义为多个加权函数的总和，这些加权函数评估地形和道路不同特征的相对影响。建议将成本函数定义为：
$$
\begin{aligned}
c(p,\dot{p},\ddot{p})=\sum_{i=0}^{i=n-1}\mu_i\circ\kappa_i(p,,\dot{p},\ddot{p})
\end{aligned}
$$
函数集 $\kappa_i:\mathbb{R}^3\times\mathbb{R}^3\times\mathbb{R}^3\rightarrow\mathbb{R}$ 评估地形的不同特性以及道路在点 $p$ 处轨迹的几何属性。函数 $\mu_i$ 是传递函数，用于加权并组合特征 $\kappa_i(p,\dot{p},\ddot{p})$ 的影响。传递函数允许用户控制场景参数的相对影响，从而控制最短路径的形状。

### 4.1 Surface roads

地形表面的道路轨迹应受到地形坡度和自然障碍物(如河流、湖泊和森林)的约束。代价函数还考虑了道路的曲率以避免生成急转弯。

#### Characteristic function

设 $p$ 表示地形表面道路轨迹上的一个点。计算以下特征函数：植被密度 $v(p)$，水位高度 $w(p)$、沿道路轨迹方向的地形坡度 $s(p,\dot{p})$ 以及道路轨迹的曲率 $c(p,\dot{p},\ddot{p})$。

水位 $w(p)$ 定义为点 $p$ 在地面上投影的周围最小区域 $\Omega(p,r)$ 内的最大水位高度。$\Omega(p,r)$ 表示以点 $p$ 为中心、$r$ 为半径的球体。植被密度 $v(p)$ 的计算方法是评估位于 $\Omega(p,r)$ 内的树木数量。

#### Transfer functions

本节中，介绍了 transfer functions，这些函数用于权衡场景和道路轨迹特征的影响。在我们的系统中，transfer functions 的特征由其图形表示，并可以进行交互式编辑(Figure 4)。  
<img src="/imgs/Road Networks/Figure4.png" alt="地面道路成本函数的综合表示。" title="地面道路成本函数的综合表示。" style="zoom:80%;" />

Transfer functions的特性由一个阈值决定，记为 $\kappa_0$。如果输入的特性 $\kappa$ 大于 $\kappa_0$，则对应的transfer function $\mu(\kappa)$ 应返回无穷。这使我们能够控制定义不能创建路径的区域。回顾两个 transfer function。

* Slope，将 p 点的坡度定义为 $s(p,\dot{p})$。如果坡度过大 transfer function 应返回无限大避免生成不切实际的路径。否则，使用坡度相关函数，$\mu(\kappa_0)$ 为允许的最大代价。
* Water，将区域 $\Omega(p,r)$ 的最大水深定义为 $w(p)$。如果水深过大，不能生成道路，water transfer function 返回无限大。否则如果水位足够低，可以生成道路，(但需要路堤)。这种情况使用水深函数，并且 $\mu(\kappa_0)$ 表示最大水深的 cost 成本。

### 4.2 Bridge and tunnels

桥梁和隧道是复杂结构，其生成受到地质条件、几何参数等多因素的影响。

在本文中桥梁的代价由高度 $h(p)$，坡度，跨越的水体深度决定。同时要考虑桥梁的曲率，避免产生危险急转的桥梁。函数 $h(p)$ 表示桥体在点 p 相对地面的垂直高度。相应的 transfer function $\mu_{height}$ 是一个分段函数，如果高度超过所考虑桥类型的最大高度，则返回无穷大。

本文中的隧道成本由隧道到地表深度 $d(p)$、坡度以及隧道上方可能存在的水体深度所参数化。关于水体深度的参数使得能够模拟闯过河流或海洋的特别隧道。

## 5. Segment path masks

本节介绍在均匀采样网格上计算各向异性最短路径的办法。

#### The limit on direction problem

均匀网格采样的一个重要问题是，如果希望 最短离散路径 近似 最短连续路径的位置和方向，网格点的数量会增长的非常快，因此效率不高。原因是因为传统网格的方向限制问题。  
<img src="/imgs/Road Networks/Figure7.png" style="zoom:80%;" />

离散路径是一条由连续采样点的直线段组成的直线路径。原本只考虑采样点之间的 4/8 连通。因此离散路径的线段只能有 4/8 个方向，角度的最大分辨率为 45 度(是不是因为角度的限制导致会不断sample point到路径来逼近最短连续路径)。使用Shift Path 可以允许路径偏离网格节点以部分克制这个问题，但是需要一个计算量大的步骤计算偏移路径。

#### Overview

为了克服方向限制问题，增加领域距离，引入更多方向，这使得我们能够提高路径之间的角度精度。这种方法同时能够考虑连接远距离网格点的长弧段，以便生成桥梁、隧道等特殊路径。

### 5.1 Path segment masks

由于显式存储网格节点的连接弧会消耗大量的内存，所以使用一组路径段掩码来存储网格点之间的连接信息，记作 $\mathcal{M}_k$。

#### Definition

将路径段掩码 $\mathcal{M}_k$ 定义为连接原点 $(0,0)$ 到点 $(i,j)\in[-k,k]^2$ 的段的集合，其中 $i,j$ 的最大公约数为 1(这是为了排除冗余路径方向)。使用这样的定义，而不是检查所有 $(i,j)\in[k,k]^2$ 的 $(2k+1)^2$ 段，可以在应用离散最短路径时避免冗余路径检查。

#### Influence of the size of the masks

增大路径段mask的尺寸能够以更好的角度分辨率检测路径，并减少方向限制的影响，但是代价就是最短路径算法中迭代次数增加。Table 1 展示了路径段数量和角度分辨率的关系。  
<img src="/imgs/Road Networks/Table1.png" style="zoom:80%;" />

如果路径段数 k 太小生成的路径，会不切实际，乃至忽略坡度影响。随着 k 的增加，生成路径才更为现实。  
<img src="/imgs/Road Networks/Figure9.png" style="zoom:80%;" />

Table 2 展示了在 60x60 的网格上使用不同路径段掩码计算各向异性最短路径的时间(以秒为单位)、成本(in arbitrary unit) 和 长度(以米为单位)。时间随着 $O(k^2)$ 增加。

实验表明，随着掩码大小的增加，最短路径的成本会收敛到一个极限。一般来说，选择 k = 5 被证明是在速度和角度分辨率之间生成逼真路径的良好折中。  
<img src="/imgs/Road Networks/Table2.png" style="zoom:80%;" />

### 5.2  Curvature

仅靠路径段掩码生成的路径往往存在过多急转弯，因此需要引入曲率限制：将离散的二维地形域 $\Omega\subset\mathbb{R}^2$ 拓展到三维域 $\Omega\times[0,2\pi]$，额外加入方向维度，该域表示 $\Omega$ 内所有位置以及所有可能的朝向。将角度域 $[0,2\pi]$ 离散为 $m$ 个区间。因此，需要计算的是在 $n^2\times m$ 的三位定向网格上的离散各向异性最短路径，记作  $p_{ija}$，其中 $i,j\in[0,n]^2,a\in[0,m-1]$。分段路径掩码 $\mathcal{M}$ 被推广为拓展掩码记作 $\mathcal{E}$，令 $\mathcal{E(p_{ija})}$ 指所有点 $\mathcal{M(p_{ij})}$ 在 $a\in[0,m-1]$ 下的重复集合。

这样可以生成更真实的路径，但是计算量会上升。

### 5.3 Tunnels and bridges

隧道和桥梁可以视为两个远距离网格点的长路径段，所以可以拓展路径段掩码后作为路径段统一处理。

为了不失一般性，我们首先考虑隧道，和地面道路一样，我们引入通用隧道掩码，记作 $\mathcal{T}$，表示连接两个网格节点的哪些路径应有最短路径算法作为隧道处理。这些路段具有最小和最大距离，分别记作 $r_i,r_e$，对应某种类型隧道或桥梁的最小和最大长度，因此，定义：
$$
\begin{aligned}
\mathcal{T}(p_{ij})=\{q\neq p_{ij}|r_{i}\le \parallel p_{ij}-q_{}\parallel\le r_e\}
\end{aligned}
$$
<img src="/imgs/Road Networks/Figure12.png" style="zoom:80%;" title="隧道路径段掩码 T (pij)。出于清晰起见，只绘制了三个段。"/>

路径规划的第3步修改如下，在处理候选网格节点 $p_{ij}$ 的时候，计算并更新两个集合 $q\in\mathcal{M}(p_{ij}),q\in\mathcal{T}(p_{ij})$ 中网格节点的值。  
3.对于所有的点 $q\in\mathcal{M}(p_{ij})$，评估地面道路成本 $c(p_{ij},q)$；  
对于所有的点 $q\in\mathcal{T}(p_{ij})$，评估隧道成本 $c(p_{ij},q)$。  
如果成本 $c(p_{ij},q)<c(q)$，则将 $q$ 的前驱节点设为 $p_{ij}$。

桥梁的定义方式相同，将对应的掩码定义为 $\mathcal{B}$。

### 5.4 Stochastic sampling

检测所有可能的隧道、桥梁需要评估大量的弧，弧的数量随着搜索域外半径 $r_e$ 的增长呈二次方速度上升。因此使用一种随机抽样技术加快计算。在该算法的每一步中，不对集合 $\mathcal{T}(p_{ij})$ 中所有网格节点的桥梁和隧道函数值进行评估，而只对分段路径的一个受限的子集 $S(p_{ij})\subset\mathcal{T}_k(p_{ij})$ 进行计算。候选网格节点是在算法每次的迭代过程中，使用低差异序列在空心圆盘内随机选择一些网格点获得的。  
这样虽然不能得到最“正确”的结果，但是能节省非常多的计算成本，是 acceptable。

## 6. Procedural generation of road models

本节中介绍如何从离散最短路径生成道路、隧道和桥梁，分三个步骤进行。  
首先，将离散最短路径转换为一组分段回旋曲线(曲率连续)，来表示道路轨迹。使用回旋曲线是因为回旋曲线符合现实中的曲率约束。轨迹进一步被分段，以识别哪些部分应该被实例化成道路、隧道或桥梁。沿轨迹修改地形和植被，进行挖掘和堆填，并从参数化程序模型中生成道路、隧道和桥梁的网格表现。

### 6.1 Trajectory computation

回想一下，之前离散各向异性最短路径算法生成了一条路径，定义为 $\rho^*=\{p_k\}_{k\in[0,n]}$，其中点 $p_k$ 是位于离散搜索域 $\Omega$ 的网格点 $p_{ij}$ 上。从控制点 $p_k$ 创建一条分段螺线曲线，记作 $\Gamma$，以便生成平滑真实的道路轨迹。

由于平滑过程中，曲线 $\Gamma$ 中对应的原本的地表路段某些部分可能发生变化，略微地位于地形内部或者地形之上。因此，需要对轨迹进行根据地形高度的分段，确定哪些部分是平滑后的通用道路、隧道或者是桥梁。

该分割是通过对 $\Gamma$ 进行均匀采样生成一条分段线性曲线，表示为 $\Gamma^*=\{q_k\},k\in[0,n]$。对于该离散曲线中的每个点计算 $h(q_k)$，定义是 $q_k$ 的高度和其在地形表面上的投影之间的高度差。假设 $h_{\mathcal{T}}$ 为修建隧道的最小高度值，$h_{\mathcal{B}}$ 为修建桥梁的最小高度值。分割标准：  
若 $h(q_k)<h_{\mathcal{T}}$，$q_k$ 记为隧道点；  
若 $h_\mathcal{T}<h(q_k)<h_\mathcal{B}$，$q_k$ 记作道路点；  
否则，记作桥梁点。  
(这里的分段规则很乱，原文的文本描述和图例也对不上= =，看个乐吧，原文图例中的Excavation和Embankment是后面会提到的)

流程结束后，轨迹 $\Gamma^*$ 被一致定义为由渐变曲线样条构成的分段线性曲线，并且每个点 $q_k$ 都被明确标记为道路、隧道或桥梁。

### 6.2 Road generation

用 $R$ 表示道路模型的宽度，将曲线 $\Gamma^*$ 的领域定义为：
$$
\begin{aligned}
\Omega_\Gamma=\{p\in\Omega|d(p,\Gamma)<R\}
\end{aligned}
$$
道路生成分两个步骤进行。首先，移除位于 $\Omega_\Gamma$ 中的树木实例来清楚道路附近的现有植被。地形也在轨迹附近进行如下修改。首先，对区域 $\Omega_\Gamma$ 的地形网格进行自适应细化。对于细化网格的每个顶点 $v$ 计算到轨迹的距离 $d(v,\Gamma)$。Excavation 和 Embankment 按照如下方式进行：

1. 如果 $d(v,\Gamma)<r$，将顶点 $v$ 的高度设置为曲线上对应最近点的高度。
2. 如果 $r<d(v,\Gamma)<R$，通过将道路的轮廓曲线 $C(d(v,\Gamma))$ 和 地形高度进行插值，来计算顶点高度。

根据参数化定义和离散分段轨迹生成道路、隧道和桥梁模型，能够自动适应di'x

$\mathbb{R}$,$\mathcal{P},\mathscr{P}$

# Authoring Hierarchical Road Networks

* Delaunay triangulation：平面上的点集P是一种三角剖分得到的，使得P中没有点严格处于剖分后中任意一个三角形外接圆的内部。
* Gabriel Graph：

  1. 先构造以 $p_i$ 和 $p_j$ 的连线为**直径**的圆：圆的圆心是 $p_i,p_j$ 的中点，半径是两点欧氏距离的一半，圆的边界恰好经过 $p_i$ 和 $p_j$；
  2. 边的生成规则：只有当该直径圆的内部（不含边界）没有任何其他节点时，$p_i$ 和 $p_j$ 之间才会存在一条边。
* Relative Neighborhood Graph，简称RNG。  
  仅当不存在第三个点 $p_k$，同时满足 $d(p_i,p_k)<d(p_i,p_j)$ 且 $d(p_j,p_k)<d(p_i,p_j)$ 时，  $p_i$ 和 $p_j$ 之间才会存在一条边。其中是 $d(·)$ 两点间的欧氏距离。
  这个规则可以用几何化的邻域直观理解：构造两个以$p_i,p_j$ 为圆心，以两点间距$d(p_i,p_j)$为半径的圆，两个圆的重叠区域 (形似月牙的透镜区域) 内没有任何其他点时，就保留 $p_i$ 和 $p_j$ 之间的边，这个重叠区域就是RNG的「相对邻域」。

# Reference

[2011-network.pdf](https://perso.liris.cnrs.fr/egalin/Articles/2011-network.pdf)  
[(PDF) Procedural Generation of Roads](https://www.researchgate.net/publication/229707505_Procedural_Generation_of_Roads)  
[Houdini程序化道路（一）：确定路口范围 - 大大凯的文章 - 知乎](https://zhuanlan.zhihu.com/p/334788179)  
[Houdini程序化道路（三）：具体实现方法 - 大大凯的文章 - 知乎](https://zhuanlan.zhihu.com/p/438570925)  
[houdini路网 - 杨超wantnon的文章 - 知乎](https://zhuanlan.zhihu.com/p/73570541)
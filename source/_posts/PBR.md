---
title: PBR备忘笔记
date: 2022-06-17 20:05:13
tags: 
typora-root-url: ./..
---

记录一些有关`PBR`的知识，以免自己忘记了。网页居然不支持LATEX。
回头再看自己这篇文章写的稀烂，入门学习PBR还是看[LearnOpenGL]([LearnOpenGL - Theory](https://learnopengl.com/PBR/Theory))吧

<!--more-->

# 几个问题

什么是`PBR`呢？
即`Physicallly-Based-Rendering`，中文可以叫做基于物理的渲染。是一种渲染技术。

为什么要有PBR呢？
随着游戏行业的发展玩家对于画面的要求也是越来越严格，对于真实性和细节有更进一步的要求。
而PBR能做到的就是表现十分**写实的材质**，而且还可以制造出**风格化**迥异的资源。

# PBR

## 初步了解

### 什么叫基于物理

物理的渲染包含了三个条件：

* 基于物理的材质

* 基于物理的灯光

* 基于物理适配的摄像机

### 基于物理的材质

这种材质归根结底是对于现实世界的一种模拟。

#### 物理材质的三个条件

* 必须遵循能量守恒，比如传播过程的光能+光转换为其他形式的能量不会超过光发出是的能量。

* 基于微表面模型

* 基于物理的`BRDF`

##### 微平面理论

为了模拟现实世界的物体表面，我们认为物体表面都是由很小、随机朝向的镜面平面(发生镜面反射的小平面)。
光滑表面的小镜面分布更加规则。
而粗糙表面的小镜面十分混沌。

##### 能量守恒

高中物理(
对于粗糙平面更多的光被转换为其他能量，反射强度低。
光滑平面，由于折射、散射更少，镜面反射会更强。

###### 反射率方程

我们在模拟过程中，往往用反射率方程描述能量守恒

<div style="margin:left;width:100%"><img src="/imgs/PBR/math1.png" alt="math1"></div>

而重点就在于反射比例，它规定了光在物体表面如何反射。

**BRDF 双向反射分布函数**

粗糙来讲BRDF就是描述光在物体表面如何反射。
*双向*：是说相机方向和光源方向调换后，他们的反射规律是一致的，即一束光打到表面分配到各个方向这个过程是可逆的，可以变成“分配出去的那些光”再汇聚成原来打进来的光，有点类似光路可逆。

当光打到物体表面，会发生散射和高光反射。散射是向各个角度射出强度相近的光，高光反射就是我们常常在物体表面看到光斑。

而与BRDF对应的还有BTDF，BTDF是双向透射分布函数，从名字就可以大概知道这个函数描述的是物体投射后在表面向各方向如何发射光线。

<div style="margin:auto;width:30%"><img src="/imgs/PBR/BSDF.png" alt="BSDF示意"></div>

BRDF+BTDF就是BSDF，双向散射分布函数，感性来讲有了BSDF我们就知道，当一束光打到一块玻璃砖上表面时，上表面光如何发射(BRDF)，下表面如何透射(BTDF)。

一般的BRDF：$f_{r}(p,w_{i},w_{o})$或者$f_{r}(w_{i},w_{r})$
即通过入射光反向和反射光方向，计算得到反射比例。

**BRDF计算**

我们可以把BRDF分为两个部分进行计算，即**漫反射**和**高光反射**。

* 漫反射：可以采用经典的Lambert模型，这只是一种经验模型
  我们根据光与平面法线的夹角计算光强，夹角越大光强越小，可以参照太阳对于地球夏冬温度的影响。
  
  <div style="margin:left;width:100%"><img src="/imgs/PBR/math2.png" alt="math2"></div>

* 高光反射：
  
  * Phong模型，根据反射方向和视角方向衡量高光的强弱。
  
  * Blinn-Phong：根据半角向量和法线方向，半角向量即视角方向和入射方向的“角分线”，该向量越接近法线说明反射路径越接近“`入射反向`->`|法线`->`视角方向`”，反射就越强。
    半角向量的引入极大的节约了计算过程，由于Phong模型需要计算反射反向所以比较于Blinn-Phong效率低。而Blinn-Phong已经有很好的效果了。

<div style="margin:auto;width:100%"><img src="/imgs/PBR/phong.png" alt="PhongAndBlinnPhong"></div>

* 基于物理的高光：
  
  <div style="margin:left;width:100%"><img src="/imgs/PBR/math3.png" alt="math3"></div>

###### 法线分布函数(NDF)

<div style="margin:left;width:100%"><img src="/imgs/PBR/math4.png" alt="math4"></div>

**与Blinn-Phong模型区别**：二者虽然都是法线对于光的影响，但是Blinn-Phong终究是经验模型，NDF更贴切实际，尤其是在末端部分，GGX-NDF末端过度平滑更加具有真实感更贴近物理，而Blinn-Phong相比较为生硬。

###### 几何遮蔽

我们先前讨论的微平面理论，把每个小平面单独计算互不干扰，但是在实际中面与面的遮挡关系是存在的，比如人的内眼角和鼻子交汇处(这里也常常运用AO贴图，这个例子不太恰当但是主要是说明这种遮挡现象)，真实物体的表面凹凸不平，微平面间势必存在相互遮挡的关系。

<div style="margin:auto;width:80%"><img src="/imgs/PBR/jihe.png" alt="几何遮蔽"></div>

左侧为几何遮蔽示意，右侧为几何阴影示意。
对于几何遮蔽我们通常运用统计学(概率论)的有关知识近似计算相互遮蔽，因为我们很难一个一个去算微平面间的遮挡。

<div style="margin:auto;width:80%"><img src="/imgs/PBR/几何遮蔽.png" alt="几何遮蔽公式"></div>

<div style="margin:left;width:100%"><img src="/imgs/PBR/math5.png" alt="math5"></div>

###### 菲涅尔方程

被反射的光线对比光线被折射部分所占的比率，描述了反射相对于折射的强度。比如我们垂直观察水面很容易看到水底，而我们远离水面水平观察时往往只能看到水面波光粼粼的样子这就是菲涅尔反射，ta描述了反射相对于折射的强度。
还有一个典型的物理例子就是在夏天时路远处的柏油会发生镜面反射。

<div style="margin:left;width:100%"><img src="/imgs/PBR/math7.png" alt="math7"></div>

## Last

<div style="margin:left;width:100%"><img src="/imgs/PBR/math6.png" alt="math6"></div>

材质的效果关键就是FDG，但是BRDF不止于FDG，而PBR也不止于BRDF。

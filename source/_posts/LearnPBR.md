---
title: Learn PBR
tags: 图形学
math: true
index_img: /imgs/LearnPBR/pbr.png
banner_img: /imgs/LearnPBR/pbr.png
date: 2023-11-14
typora-root-url: ../
---

关于PBR的一些知识点

# Learn PBR

# PBR理论

## **简介**

**PBR**（**Physically Based Rendering**）译成中文是基于物理的渲染。它是利用真实世界的原理和理论，通过各种数学方法推导或简化或模拟出一系列渲染方程，并依赖计算机硬件和图形API渲染出拟真画面的技术。

### **PBR** **特征**

更高质量的着色效果和更多复杂的材质特性。

- 表面细节
- 物体粗糙度
- 区别明显的金属和绝缘体
- 物体的浑浊程度
- 菲涅尔现象：不同角度有不同强度的反射光
- 半透明物体
- 多层混合材质
- 清漆效果
- 其它更复杂的表面特征

> 近今年，PBR的技术主要朝着更逼真、更复杂、效能更好的方向，或是结合若干种模型的综合性技术迈进。代表性技术有：
>
> - PBR Diffuse for GGX + Smith (2017)
> - MultiScattering Diffuse (2018)
> - Layers Material（分层材质）
> - Mixed Material（混合材质）
> - Mixed BxDF（混合BxDF）
> - Advanced Rendering（进阶渲染）

### **OutPut**

![output](/imgs/LearnPBR/output.PNG)

## **PBR** **和 游戏引擎**

### **UE4 的** **PBR**

- **Base Color**，基础的纹理颜色 非金属物体只有**单色**，即强度

| Material                 | Base Color Intensity |
| ------------------------ | -------------------- |
| 木炭(Charcoal)           | 0.02                 |
| 新沥青(Fresh asphalt)    | 0.02                 |
| 旧沥青(Worn asphalt)     | 0.08                 |
| 土壤(Bare soil)          | 0.13                 |
| 绿草(Green Grass)        | 0.21                 |
| 沙漠沙(desert sand)      | 0.36                 |
| 新混泥土(Fresh concrete) | 0.51                 |
| 海洋冰(Ocean Ice)        | 0.56                 |
| 鲜雪(Fresh snow)         | 0.81                 |

- **金属**材质，在 Linear 空间的值

| 材质(Material) | 基础色(BaseColor)     |
| -------------- | --------------------- |
| 铁(Iron)       | (0.560, 0.570, 0.580) |
| 银(Silver)     | (0.972, 0.960, 0.915) |
| 铝(Aluminum)   | (0.913, 0.921, 0.925) |
| 金(Gold)       | (1.000, 0.766, 0.336) |
| 铜(Copper)     | (0.955, 0.637, 0.538) |
| 铬(Chromium)   | (0.550, 0.556, 0.554) |
| 镍(Nickel)     | (0.660, 0.609, 0.526) |
| 钛(Titanium)   | (0.542, 0.497, 0.449) |
| 钴(Cobalt)     | (0.662, 0.655, 0.634) |
| 铂(Platinum)   | (0.672, 0.637, 0.585) |

- **粗糙度(Roughness)**：表面的粗糙程度, [0,1]，越粗糙高光越弱。

![roughness](/imgs/LearnPBR/roughness.png)

![roughness2](/imgs/LearnPBR/roughness2.png)

上为非金属，下为金属，粗糙度从0至1

- **金属度(Metallic)**：表示材质像金属的程度，0是绝缘体(电介质)，1 是金属，金属只有镜面反射，没有漫反射。

![metallic](/imgs/LearnPBR/metallic.png)

金属度从0至1

- **镜面度(Specular)**：表示物体镜面反射的强度，从0(完全没有镜面反射)到1(完全镜面反射)

![specular](/imgs/LearnPBR/specular.png)

From 0 ~ 1

| 材质(Material) | 镜面度(Specular) |
| -------------- | ---------------- |
| 草(Glass)      | 0.5              |
| 塑料(Plastic)  | 0.5              |
| 石英(Quartz)   | 0.57             |
| 冰(Ice)        | 0.224            |
| 水(Water)      | 0.255            |
| 牛奶(Milk)     | 0.277            |
| 皮肤(Skin)     | 0.35             |

### **Unity 的** **PBR**

- **Albedo**，和 UE 的 Base Color 一样。 可以用颜色或者Tex
- **Metallic**，可以用金属贴图，但是用了 Smoothness 参数就消失了
- **Smoothness**，光滑度，和 UE 的 粗糙度正相反
  - **Smoothness Source**，指定光滑度的存储通道，可选金属度、镜面贴图的 Alpha 或基础色的 Alpha
- **Occlusion**：遮蔽图，指定材质接收间接光的光照强度和反射强度。 能够使物体经常是暗部的位置更暗，比如人的眼窝，脸和脖子的交接处。
- **Fresnel**，物体边缘或者说物体法线和视线角度增大，物体的反射能力更强，Unity 里面是自动处理，越光滑越强，越粗糙 Fresnel 越弱。

## **PBR** **基本原理**

满足以下条件的光照模型才能称之为PBR光照模型：

- 基于微平面模型（Be based on the microfacet surface model）。
- 能量守恒（Be energy conserving）。
- 使用基于物理的BRDF（Use a physically based BRDF）。

### **微表面理论(Microfacet)**

很多 PBR 技术都是基于理论 认为**在微观上**，所有的物体表面都是由很多的朝向不一的微小平面组成的。

> 真实世界的物体表面其实不一定是这样的微小平面，可能会有弧度，甚至坑坑洼洼，但我们从肉眼观察、甚至*光栅化后的像素尺度*来看待的话，这种假设的结果和实际差别甚微。

![microfacet](/imgs/LearnPBR/microfacet.png)

基于这种假设，没有任何表面是光滑的，但由于这些微平面已经微小到逐像素无法对其进行细分，所以假设一个粗糙度 Roughness，用统计学的方法去估算微表面的粗糙度。

...

### **Energy Conservation (能量守恒)**

在 Microfacet 中采用近似的能量守恒定律，出射光的总能量不能超过入射光的总能量(不含自发光)，

所以材质粗糙度越大，反射的范围越大，整体的亮度会变低。

#### **镜面反射(specular)和漫反射(diffuse)**

一束光打到物体，会发生 **reflection** 反射 和 **refraction** 折射。反射的光直接离开，不进入物体发射了镜面反射光；折射的光进入物体内发生了吸收或散射，产生漫反射。 折射后的光若没被吸收会继续前进，在物体内部发生光和微粒的碰撞，这时有一部分能力转化为热量，有些光经过多次折射从表面射出，便形成漫反射光。

![reflect](/imgs/LearnPBR/reflect.png)

*照射在平面的光被分成镜面反射和折射光，折射光在跟物体微粒发生若干次碰撞之后，有可能发射出表面，成为漫反射。* 通常情况下，PBR会简化折射光，将平面上所有折射光都视为被完全吸收而不会散开。而有一些被称为次表面散射(Subsurface Scattering)技术的着色器技术会计算折射光散开后的模拟，它们可以显著提升一些材质（如皮肤、大理石或蜡质）的视觉效果，不过性能也会随着下降。 金属(Metallic)材质会立即吸收所有折射光，故而金属只有镜面反射，而没有折射光引起的漫反射。

根据上面的能量守恒关系，可以先计算镜面反射部分，此部分等于入射光线被反射的能量所占的百分比。而折射部分可以由镜面反射部分计算得出。

float kS = calculateSpecularComponent(...); // 反射/镜面部分

float kD = 1.0 - kS;                        // 折射/漫反射部分

ks + kd 不会超过1，所以近似地能量守恒。

### **Reflectance Equation**

$$L_0(p,w_0)=\int\limits_{\Omega}f_r(p,w_i,w_0)L_i(p,w_i)\,\mathrm{d}w_i$$

## **辐射度量学**

### **概念表**

![chart1](/imgs/LearnPBR/chart1.png)

![chart2](/imgs/LearnPBR/chart2.png)

> 微分符号 d 的含义：首先来说下微分的定义：设f(x)定义在区间(a,b)上，x∈(a,b),给定自变量x的一个增量Δx，得到函数的一个增量Δy，如果有Δy=f(x+Δx)-f(x)=AΔx+o(Δx)(Δx→0)，则y=f(x)称在点x可微，函数增量的线性主部AΔx称为函数的微分，记为dy=df(x)=AΔx

### **辐射通量（Radiant Flux)**

光源单位时间内的输出$$\Phi=\frac{\mathrm{d}Q}{\mathrm{d}t}$$

> 光是由多种不同波长的能量集合而成，每种波长与一种特定的（可见的）颜色相关。因此一个光源所放射出来的能量可以被视作这个光源包含的所有各种波长的一个函数。波长介于390nm（纳米）到700nm的光被认为是处于可见光光谱中，也就是说它们是人眼可见的波长。

传统物理学上的辐射通量将会计算这个由不同波长构成的函数的总面积，这种计算很复杂，耗费大量性能。在PBR技术中，不直接使用波长的强度，而是使用三原色编码（RGB）来简化辐射通量的计算。虽然这种简化会带来一些信息上的损失，但是这对于视觉效果上的影响基本可以忽略。

### **Radiant Insensity(辐射强度)**

单位球面上，一个光源向单位立体角所投送的辐射通量。

$$I=\frac{\mathrm{d}\Phi}{\mathrm{d}w}$$

the power per unit angle

假如光源均匀向四周发散

$$I = \frac{\Phi}{4\pi}$$

### **Irradiance**

The power per (perpendicular/projected) unit area incident on a surface point.

$$E(x)=\frac{\mathrm{d}\Phi(x)}{\mathrm{d}A^\perp }$$

面要和光源垂直。

即 Lambert‘s Consine Law

### **Radiance**

The radiance(luminance) is the power emitted, reflected, transmitted or received by a surface, *per unit solid angle, per projected unit area.* 在单位立体角并且在单位的面积上

$$L(p,w)=\frac{\mathrm{d}^2\Phi(p,w)}{\mathrm{d}w\,\mathrm{d}A\cos\theta}=\frac{\mathrm{d}^2\Phi(p,w)}{\mathrm{d}w\,\mathrm{d}A^\perp}$$

- Irradiance: power per projected unit area
- Intensity: power per solid angle

So

- Radiance : Irrandiance pre solid angle
- Radiance : Intensity pre unit projected area

Irradiance 是 dA 收到的能量

Irradiance per solid angle 是 dA 的能量向某一个方向辐射

![Radiance](/imgs/LearnPBR/Radiance.png)

该图为 Irrandicance per solid angle, 这的 cos 是不是应该写进 E 里面……

Incident Radiance 延申理解：

一小块面积 dA 向某个方向辐射的能量

反过来就是，从一个方向打向一个小面，到达这个面时的能量

$$L(p,w)=\frac{\mathrm{d}I(p,w)}{\mathrm{d}A\cos\theta}=\frac{\mathrm{d}I(p,w)}{\mathrm{d}A^\perp}$$

### **Irradiance vs. Radiace**

$$\mathrm{d}E(p,w)=L_i(p,w)\cos\theta\,\mathrm{d}w\\$$

两边同时积分

$$E(p)=\int_H^2L_i(p,w)\cos\theta\,\mathrm{d}w\\ Uint\;Hemisphere:H^2$$

每个方向过来到 A 的能量, 这和渲染方程是异曲同工的。

再回到渲染方程

$$L_0(p,w_0)=\int\limits_\Omega f_r(p,w_i,w_0)\underbrace{L_i(p,w_i)n\cdot w_i\,\mathrm{d}w_i}_{Radiace}$$

其中只有 f 项待解。

## **BRDF(双向反射分布函数)** 

Bidirectional Reflectance Distribution Function，BRDF

一个使用入射光方向ωi作为输入参数的函数，输出参数为出射光ωo，表面法线为n，参数a表示的是微平面的粗糙度。

![BRDF](/imgs/LearnPBR/BRDF.svg)

BRDF 描述了不透明物体表面每个单独光线，对最终反射光线的影响。也就是光线打到该表面如何反射，

假设BRDF描述的是完全镜面物体，只有当出射光线方向w0 完全符号 入射光线方向wi 的反射方向时，返回值会为1.0，其余情况为0.

$$f_r(p,w_0,w_i)=\frac{\mathrm{d}L_0(p,w_0)}{\mathrm{d}E(p,w_i)}=\frac{\mathrm{d}L_0(p,w_0)}{L_i(p,w_i)\cos\theta_i\,\mathrm{d}w_i}$$

BRDF 有多种模拟表面光照的算法，实时渲染所用的基本是 **Cook-Torrance BRDF**

### **参考**

- **PBRT-BRDF章节：**[Surface Reflection (pbr-book.org)](https://www.pbr-book.org/3ed-2018/Color_and_Radiometry/Surface_Reflection)

### **Cook-Torrance BRDF**

$$f_r=k_df_{lambert}+k_sf_{cook-torrance}\\ k_d是入射光被折射比例，\\ k_s是被镜面反射比例$$

而左侧的 f_lambert 表示漫反射部分，Lambertian Diffuse，一般是恒定的算式：

$$f_{lambert}=\frac{c}{\pi}$$

c 表示 Albedo，π 是为了归一化 漫反射，因为后面要积分的

#### **BRDF的高光项**

$$f_{cook-torrance}=\frac{DFG}{4(w_0\cdot n)(w_i\cdot n)}$$

- **D( Normal Distribution Function, NDF )** : 法线分布，估算在表面粗糙度的影响下，表现出的表面法线和半角向量(half Dir)的一致性或者说微表面的数量。 估算微表面的主要函数。
- **F( Fresnel Equation )**：菲涅尔方程，描述菲涅尔现象，当视线和表面法线夹角变大时更容易发生光的反射。 不同表面角下，表面反射光线的比例。
- **G( Geometry Function )** : 几何函数，描述了微表面自成影的现象，当一个微表面特别粗糙时，彼此之间可能相互遮挡，从而减少反射的光线。

[brdf为什么要定义为一个单位是sr-1的量？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/28476602/answer/41003204) [brdf为什么要定义为一个单位是sr-1的量？ - 文刀秋二的回答:](https://www.zhihu.com/question/28476602/answer/41003204)

### **D(Normal Distribution Function, NDF)**

描述和微表面法线和半角向量的一致性，从统计学上近似 **Trowbridge-Reitz GGX(GGXTR) :** 

$${NDF}_{GGXTR}(n,h,\alpha)=\frac{\alpha^2}{\pi((n\cdot h)^2(\alpha^2-1)+1)^2}\\ h=normalize(viewDir+lightDir)\\ \alpha={roughness}^2$$

![ggx](/imgs/LearnPBR/ggx.png)

[Trowbridge-Reitz GGX | Desmos](https://www.desmos.com/calculator/eks25xlifv?lang=zh-CN)

当 粗糙度 为零时函数值变为零

```OpenGL
float DoubleDistributeGGX(float3 N, float3 H, float roughness)
{
    float a = roughness * roughness;
    float a2 = a * a;
    float NdotH = max(0.0, dot(N, H));
    float NdotH2 = NdotH * NdotH;

    float p = (NdotH2 * (a2 - 1.0) + 1.0);
    p = PI * p * p;
    return a2 / p;
}

roughness += 0.0001;
float NDF = DoubleDistributeGGX(N, H, roughness);
```

[2013SiggraphPresentationsNotes-26915738.pdf (unrealengine.com): Specular D](https://cdn2.unrealengine.com/Resources/files/2013SiggraphPresentationsNotes-26915738.pdf)

#### **Generalized-Trowbridge-Reitz（GTR）分布**

GTR分布不具备形状不变性（shape-invariant），导致其发布以来，无法被广泛使用。

$$D_{GTR}(m)=\frac{c}{(1+(n\cdot m)^2(\alpha^2-1))^\gamma}$$

- 关于形状不变性的好处，可以总结为：
  - 方便推导出该NDF归一化的各向异性版本
  - 方便推导出遮蔽阴影项 Smith G
  - 方便基于NDF或可见法线分布推导其重要性采样
    - 对于Smith G，可用低维函数或表格处理所有粗糙度和各向异性

#### **参考**

[【基于物理的渲染（PBR）白皮书】（四）法线分布函数相关总结 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/69380665):关于形状不变性。

### **F(Fresnel Equation)**

$$F_{Schlick}(h,v,F_0)=F_0+(1-F_0)(1-(h,v))^5$$

F_0 表示的基础反射率，利用折射指数(Indices Of Refraction)求得，F_0 越大菲涅尔反射现象越强。

当越是向掠射角(又名切线角，和正視角相差90度)方向去看，菲涅尔现象越强，反光效果越明显。

![fresnel](/imgs/LearnPBR/fresnel.png)

**Fresnel-Schlick**只适用于绝缘体的表面算法。 金属表面需要其他的菲涅尔方程模拟。但是这样做很不方便，所以： 预计算出平面对法线入射的结果(F0)，基于观察角的Fresnel-Schlick近似对这个值进行插值，用这种方法进一步估算。 这样就可以使用同一个公式了。

平面对于法向入射的响应或者说基础反射率可以在一些大型数据库中找到，比如[RefractiveIndex](http://refractiveindex.info/)。

所有电介质材质表面的基础反射率都不会高于0.17，这其实是例外而非普遍情况。导体材质表面的基础反射率起点更高一些并且（大多）在0.5和1.0之间变化。此外，对于导体或者金属表面而言基础反射率一般是带有色彩的，这也是为什么F0要用RGB三原色来表示的原因（法向入射的反射率可随波长不同而不同）。这种现象我们**只能**在金属表面观察的到。

![chart3](/imgs/LearnPBR/chart3.png)

由于绝缘体和金属体存在多种的差异，其各自独有的特性引出了金属工作流，我们使用一个金属度调节材质表面特性，这个参数并非 非零即一 的布尔值，是因为我们要描述一些比如沙子、颗粒和被刮蹭后的金属表面，所以要有一个[0,1]的范围进行调整。

我们通过预计算金属和绝缘体的 F_0 进行 Fresnel-Schlick 近似，但是对于金属表面通常这样做

```OpenGL
vec3 F0 = vec3(0.04,0.04,0.04);
F0 = mix(F0, surfaceColor.rgb, metalness);
```

我们为大多数电介质表面定义了一个近似的基础反射率。F_0取最常见的电解质表面的平均值，这又是一个近似值。不过对于大多数电介质表面而言使用0.04作为基础反射率已经足够好了，而且可以在不需要输入额外表面参数的情况下得到物理可信的结果。然后，基于金属表面特性，我们要么使用电介质的基础反射率要么就使用F_0来作为表面颜色。因为金属表面会吸收所有折射光线而没有漫反射，所以我们可以直接使用表面颜色纹理来作为它们的基础反射率。

```OpenGL
vec3 fresnelSchlick(float cosTheta, vec3 F0)
{
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}
// cosTheta = dot(N,V)
```

这里的N是收到粗糙度影响的半角向量，在后面计算IBL时，因为预计算没办法考虑粗糙度，所以我们需要一个受粗糙度影响的 fresnelSchlick

```OpenGL
vec3 fresnelSchlick(float cosTheta, vec3 F0, float roughness)
{
    return F0 + (max(vec3(1.0 - roughness), F0) - F0) * pow(1.0 - cosTheta, 5.0);
}
```

关于金属和绝缘体的菲涅尔，在 PBRT 中有提到，8.2.1 Fresnel Reflection： [Specular Reflection and Transmission (pbr-book.org)](https://www.pbr-book.org/3ed-2018/Reflection_Models/Specular_Reflection_and_Transmission)

> 1. The first class is *dielectrics*, which are materials that don’t conduct electricity. They have real-valued indices of refraction (usually in the range 1-3) and transmit† a portion of the incident illumination. Examples of dielectrics are glass, mineral oil, water, and air.
> 2. The second class consists of *conductors* such as metals. Valence electrons can freely move within the their atomic lattice, allowing electric currents to flow from one place to another. This fundamental atomic property translates into a profoundly different behavior when a conductor is subjected to electromagnetic radiation such as visible light: the material is opaque and reflects back a significant portion of the illumination. A portion of the light is also transmitted into the interior of the conductor, where it is rapidly absorbed: total absorption typically occurs within the top 0.1 μm of the material, hence only extremely thin metal films are capable of transmitting appreciable amounts of light. We ignore this effect in `pbrt` and only model the reflection component of conductors. In contrast to dielectrics, conductors have a complex-valued index of refraction n=n0+ik.
> 3. Semiconductors such as silicon or germanium are the third class though we will not consider them in this book.
> 4. The first class is dielectrics, which are materials that don't conduct electricity. They have real-valued indices of refraction (usually in the range 1-3) and transmitt a portion of the incident illumination. Examples of dielectrics are glass, mineral oil, water, and air.
> 5. The second class consists of conductors such as metals. Valence electrons can freely move within the their atomic lattice, allowing electric currents to flow from one place to another. This fundamental atomic property translates into a profoundly different behavior when a conductor is subjected to electromagnetic radiation such as visible light: the material is opaque and reflects back a significant portion of the illumination. A portion of the light is also transmitted into the interior of the conductor, where it is rapidly absorbed: total absorption typically occurs within the top 0.1 um of the material, hence only extremely thin metal films are capable of transmitting appreciable amounts of light. We ignore this effect in pbrt and only model the reflection component of conductors. In contrast to dielectrics, conductors have a complex-valued index of refraction = n + ik.
> 6. Semiconductors such as silicon or germanium are the third class though we will not consider them in this book.
>
>   第一类是介电材料，这是一种不导电的材料。它们具有实值折射率(通常在1-3范围内)，并透射一部分入射光。电介质的例子有玻璃、矿物油、水和空气。
>
>   第二类由金属等导体组成。价电子可以在其原子晶格内自由移动，从而使电流从一个地方流向另一个地方。当导体受到电磁辐射(如可见光)时，这种基本的原子性质会转化为一种截然不同的行为:这种材料是不透明的，会反射回相当一部分照明。一部分光也被传输到导体的内部，在那里它被迅速吸收:完全吸收通常发生在材料的顶部0.1微米内，因此只有极薄的金属薄膜才能传输相当数量的光。我们在pbrt中忽略了这种影响，只对导体的反射成分进行了建模。与电介质相比，导体具有复值折射率= n + ik。
>
>   硅或锗等半导体是第三类，但我们在本书中不考虑它们。

### **G(Geometry Function)**

几何遮蔽模拟微表面的互相遮挡导致光线能量丢失或减少的现象。

类似 NDF，也使用 Roughness 作为输入，粗糙度越高意味着几何遮蔽的概率越大。 几何遮蔽有 GGX 和 Schlick-Beckmann 组合而成的模拟函数 **Schlick-GGX**：

$$G_{SchlickGGX}(n,v,k)=\frac{n\cdot v}{(n\cdot v)(1-k)+k}$$

![G](/imgs/LearnPBR/G.png)

这里的 k 由粗糙度 α 计算得来，用于直接光照和 IBL 光照的几何函数参数:

$$k_{direct}=\frac{(\alpha+1)^2}{8}\\ k_{IBL}=\frac{\alpha^2}{2}$$

这里的 α 取决于我们怎么从粗糙度转换。

为了更好的模拟，我们可以同时考虑两个视角，视线方向(几何遮蔽)和光线方向(几何阴影) 几何遮蔽类似“看不到”，几何阴影类似“照不到”。 使用 **Smith** 函数将其放在一起:

$$G(n,v,l,k)=G_{sub}(n,v,k)G_{sub}(n,l,k)$$

```OpenGL
float GeometrySchlickGGX(float NdotV, float k)
{
    float nom = NdotV;
    float denom =  NdotV * (1 - k) + k;
    return nom / denom;
}

float GeometeySmith(vec3 N, vec3 V, vec3 L, float k)
{
    float Gsub = GeometrySchlickGGX(saturate(dot(N,V)), k);
    float Gsub2 = GeometrySchlickGGX(saturate(dot(N,L)), k);
    return Gsub * Gsub2;
}
```

### **Kulla-Conty Approximation**

由于几何遮蔽造成能量损失，使得粗糙度较大时物体表面较暗，但实际上光线在表面经过多次弹射后不会被遮挡，可以反射出去(不考虑热能损失)。BRDF 只是考虑一次反射罢了。

所以使用经验模型去补全损失的能量，首先要知道有多少能量损失了。

$$E(\mu_o)=\int_0^{2\pi}\int_0^1f_r(\mu_o,\mu_i,\phi)u_i\,\mathrm{d}\mu_i\,\mathrm{d}\phi\\ u=\sin\theta$$

- Key idea
  - 损失的能量就是 1 - E(\mu_o)，不过E(\mu_o)是和观察方向相关的。 我要做的就是补上这部分能量，能量加起来就是1了啊。
  - E(\mu_0)是和观察方向相关的。
  - 要补一种多次散射的BRDF结果，也就是用一个模型去模拟多次反射的计算结果，而且因为 BRDF 具有对称性，有1 - E(\mu_o)那么应该也有一项1 - E(\mu_i)，因为我们的这个经验式子是模拟一个多级的基于 BRDF 的反射，然后补上一个归一化的参数c，得到这样的结果： c(1-E(\mu_i))(1 - E(\mu_o)) 这么设计只是为了简单……
  -  $$c=\frac{1}{\pi(1-E_{avg})}\\ E_{avg}=2\int_0^1E(\mu)\mu\,\mathrm{d}\mu\\ f_{ms}(\mu_o,\mu_i)=\frac{(1-E(\mu_i))(1 - E(\mu_o))}{\pi(1-E_{avg})}$$
  - 
- 但E_{avg}还是不知道的，这个可以预计算。
  - Precompute / tabulate
  -  $$E_{avg}(\mu_o)=2\int_0^1E(\mu_i)\mu_i\,\mathrm{d}\mu_i\\$$
  - $$E_{avg}$$和$$\mu_o$$, 以及 BRDF(或者说roughness) 相关 这个预计算的结果会根据 brdf 的不同而改变。

![Kulla-Conty](/imgs/LearnPBR/Kulla-Conty.png)

- 如果物体有颜色，就会有能量损失，这样积分一开始就不会是1. 我们先计算 没有颜色损失的 正确结果，最后计算时再考虑由于颜色引起的损失。
- Define the average Fresnel 不管入射角多大，每次反射平均反射掉多少能量

$$F_{avg}=\frac{\int_0^1F(\mu)\mu\,\mathrm{d}\mu}{\int_0^1\mu\,\mathrm{d}\mu}=2\int_0^1F(\mu)\mu\,\mathrm{d}\mu$$

- $$E_{avg}$$ 表示有多少能量我们可以看到，这些能量不会发生多次的反射。 NOT participate in further bounces
- 所以最后的 能量/颜色 
  - 能够直接看到的 $$F_{avg}E_{avg}$$ 
  - 光反射一次被看到：$$F_{avg}(1-E_{avg})\cdot F_{avg}E_{avg}$$  $$F_{avg}(1-E_{avg})$$是反射后(F)未能从物体表面反射出去的能量(1-E)， $$F_{avg}(1-E_{avg})\underline{F_{avg}}$$ 未能出去的能量发生第二次 $$F_{avg}(1-E_{avg})\cdot F_{avg}\underline{E_{avg}}$$ 发生反射后有多少能量被看到
  - 反射k次：F_{avg}^k(1-E_{avg})^k\cdot F_{avg}E_{avg}
  - 累加得到 color term：
  -  $$\frac{F_{avg}E_{avg}}{1-F_{avg}(1-E_{avg})}$$
  - 最后将 color term directly multiplied on the uncolored **additional BRDF**

### **Cook-Torrance 反射方程**

$$L_0(p,w_0)=\int_{\Omega}(k_d\frac{c}{\pi} + k_s\frac{FDG}{4(w_0\cdot n)(w_i\cdot n)})L_i(p,w_i)n\cdot w_i\,\mathrm{d}w_i$$

#### 直接光 + 附加光

![output2](/imgs/LearnPBR/output2.PNG)

## **IBL**

### **Diffuse Irradiance**

Imaged base lighting, IBL 是一类光照技术的集合，若光源不是可分解的直接光源，比如可以用辐射度量学计算的的点光源方向光等等，**而是将周围环境整体视为一个大光源**。IBL( 取自现实世界或者在3D场景生成) 环境立方体贴图(cubemap)，我们可以将立方体贴图的每个像素视为光源，在渲染方程中直接使用，这样可以有效的捕获环境的全局光照和氛围，使物体更好的融入环境。 由于基于图像的光照算法会捕捉部分甚至全部的环境光照，通常认为它是一种更精确的环境光照输入格式，甚至也可以说是一种全局光照的粗略近似。基于此特性，IBL 对 PBR 很有意义，因为当我们将环境光纳入计算之后，物体在物理方面看起来会更加准确。

$$L_0(p,w_0)=\int_{\Omega}(k_d\frac{c}{\pi} + k_s\frac{FDG}{4(w_0\cdot n)(w_i\cdot n)})L_i(p,w_i)n\cdot w_i\,\mathrm{d}w_i$$

对于反射方程的求解主要是在半球上对所有入射光方向 w_i 的积分。 直接光照的话，我们事先知道对积分有贡献的、若干精准的光线方向，但是来自环境的**每个**方向w_i都有可能具有一定的 Radiance，这就很麻烦了。 我们需要：

- 对给定任何方向w_i，能获取到该方向的场景 Radiance。
- 积分需要快，因为是实时渲染。

第一个思路就是用 环境立方体贴图，每个纹素都视为一个光源，使用一个w_i采样即可。

为了更高效的解决积分，我们需要对其中大部分结果做预处理，再来看反射方程:

$$L_0(p,w_0)=\int_{\Omega}(k_d\frac{c}{\pi} + k_s\frac{FDG}{4(w_0\cdot n)(w_i\cdot n)})L_i(p,w_i)n\cdot w_i\,\mathrm{d}w_i\\ Because\;diffuse\;k_d\;and\;specular\;are\;independent\;for\;each\;other.\\ We\;can\;break\;up.\\ L_0(p,w_0)=\int_{\Omega}k_d\frac{c}{\pi}L_i(p,w_i)n\cdot w_i\,\mathrm{d}w_i + \int_{\Omega}k_s\frac{FDG}{4(w_0\cdot n)(w_i\cdot n)}L_i(p,w_i)n\cdot w_i\,\mathrm{d}w_i$$

先来研究 diffuse，将常数提出，能得到只依赖于w_i的积分，我们就可以计算或预计算一个新的立方体贴图，它在每个采样方向——也就是纹素——中存储漫反射积分的结果，这些结果是通过卷积计算出来的。

$$L_0(p,w_0)=k_d\frac{c}{\pi}\int_{\Omega}L_i(p,w_i)n\cdot w_i\,\mathrm{d}w_i$$

卷积的特性是，对数据集中的一个条目做一些计算时，要考虑到数据集中的所有其他条目。这里的数据集就是场景的辐射度或环境贴图。因此，要对立方体贴图中的每个采样方向做计算，我们都会考虑半球 \Omega 上的所有其他采样方向。

为了对环境贴图进行卷积，我们通过对半球 \Omega上的大量方向进行离散采样并对其辐射度取平均值，来计算每个输出采样方向 w_0的积分。用来采样方向 w_i 的半球，要面向卷积的输出采样方向 w_0 。

![ibl1](/imgs/LearnPBR/ibl1.png)

是不是看不懂，看不懂就对了，因为应该是这样的！

![ibl2](/imgs/LearnPBR/ibl2.png)

该预计算的立方体贴图在每个采样方向 w_0(n)上存储结果，也就是场景中所有能够击中表面朝向为w_0(n)的间接漫反射光的预计算和。

辐射方程也依赖了位置 p ，不过这里我们假设它位于辐照度图的中心。这就意味着所有漫反射间接光只能来自同一个环境贴图，这样可能会破坏现实感（特别是在室内）。渲染引擎通过在场景中放置多个反射探针来解决此问题，每个反射探针单独预计算其周围环境的辐照度图。这样，位置 p 处的辐照度（以及辐射度）是取离其最近的反射探针之间的辐照度（辐射度）内插值

关于在半球的积分，可以将立体角 soiled angle 展开

$$L_0(p,w_0)= k_d\frac{c}{\pi} \int_{\phi=0}^{2\pi}\int_{\theta=0}^{\frac{1}{2}\pi} L_i(p,\phi_i,\theta_i) \cos\theta\sin\theta\,\mathrm{d}\theta\,\mathrm{d}\phi\\  Here\;\cos\theta\sin\theta,\;\sin\theta\;for\;soiled\;angle,\;\cos\theta\;for\;\overrightarrow{up}:w_i\cdot n\\  L_0(p,w_0)=k_d\frac{c}{\pi} \frac{1}{n_1n_2}\sum\limits^{n_1}_{\phi=0}\sum\limits^{n_2}_{\theta=0} L_i(p,\phi_i,\theta_i) \cos\theta\sin\theta\,\mathrm{d}\theta\,\mathrm{d}\phi$$

### **实现**

Roughness in [0,1]->[0,5]

做五级的skybox，存在cubemap tex内

#### **参考**

[漫反射辐照 - LearnOpenGL CN (learnopengl-cn.github.io)](https://learnopengl-cn.github.io/07 PBR/03 IBL/01 Diffuse irradiance/) [codinglabs.net/article_physically_based_rendering.aspx](http://www.codinglabs.net/article_physically_based_rendering.aspx)

### **Specular IBL**

#### **The Split Sum: 1st Stage**

现在来看镜面反射部分，反射方程为：

$$L_0(p,w_0)=\int\limits_\Omega(k_d\frac{c}{\pi}+k_s\frac{DFG}{4(w_0\cdot n)(w_i\cdot n)})L_i(p,w_i)n\cdot w_i\,\mathrm{d}w_i$$

这下坏了，因为 ks 是受入射光还有视角影响的。如果进行实时计算，视线和光线的组合数极其庞大，这样的开销是很昂贵的。 Epic Games 提出了一个解决方案，他们预计算镜面部分的卷积，为实时计算作了一些妥协，这种方案被称为分割求和近似法（**split sum approximation**）。 分割求和近似将方程的镜面部分分割成两个独立的部分，我们可以单独求卷积，然后在 PBR 着色器中求和，以用于间接镜面反射部分 IBL。分割求和近似法类似于我们之前求辐照图预卷积的方法，需要 HDR 环境贴图作为其卷积输入。为了理解，我们回顾一下反射方程，但这次只关注镜面反射部分：

$$L_0(p,w_0)=\int\limits_\Omega(k_s\frac{DFG}{4(w_0\cdot n)(w_i\cdot n)})L_i(p,w_i)n\cdot w_i\,\mathrm{d}w_i\\ =\int\limits_\Omega f_r(p,w_i,w_0)L_i(p,w_i)n\cdot w_i\,\mathrm{d}w_i$$

我们依然想计算出一个类似镜面 IBL 贴图的东西，然后使用pixel的法线采样，但是问题在于，辐照度图只依赖于 w_i，但是这次积分还依赖于 BRDF

$$f_r(p,w_i,w_0)=\frac{DFG}{4(w_0\cdot n)(w_i\cdot n)}$$

BRDF内还有w_0，更不可能要用入射和出射光的组合了，那计算肯定爆掉了。

$$\int_{\Omega}f(x)g(x)\,\mathrm{d}x\approx\frac{\int_{\Omega_G}f(x)\,\mathrm{d}x}{\int_{\Omega_G}\,\mathrm{d}x}\cdot\int_{\Omega}g(x)\,\mathrm{d}x$$

所以 E宝先将其分为两个部分求解，再将两个部分组合计算得到预计算结果。

$$L_0(p,w_0)=\int\limits_\Omega L_i(p,w_i)\,\mathrm{d}w_i*\int\limits_\Omega f_r(p,w_i,w_0)n\cdot w_i\,\mathrm{d}w_i$$

卷积的第一部分被称为预滤波环境贴图，它类似于辐照度图，是预先计算的环境卷积贴图，但这次考虑了粗糙度。因为随着粗糙度的增加，参与环境贴图卷积的采样向量会更分散，导致反射更模糊，所以对于卷积的每个粗糙度级别，我们将按顺序把模糊后的结果存储在预滤波贴图的 mipmap 中。例如，预过滤的环境贴图在其 5 个 mipmap 级别中存储 5 个不同粗糙度值的预卷积结果，如下图所示：

![IBLCubemap](/imgs/LearnPBR/IBLCubemap.png)

为什么是不同粗糙度的图呢？这是因为对于不同的粗糙度，我们镜面反射是不同的，当表面越粗糙镜面反射越松散，我们观测的光照结果会越分散在更广的范围，而越光滑，反射范围越小越集中，视觉上也就是越清晰的。

![iblAlpha](/imgs/LearnPBR/iblAlpha.png)

从上图我们可以看到我们对r方向采样，实际上是对cubemap的一个橙色的 fliter 进行采样的，当越粗糙时这个 filter 越大。

为什么两部分分开了还要前面项会受到 BRDF 影响，因为我们的采样方向其实就是会受到 BRDF 的影响的。

![boban](/imgs/LearnPBR/boban.jpg)

我们对cubemap的采样范围或者说采样的样本实际就是图中所示的波瓣，而根据 brdf 定义可知：

$$R=reflection(w_o,n)，w_o就是\overrightarrow{view}。$$

**假设不同方向入射，波瓣变化不大**，我们可以得到:

$$f(w_o,w_i(n),n)\approx f(R,w_i(R),R)，也就是 N=V=R。$$ f(w_o,w_i(n),n)\approx f(R,w_i(R),R)，也就是 N=V=R。

![poban](/imgs/LearnPBR/poban.jpg)

我们使用 Cook-Torrance BRDF 的法线分布函数(NDF)生成采样向量及其散射强度，该函数将法线和视角方向作为输入。由于我们在卷积环境贴图时事先不知道视角方向，因此 Epic Games 假设视角方向——也就是镜面反射方向——总是等于输出采样方向ωo，以作进一步近似。翻译成代码如下：

vec3 N = normalize(w_o);

vec3 R = N;

vec3 V = R;

但是这样的假设会导致在掠射角处失去各向异性，因为菲涅尔现象会导致不同w_o的反射方向或者反射现象是不一样的，所以基于波瓣不变的假设所做的结果必然会有缺失。

Moving Frostbite to Physically Based Rendering 3.0-4.9.2 Light probe filtering

To simplify this evaluation, we can pre-integrate the integral by making some approximations. Pre-integrating this equation for every v and Θ would require a huge memory footprint. Thus, a first approximation is to remove the view dependency. This leads to a coarse approximation of the BRDF but it is an acceptable trade-off: the shape of a BRDF based on the micro-facets framework and/or half-angle parametrization is strongly dependent on the view angle as shown on Figure 54. At normal incident direction, the shape of a BRDF is isotropic. At grazing angles the shape of a BRDF is anisotropic. Removing the view dependency for pre-integrating Equation 46 would make the assumption that the BRDF shape is isotropic at all view angles. This leads to key visual differences, preventing stretched reflections. This approximation can be quite noticeable on flat surfaces as shown on Figure 55 but less on curvy surfaces38

![ueReference](/imgs/LearnPBR/ueReference.png)

#### **The Split Sum: 2nd Stage**

$$Lo(p,w_o)\approx\frac{\int_{\Omega_{fr}}L_i(p,w_i)\,\mathrm{d}w_i}{\int_{\Omega_{fr}}\,\mathrm{d}w_i}\cdot\underline{\int_{\Omega^+}f_r(p,w_i,w_o)\cos\theta_i\,\mathrm{d}w_i}$$

这部分计算和 F_0,\alpha,\theta相关，但是3D贴图太大了！！！

菲涅项其实比较好拆，我们可以对这部分做一些处理

$$R(\theta)=R_0+(1-R_0)(1-\cos\theta)^5\\ R_0-R_0(1-\cos\theta)^5+(1-\cos\theta)^5\\ \int_{\Omega^+}f_r(p,w_i,w_o)\cos\theta_i\,\mathrm{d}w_i\approx\\ R_0\int_{\Omega^+}\frac{f_r}{F}(1-(1-\cos\theta)^5)\cos\theta_i\,\mathrm{d}w_i+\int_{\Omega^+}\frac{f_r}{F}(1-\cos\theta)^5\cos\theta_i\,\mathrm{d}w_i$$

![uecode1](/imgs/LearnPBR/uecode1.png)

### **The Split Sum: 1st Stage Sample**

在上一节教程中，我们使用球面坐标生成均匀分布在半球 $$\Omega$$ 上的采样向量，以对环境贴图进行卷积。虽然这个方法非常适用于辐照度，但对于镜面反射效果较差。镜面反射依赖于表面的粗糙度，反射光线可能比较松散，也可能比较紧密，但是一定会围绕着反射向量r，除非表面极度粗糙：

![sample](/imgs/LearnPBR/sample.png)

所有可能出射的反射光构成的形状称为镜面波瓣。随着粗糙度的增加，镜面波瓣的大小增加；随着入射光方向不同，形状会发生变化。因此，镜面波瓣的形状高度依赖于材质。 在微表面模型里给定入射光方向，则镜面波瓣指向微平面的半向量的反射方向。考虑到大多数光线最终会反射到一个基于半向量的镜面波瓣内，采样时以类似的方式选取采样向量是有意义的，因为大部分其余的向量都被浪费掉了，这个过程称为重要性采样。

F = FresnelSchlickFunction(F0, max(0.0, dot(N, H)), roughness);

float3 spe_ibl = SAMPLE_TEXTURE2D_LOD(_MySplit1st, sampler_MySplit1st, float2(uv.xy), _Roughness * 4.0);

col += spe_ibl * (F * brdf.x + brdf.y)+ ev_diffuse*albedo;

![uecode2](/imgs/LearnPBR/uecode2.png)

### **蒙特卡洛(Monte Carlo)积分和重要性采样(Importance Sampling)**

[蒙特卡洛积分 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/146144853)

有了蒙特卡洛，那么如何对半球面上的进行采样。

### **低差异序列**

Hammersley 序列

```OpenGL
float RadicalInverse_VdC(uint bits)
{
    bits = (bits << 16u) | (bits >> 16u);
    bits = ((bits & 0x55555555u) << 1u) | ((bits & 0xAAAAAAAAu) >> 1u);
    bits = ((bits & 0x33333333u) << 2u) | ((bits & 0xCCCCCCCCu) >> 2u);
    bits = ((bits & 0x0F0F0F0Fu) << 4u) | ((bits & 0xF0F0F0F0u) >> 4u);
    bits = ((bits & 0x00FF00FFu) << 8u) | ((bits & 0xFF00FF00u) >> 8u);
    return float(bits) * 2.3283064365386963e-10; // / 0x100000000
}
// ----------------------------------------------------------------------------
float2 Hammersley(uint i, uint N)
{
    return float2(float(i)/float(N), RadicalInverse_VdC(i));
}
```

[看懂蒙特卡洛积分(三) 低差异采样序列](https://zhuanlan.zhihu.com/p/343666731) [低差异序列（一）- 常见序列的定义及性质 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/20197323) [低差异序列（二）- 高效实现以及应用 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/20374706)

#### **GGX 重要性采样**

```OpenGL
float3 ImportanceSampleGGX(float2 Xi, float3 N, float roughness)
{
    // use roughness for better view
    float alpha2 = roughness * roughness;

    float phi = 2.0 * PI * Xi.x;
    float cosTheta = sqrt( (1.0 - Xi.y) / (1.0 + ( alpha2 * alpha2 - 1.0) * Xi.y));
    float sinTheta = sqrt(1 - cosTheta * cosTheta);

    float3 H;
    H.x = cos(phi) * sinTheta;
    H.y = sin(phi) * sinTheta;
    H.z = cosTheta;

    float3 up = abs(N.z) < 0.999 ? float3(0.0, 0.0, 1.0) : float3(1.0, 0.0, 0.0);
    float3 tangent = normalize(cross(up,N));
    float3 biTangent = normalize(cross(N, tangent));

    float3 sampleVec = tangent * H.x + biTangent * H.y + N * H.z;
    return normalize(sampleVec);
}
```

### The Split Sum

```OpenGL
// 1st
void PrefilteredColor(float3 dir, uint3 id)
{
    float3 N = normalize(dir);
    float3 R = N;
    float3 V = R;

    const uint SAMPLE_COUNT = 1024;
    float totalWeight[5] = {0.0, 0.0, 0.0, 0.0, 0.0};
    float3 preColor[5] = { float3(0.0,0.0,0.0), float3(0.0,0.0,0.0),float3(0.0,0.0,0.0),float3(0.0,0.0,0.0),float3(0.0,0.0,0.0)};
    for(uint i = 0; i < SAMPLE_COUNT; ++i)
    {
        float2 Xi = Hammersley(i, SAMPLE_COUNT);
        float3 H[5];
        float3 L[5];
        float NdotL[5];
        for(uint a = 0; a < 5; a++)
        {
            H[a] = ImportanceSampleGGX(Xi, N, (float)a*2.0/10.0);
            L[a] = normalize(2.0 * dot(V, H[a]) * H[a] - V);
            NdotL[a] = max(0.0,dot(N,L[a]));
            if(NdotL[a] > 0.0)
            {
                float D = DistributeGGX(N, H[a], (float)a/5.0);
                float NdotH = max(0.0, dot(N, H[a]));
                float HdotV = max(0.0, dot(H[a], V));
                float pdf = D * NdotH / (4.0 * HdotV) + 0.0001;

                float res = 512.0;
                float saTexel = 4.0 * PI / (6.0 * res * res);
                float saSample = 1.0 / (float(SAMPLE_COUNT) * pdf + 0.0001);
                float roughness = (float)a / 5.0;
                float mipLevel = roughness == 0.0 ? 0.0 : 0.5 * log2(saSample / saTexel); 
                
                preColor[a] += _SkyBoxTex.SampleLevel(LinearClampSampler, L[a], mipLevel).xyz * NdotL[a];
                totalWeight[a] += NdotL[a];
            }
        }         
    }
    int powLog = 1;
    for(uint b = 0; b < 5; b++)
    {    
        preColor[b] = preColor[b] / totalWeight[b];
        _SplitSum1stMip[b][id.xy/powLog] = float4(preColor[b],1);
        powLog *= 2;
    }
}

// 2nd
float2 IntegrateBRDF(float NdotV, float roughness)
{
    float3 V;
    V.x = sqrt(1 - NdotV * NdotV);
    V.y = 0.0;
    V.z = NdotV;

    float A = 0.0;
    float B = 0.0;

    float3 N = float3(0.0, 0.0, 1.0);

    int sampleCount = 1024;
    for(int i = 0; i < sampleCount; ++i)
    {
        float2 Xi = Hammersley(i, sampleCount);
        float3 H = ImportanceSampleGGX(Xi, N, roughness);       // D. NDF
        float3 L = normalize(2.0 * dot(V,H) * H - V);

        float NdotL = max(L.z, 0.0);
        float NdotH = max(H.z, 0.0);
        float VdotH = max(0.0, dot(V, H));

        if(NdotL > 0.0)
        {
            float G = GeometrySmith(N, V, L, roughness * roughness / 2.0);      // G
            float G_Vis = (G * VdotH) / (NdotH * NdotV);    
            float Fc = pow(1 - VdotH, 5.0);
            
            A += (1.0 - Fc) * G_Vis;
            B += Fc * G_Vis;
        }
    }
    A /= float(sampleCount);
    B /= float(sampleCount);
    return float2(A, B);
}
```

### **反射探针**

在unity中添加 probe 然后 baked

使用 sampleSH 对 probe 进行采样

### Environment Diffuse + IBL Specular

![output3](/imgs/LearnPBR/output3.PNG)

中间是粗糙度越来越小 下面是金属度越来越大

### **IBL 参考**

[Image Based Lighting | Chetan Jags (wordpress.com)](https://chetanjags.wordpress.com/2015/08/26/image-based-lighting/) [course_notes_moving_frostbite_to_pbr_v32.pdf (wordpress.com)](https://seblagarde.files.wordpress.com/2015/07/course_notes_moving_frostbite_to_pbr_v32.pdf):4.9章节 [镜面IBL - LearnOpenGL CN (learnopengl-cn.github.io)](https://learnopengl-cn.github.io/07 PBR/03 IBL/02 Specular IBL/) [深入理解 PBR/基于图像照明 (IBL) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/66518450) [Lecture5 Real-time Environment Mapping哔哩哔哩bilibili](https://www.bilibili.com/video/BV1YK4y1T7yY?p=5&vd_source=1fa1b82383f6efb8a2632316da9afad0): Split Sum

## **杂项**

- ComputeShader 使用 cubetex：[[Compute shader\] Use TextureCube(Resolved)](https://forum.unity.com/threads/compute-shader-use-texturecube-resolved.628891/)
- TextureCube<float4 cubemap; SamplerState _LinearClamp; float3 dir; cubemap.SampleLevel(_LinearClamp, dir, 0);
- 如何向Tex指定mipmap层级写入，在 setrendertarget 指定
- compute Shader sample
  - `float4 c = tex[id];`
  - mipmap: `float4 c = tex.mips[0][id]` or `tex.Load(uint3(id,0))`
  - SampleLevel: 
  - SampleState Sampler1 {    Filter = MIN_LINEAR_MAG_MIP_POINT; }; float4 t = tex.SampleLevel(Sample1, uv, 0);
- Sample filter [8.5 纹理采样 (enjoyphysics.cn)](https://enjoyphysics.cn/Article1554)

```OpenGL
// 在倍增、缩减、多级渐进纹理上使用线性过滤。
SamplerState mySampler0 
{
    Filter = MIN_MAG_MIP_LINEAR; 
}; 

// 在缩减上使用线性过滤，倍增和多级渐进纹理上使用点过滤。
SamplerState mySampler1 
{ 
    Filter = MIN_LINEAR_MAG_MIP_POINT; 
}; 

// 在缩减上使用点过滤，倍增上使用线性过滤，多级渐进纹理上使用点过滤。
SamplerState mySampler2 
{ 
    Filter = MIN_POINT_MAG_LINEAR_MIP_POINT; 
}; 

// 在倍增、缩减、多级渐进纹理上使用各向异性过滤。
SamplerState mySampler3 
{ 
    Filter = ANISOTROPIC; 
    MaxAnisotropy = 4;
};
```

### **Unity And Mipmap**

[【渲染】用计算着色器生成Mipmap - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/419644293)

### **Computer 指定 Mipmap 写入**

```
computeShader.SetTexture(kernal, “name”, tex, mipmapLevel);
```

### **不同光源的衰减**

[投光物 - LearnOpenGL CN (learnopengl-cn.github.io)](https://learnopengl-cn.github.io/02 Lighting/05 Light casters/) [Point and Spot Lights (catlikecoding.com)](https://catlikecoding.com/unity/tutorials/custom-srp/point-and-spot-lights/)

#### **Point Light**

- $(\,max(0, 1-(\frac{d^2}{r^2})^2)\,)^2$，r 是 point light 的范围
- $\frac{1.0}{K_c+K_l*d+K_q*d^2}$

### **ShaderGUI**

通过编写 ShaderGUI 在 Material 面板控制 shader properties

或 en/dis able keyword

```OpenGL
public class MyShaderGUI : ShaderGUI
{
    public override void OnGUI(MaterialEditor materialEditor, MaterialProperty[] properties)
    {
        base.OnGUI(materialEditor, properties);
        Material material = materialEditor.target as Material;
    }
}
```

在 Shader 结尾加上

CustomEditor "MyLearnPBRGUI";

## **URP 光**

[Light 组件参考 | Universal RP | 12.1.1 (unity3d.com)](https://docs.unity3d.com/cn/Packages/com.unity.render-pipelines.universal@12.1/manual/light-component.html) [光照模式 - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/current/Manual/LightModes.html) [光源模式：Baked - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/current/Manual/LightMode-Baked.html)

### **Lighting**

- Scene/Mixed Lighting/Lighting Mode
  - **Baked Indirect**: [Lighting Mode：Baked Indirect - Unity 手册](https://docs.unity.cn/cn/2022.3/Manual/LightMode-Mixed-BakedIndirect.html) 混合光源的行为类似于[实时光源](https://docs.unity.cn/cn/2022.3/Manual/LightMode-Realtime.html)，但有额外的好处是会**将间接光照烘焙到光照贴图中**。**混合(Mixed)光源照亮的游戏对象会投射实时阴影**，最大距离是在项目中定义的[阴影距离 (Shadow Distance)](https://docs.unity.cn/cn/2022.3/Manual/shadow-distance.html)。
  - **Shadow Mask**: [Lighting Mode：Shadowmask - Unity 手册](https://docs.unity.cn/cn/2022.3/Manual/LightMode-Mixed-Shadowmask.html) 与烘焙间接照明模式类似，阴影遮罩照明模式将**实时直接照明**与**[烘焙间接照明](https://docs.unity.cn/cn/2022.3/Manual/LightMode-Baked.html)****(Baked Indirect Lighting Mode)**相结合。但是，“阴影蒙版”照明模式与烘焙间接照明模式的不同之处在于它渲染阴影的方式。阴影蒙版光照模式使 Unity 可以在运行时组合烘焙阴影和实时阴影，并在远处渲染阴影。它通过使用称为阴影蒙版的附加光照贴图纹理，并在[光照探针](https://docs.unity.cn/cn/2022.3/Manual/LightProbes.html)中存储其他信息来实现此目的。Unity 为烘焙阴影生成阴影蒙版和 Light Probe 遮挡数据。
  - **Subtractive**: [Lighting Mode：Subtractive - Unity 手册](https://docs.unity.cn/cn/2022.3/Manual/LightMode-Mixed-Subtractive.html) 在 Subtractive 光照模式下，场景中的所有混合光源都提供烘焙直接光照和间接光照。Unity 将静态游戏对象投射的阴影烘焙到光照贴图中。除了烘焙阴影外，一种方向光（称为主方向光）还为动态游戏对象提供实时阴影。 因为阴影被烘焙到光照贴图中，所以 Unity 在运行时缺少将烘焙阴影和实时阴影准确地结合在一起所需的信息。但是，Unity 提供了 **Realtime Shadow Color** 属性来减少光照贴图的影响，从而在烘焙阴影和实时阴影之间创建正确的混合视觉效果。还可以调整颜色来实现某种艺术风格。 Subtractive 光照模式在低端硬件上非常有用，因为低端硬件需要注重性能，并且只需要一个实时阴影投射光源。这种光照模式不会提供特别逼真的光照效果，而是更适合风格化美学，例如卡通风格。

## **Isotropic / Anisotropic BRDFs**

$$f_r(\theta_i,\phi_i;\theta_r,\phi_r)\ne f_r(\theta_i,\phi_i,\phi_r-\phi_i)\\ i\;is\;input,\;r\;is\;reflection\;dir.$$

### **NDF**

**形状不变性**

- 是一个合格的法线分布函数需要具备的重要性质。具有形状不变性（shape-invariant）的法线分布函数，可以用于推导该函数的归一化的各向异性版本，并且可以很方便地推导出对应的遮蔽阴影项G。
- 若一个各向同性的NDF可以改写成以下形式，则这个NDF具有形状不变性（shape-invariant）：
- $$D(m)=\frac{1}{\alpha_2(n\cdot m)^4}g(\frac{\sqrt{1-(n\cdot m)^2}}{\alpha(n\cdot m)})$$

- 其中g（）代表一个表示了NDF形状的一维函数。

#### **Anisotropic Beckmann Distribution**

$$D_{Baniso}(m)=\frac{1}{\pi\alpha_x\alpha_y(n\cdot m)^4}exp(-\frac{\frac{(t\cdot m)^2}{\alpha_x^2}+\frac{(b\cdot m)^2}{\alpha_y^2}}{(n\cdot m)^2})$$

```OpenGL
// Anisotropic Beckmann
float D_Beckmann_aniso( float ax, float ay, float NoH, float3 H, float3 X, float3 Y )
{
    float XoH = dot( X, H );
    float YoH = dot( Y, H );
    float d = - (XoH*XoH / (ax*ax) + YoH*YoH / (ay*ay)) / NoHNoH;
*    return exp(d) / ( PI * ax*ay * NoH * NoH * NoH * NoH );
}
```

#### **Trowbridge-Reitz GGX Anisotropic**

$$D_{GGXaniso}(m)=\frac{1}{\pi\alpha_x\alpha_y}\frac{1}{(\frac{(x\cdot m)^2}{\alpha_x^2}+\frac{(y\cdot m)^2}{\alpha_y^2}+(n\cdot m))^2}$$

```OpenGL
// Anisotropic GGX
// [Burley 2012, "Physically-Based Shading at Disney"]
float D_GGXaniso( float ax, float ay, float NoH, float3 H, float3 X, float3 Y )
{
    float XoH = dot( X, H );
    float YoH = dot( Y, H );
    float d = XoH*XoH / (ax*ax) + YoH*YoH / (ay*ay) + NoHNoH;
*    return 1 / ( PI * ax*ay * d*d );
}
```

- 其中，X为tangent，t切线方向，Y为binormal，b，副法线方向
- 需要注意的是，将法线贴图与各向异性BRDF组合时，重要的是要确保法线贴图扰动（perturbs）切线和副切线矢量以及法线

[2010-anisobrdf.pdf (cuni.cz)](https://cgg.mff.cuni.cz/~jaroslav/papers/2010-anisobrdf/2010-anisobrdf.pdf) [Slope: GGX Anisotropic (shadertoy.com)](https://www.shadertoy.com/view/3tyXRt)

### **Geometry Function**

$$G_1(m,v)=\frac{clamp(0,1,m\cdot v)}{1+\Lambda(v)}$$

#### **Beckmann**

$$\Lambda(v)=\frac{erf(a)-1}{2}+\frac{1}{2a\sqrt{\pi}}exp(-a^2)\\ a=\frac{1}{\alpha\tan\theta_o}$$

#### **GGX**

$$\Lambda(v)=\frac{-1+\sqrt{1+\frac{1}{a^2}}}{2}\\ a=\frac{1}{\alpha\tan\theta_o}$$

### **Anisotropic Geometry Function**

#### **Smith**

假设我们拉伸x轴，将各向同性的法线分布转变为各向异性的。

假设各向异性的粗糙度参数有a_x,a_y，视线v(x_o,y_o,z_o)，通过拉伸x轴\frac{a_x}{a_y}

$$a_x'=a_x\frac{a_y}{a_x}=a_y\\ a_y'=a_y$$

也就是粗糙度\alpha=\alpha_y

$$v'=(\frac{\alpha_x}{\alpha_y}x_o,y_o,z_o)=(\frac{\alpha_x}{\alpha_y}\cos\phi_o\sin\theta_o,\sin\phi_o\sin\theta_o,\cos\theta_o)\\ \frac{1}{\tan\theta'_o}=\frac{z_o}{\sqrt{\frac{\alpha_x^2}{\alpha_y^2}x_o^2+y_o^2}}=\frac{1}{\sqrt{\frac{\alpha_x^2}{\alpha_y^2}\cos^2\phi_o+\sin^2\phi_o}\cdot\tan\theta_o}\\ Because:a=\frac{1}{\alpha\tan\theta_o}\\ a'=\frac{1}{a_y\tan\theta_o}=\frac{1}{a_y\sqrt{\frac{\alpha_x^2}{\alpha_y^2}\cos^2\phi_o+\sin^2\phi_o}\cdot\tan\theta_o}\\ =\frac{1}{\sqrt{\alpha_x^2\cos^2\phi_o+\alpha_y^2\sin^2\phi_o}\cdot\tan\theta_o}$$

[PBR 五 几何遮蔽函数遮蔽因子函数wuhaocat的博客-CSDN博客](https://blog.csdn.net/haozi2008/article/details/112284028)

[PBR-White-Paper/content/part 5/README.md at master · QianMo/PBR-White-Paper (github.com)](https://github.com/QianMo/PBR-White-Paper/blob/master/content/part 5/README.md)

### **Output**

<img src="/imgs/LearnPBR/output4-1.PNG" alt="output4-1" style="zoom:50%;" />

<img src="/imgs/LearnPBR/AnisoSphere2.png" alt="AnisoSphere2" style="zoom:50%;" />

### **Anisotropic IBL**

#### **Split Sum 2nd**

$$R(\theta)=R_0+(1-R_0)(1-\cos\theta)^5\\ R_0-R_0(1-\cos\theta)^5+(1-\cos\theta)^5\\ \int_{\Omega^+}f_r(p,w_i,w_o)\cos\theta_i\,\mathrm{d}w_i\approx\\ R_0\int_{\Omega^+}\frac{f_r}{F}(1-(1-\cos\theta)^5)\cos\theta_i\,\mathrm{d}w_i+\int_{\Omega^+}\frac{f_r}{F}(1-\cos\theta)^5\cos\theta_i\,\mathrm{d}w_i$$

roughness_x, roughness_y, theta 相关3D LUT了

## **[TODO]动态天空的环境光怎么计算**

提前烘焙，插值。

TOD

像大气散射，其实已经计算了 LUT 所以可以直接使用。



## Unity Specular Cube0

[Graphics/Packages/com.unity.shadergraph/Editor/Generation/Targets/BuiltIn/ShaderLibrary/Lighting.hlsl at 19518485b3edcf19f267f293f899d5d25e734a17 · Unity-Technologies/Graphics (github.com)](https://github.com/Unity-Technologies/Graphics/blob/19518485b3edcf19f267f293f899d5d25e734a17/Packages/com.unity.shadergraph/Editor/Generation/Targets/BuiltIn/ShaderLibrary/Lighting.hlsl#L620)

~~~hlsl
half mip = PerceptualRoughnessToMipmapLevel(perceptualRoughness);
    half4 encodedIrradiance = SAMPLE_TEXTURECUBE_LOD(unity_SpecCube0, samplerunity_SpecCube0, reflectVector, mip);
    half3 irradiance = DecodeHDREnvironment(encodedIrradiance, unity_SpecCube0_HDR);
    return irradiance * occlusion;
~~~

### PerceptualRoughnessToMipmapLevel

[Graphics/Packages/com.unity.render-pipelines.core/ShaderLibrary/ImageBasedLighting.hlsl at 19518485b3edcf19f267f293f899d5d25e734a17 · Unity-Technologies/Graphics (github.com)](https://github.com/Unity-Technologies/Graphics/blob/19518485b3edcf19f267f293f899d5d25e734a17/Packages/com.unity.render-pipelines.core/ShaderLibrary/ImageBasedLighting.hlsl#L27)

~~~
real PerceptualRoughnessToMipmapLevel(real perceptualRoughness, uint maxMipLevel)
{
    perceptualRoughness = perceptualRoughness * (1.7 - 0.7 * perceptualRoughness);

    return perceptualRoughness * maxMipLevel;
}
~~~



SAMPLE_TEXTURECUBE_LOD(unity_SpecCube0, samplerunity_SpecCube0, reflectVector, mip);

## **参考**

[由浅入深学习PBR的原理和实现 - 0向往0 - 博客园 (cnblogs.com)](https://www.cnblogs.com/timlly/p/10631718.html)   
[course_notes_moving_frostbite_to_pbr_v32.pdf (wordpress.com)](https://seblagarde.files.wordpress.com/2015/07/course_notes_moving_frostbite_to_pbr_v32.pdf)   
[【基于物理的渲染（PBR）白皮书】（一） 开篇：PBR核心知识体系总结与概览 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/53086060)   
[Physically Based Rendering: From Theory to Implementation (pbr-book.org)](https://www.pbr-book.org/3ed-2018/contents)  
Real-Time Rendering Fourth Edition 第九章   
[Lecture10 Real-Time Physically-based Materials (surface models)哔哩哔哩bilibili](https://www.bilibili.com/video/BV1YK4y1T7yY?p=10&vd_source=1fa1b82383f6efb8a2632316da9afad0)   
[Lecture11 Real-Time Physically-based Materials (surface models cont.)哔哩哔哩bilibili](https://www.bilibili.com/video/BV1YK4y1T7yY?p=11&vd_source=1fa1b82383f6efb8a2632316da9afad0)  
UE4: [2013SiggraphPresentationsNotes-26915738.pdf (unrealengine.com)](https://cdn2.unrealengine.com/Resources/files/2013SiggraphPresentationsNotes-26915738.pdf)   
[寒霜引擎的PBR实践3.0（一）材质篇 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/144611412)  
[寒霜引擎的PBR实践3.0（二）光照篇——光照强度与精确光源 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/158261389)   
[寒霜引擎的PBR实践3.0（三）光照篇——光度学灯与区域光 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/186541854)   
[[译\]Real Shading in Unreal Engine 4（UE4中的真实渲染)(1) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/121719442)  
[使用Compute Shader计算球谐全局光照 | ZZNEWCLEAR13](https://zznewclear13.github.io/posts/calculate-spherical-harmonics-using-compute-shader/)
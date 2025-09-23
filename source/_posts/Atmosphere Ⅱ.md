---
title: Atmosphere Ⅱ
math: true
tags: [Math]
index_img: 
banner_img: 
data: 2025--09--19 17:09:48
typora-root-url: ./..
---
搜寻找关键卡了，再战atmosphere，补充。  
<!--more-->

# Atmosphere Ⅱ

## Transmittance

当光在大气中从点 $p$ 传播至点 $q$ 时，由于空气分子和气溶胶粒子的作用，光被部分吸收并发生散射偏离初始的传播方向。因此，到达 $q$ 点的光只是从 $p$ 点出发的一部分，这一比例和波长相关，被称为 [transmittance](https://en.wikipedia.org/wiki/Transmittance)<font color=#A0A0A0>(透射率)</font>。接下来我们会描述如何计算，在预计算Texture中存储，并如何读取回来。

### Computation

对于在大气中共线的三点 $p,q,r$，按照这个顺序，$p,r$ 之间的透射率等于 $p,q$ 之间的透射率乘上 $q,r$ 之间的透射率。<font color=#ADADAD>这里的 $p,q,r$ 和下图无关。</font>  
特别的，$p,q$ 间的透射率为点 $p$ 和射线$[p,q)$ 与大气顶部或底部最近交点 $i$ 之间的透射率，除以 $q,i$ 间的透射率，或者，如果射线和地面相交则透射率就为 0：<font color=#ADADAD>(下图就是和大气顶部相交，如果q点下移，就是和大气底部相交，所谓和地面相交，就是qp两点连线和地面相交……)</font>  
![](/imgs/Atmosphere Ⅱ/Transmittance.svg)  
并且 $p,q$ 和 $q,p$ 之间的透射率是相同的，只和两点的位置有关和方向无关。因此，想要计算任意两点的透射率，只需要知道大气中一点 $p$ 和大气顶部边界上点 $i$ 之间的透射率。这个透射率只依赖两个参数，半径 $r=\lvert op\rvert$ 和“观测天顶角”的余弦 $\mu=op\cdot pi/\lvert op\rvert\lvert pi\rvert=\cos(\theta)$。为了计算该透射率，我们要知道 $\lvert pi\rvert$ 的长度，并且需要知道线段 $[p,i]$ 是否和地面相交。

#### Distance to the top atmosphere boundary

一点从 $p$ 出发沿着 $[p,i)$ 方向距离为 $d$，其坐标可以描述为：$[d\sqrt{1-\mu^2},r+d\mu]^T$，其模的平方 $d^2+2r\mu d+r^2$。因此根据点 $i$ 的定义能够得到 $\lvert pi\rvert^2+2r\mu\lvert pi\rvert+r^2=r_{top}^2$，由此可以推导出长度 $\lvert pi\rvert$：<font color=#9D9D9D>这里是以地球球心为原点，i 坐标的模就是大气top半径，二元一次方程，$\frac{-b\pm\sqrt{b^2-4ac}}{2a}$</font>
$$
\begin{aligned}
&d^2+2r\mu d+r^2-r_{top}^2=0\\
&a=1,b=2r\mu,c=r^2-r_{top}^2\\
&d=\frac{-b+\sqrt{b^2-4ac}}{2a}
=\frac{-2r\mu+\sqrt{4r^2\mu^2-4(r^2-r_{top}^2)}}{2}\\
&d=-r\mu+\sqrt{r_{top}^2+r^2(\mu^2-1)}
=-r\mu+\sqrt{r_{top}^2-r^2\sin\theta}
\end{aligned}
$$


~~~c++
Length DistanceToTopAtmosphereBoundary(IN(AtmosphereParameters) atmosphere,
    Length r, Number mu) {
  assert(r <= atmosphere.top_radius);
  assert(mu >= -1.0 && mu <= 1.0);
  Area discriminant = r * r * (mu * mu - 1.0) +
      atmosphere.top_radius * atmosphere.top_radius;
  return ClampDistance(-r * mu + SafeSqrt(discriminant));
}
~~~

我们也需要另外一种情况，到大气底部的距离，可以用相似的方式计算(代码表示 $[p,i)$ 相交地面)

~~~C++
Length DistanceToBottomAtmosphereBoundary(IN(AtmosphereParameters) atmosphere,
    Length r, Number mu) {
  assert(r >= atmosphere.bottom_radius);
  assert(mu >= -1.0 && mu <= 1.0);
  Area discriminant = r * r * (mu * mu - 1.0) +
      atmosphere.bottom_radius * atmosphere.bottom_radius;
  return ClampDistance(-r * mu - SafeSqrt(discriminant));
}
~~~

注：上述两种情况是对距离 d 求解，所以肯定是正值。  
负值是 $[i,p)$ 射线和大气顶部的交点。

#### Intersections with the ground

如果 $d^2+2r\mu d+r^2=r_{bottom}^2$ 有 $d\ge 0$ 的解，说明线段 $[p,i]$ 和地面相交。这需要式子 $r^2(\mu^2-1)+r_{bottom}^2$ 为正，我们能推导出：

~~~c++
bool RayIntersectsGround(IN(AtmosphereParameters) atmosphere,
    Length r, Number mu) {
  assert(r >= atmosphere.bottom_radius);
  assert(mu >= -1.0 && mu <= 1.0);
  return mu < 0.0 && r * r * (mu * mu - 1.0) +
      atmosphere.bottom_radius * atmosphere.bottom_radius >= 0.0 * m2;
}
~~~

<font color=#777>底部大气相交有解也不能说明和地面相交吧，这里应该用的是地球半径啊……</font>  
注：这里虽然没用地球半径但是有原因的，本篇不会考虑光线穿过大气底部和另一侧大气发生相交的情况，因为这样的“终点”在大气外，对地面的观测点没有意义，所以一旦和大气底部有交点就认为"和地面相交"，这样的实际意义是光被地面观测而结束，不再接着考虑。  
可以看到后面计算透射率时，对这种“地面相交”会进行额外处理，然后再来理解这部分。  
这并非物理意义的地面相交而是计算处理或者观测意义上的。

#### Transmittance to the tap atmosphere boundary

我们现在可以计算 $p,i$ 之间的 transmittance。根据 [Beer-Lambert law](https://en.wikipedia.org/wiki/Beer-Lambert_law)，这涉及到 $[p,i]$ 路径上  

* 空气分子密度的积分，
* 还有气溶胶密度积分，
* 吸光性分子(比如臭氧)密度的积分。

这三种粒子的积分形式是相同的。当线段 $[p,i]$ 不和地面相交时，可通过如下辅助函数进行(采样梯形法则[trapezoidal rule](https://en.wikipedia.org/wiki/Trapezoidal_rule))数值积分：

~~~c++
Number GetLayerDensity(IN(DensityProfileLayer) layer, Length altitude) {
	// 根据预设值和对大气密度的建模进行计算
	Number density = layer.exp_term * exp(layer.exp_scale * altitude) +
      layer.linear_term * altitude + layer.constant_term;
  return clamp(density, Number(0.0), Number(1.0));
}

Number GetProfileDensity(IN(DensityProfile) profile, Length altitude) {
  	// 这里分为两个 layers 主要是对臭氧Ozone层进行建模，和瑞利散射和米氏散射无关
    return altitude < profile.layers[0].width ?
    	GetLayerDensity(profile.layers[0], altitude) :
      	GetLayerDensity(profile.layers[1], altitude);
}

Length ComputeOpticalLengthToTopAtmosphereBoundary(
    IN(AtmosphereParameters) atmosphere, IN(DensityProfile) profile,
    Length r, Number mu) {
    assert(r >= atmosphere.bottom_radius && r <= atmosphere.top_radius);
    assert(mu >= -1.0 && mu <= 1.0);
    // Number of intervals for the numerical integration.
    const int SAMPLE_COUNT = 500;
    // The integration step, i.e. the length of each integration interval.
    Length dx =
          DistanceToTopAtmosphereBoundary(atmosphere, r, mu) / Number(SAMPLE_COUNT);
    // Integration loop.
    Length result = 0.0 * m;
    for (int i = 0; i <= SAMPLE_COUNT; ++i) {
        Length d_i = Number(i) * dx;
        // Distance between the current sample point and the planet center.
        Length r_i = sqrt(d_i * d_i + 2.0 * r * mu * d_i + r * r);
        // Number density at the current sample point (divided by the number density
        // at the bottom of the atmosphere, yielding a dimensionless number).
        Number y_i = GetProfileDensity(profile, r_i - atmosphere.bottom_radius);
        // Sample weight (from the trapezoidal rule).
        Number weight_i = i == 0 || i == SAMPLE_COUNT ? 0.5 : 1.0;
        result += y_i * weight_i * dx;
    }
    return result;
}
~~~

使用这个函数 $p,i$ 之间的透射率现在很容易计算(我们假设该线段不和地面相交)：

~~~c++
DimensionlessSpectrum ComputeTransmittanceToTopAtmosphereBoundary(
    IN(AtmosphereParameters) atmosphere, Length r, Number mu) {
  assert(r >= atmosphere.bottom_radius && r <= atmosphere.top_radius);
  assert(mu >= -1.0 && mu <= 1.0);
  return exp(-(
      atmosphere.rayleigh_scattering *
          ComputeOpticalLengthToTopAtmosphereBoundary(
              atmosphere, atmosphere.rayleigh_density, r, mu) +
      atmosphere.mie_extinction *
          ComputeOpticalLengthToTopAtmosphereBoundary(
              atmosphere, atmosphere.mie_density, r, mu) +
      atmosphere.absorption_extinction *
          ComputeOpticalLengthToTopAtmosphereBoundary(
              atmosphere, atmosphere.absorption_density, r, mu)));
}
~~~

### Precomputation

上述函数的计算开销很大，并且计算单极或者多级散射要需要大量的计算。幸运的是，这个函数只依赖两个参数并且相当平滑，所以我们可以先在一张小的2D texture 上面进行预计算来优化大量的计算过程。

为了实现这个预计算，因为这些参数没有共同的单位和值域范围，我们需要将函数参数 $(r,\mu)$ 和纹理坐标 $(u,v)$ 进行映射，反之也是，需要反过来映射以备后面读取结果。即便如此，将区间 $[0,1]$ 内的函数 $f$ 存储到尺寸为 $n$ 的贴图需要在函数 $0.5/n,1.5/n,\dots(n-0.5)/n$ 进行采样，因为对纹理采样是读取纹素的中心。因此该纹理在定义边界(0和1)只能给出推测的结果。为了避免这种问题，我们需要在第 0 个像素中心存储 $f(0)$，将 $f(1)$ 存储在 $n-1$ 的纹素中心。通过下面值域 $[0,1]$ 的 $x$ 映射到范围为 $[0.5/n,1-0.5/n]$ 的纹理坐标 $u$，和这个映射的逆过程：

~~~c++
Number GetTextureCoordFromUnitRange(Number x, int texture_size) {
  return 0.5 / Number(texture_size) + x * (1.0 - 1.0 / Number(texture_size));
}

Number GetUnitRangeFromTextureCoord(Number u, int texture_size) {
  return (u - 0.5 / Number(texture_size)) / (1.0 - 1.0 / Number(texture_size));
}
~~~

$$
\begin{aligned}
Set\;&n=4,step=\frac{1}{3}\\
0:&\frac{1}{8},\\
\frac{1}{3}:&\frac{1}{8}+\frac{1}{3}\frac{3}{4}=
\frac{3}{8}\\
\frac{2}{3}:&\frac{1}{8}+\frac{2}{3}\frac{3}{4}=
\frac{5}{8}\\
1:&\frac{1}{8}+\frac{3}{4}=
\frac{7}{8}
\end{aligned}
$$

<font color=#AAA>纹理size为 $n$ 则函数采样的分段数为$n-1$ 因为有头尾的0,1 这样两者才是相互映射的。</font>
$$
\begin{aligned}
Set\;&n=4,tex\;step=\frac{1}{4}\\
\frac{1}{8}:&(\frac{1}{8}-\frac{1}{8})
\frac{4}{3}=0\\
\frac{3}{8}:&(\frac{3}{8}-\frac{1}{8})
\frac{4}{3}=\frac{1}{3}\\
\end{aligned}
$$
![](/imgs/Atmosphere Ⅱ/functionAndTex.svg)  
使用该函数能够在 $(r,\mu)$ 和纹理坐标 $(u,v)$ 之间映射和反映射，在纹理查找中避免任何的外推结果。在[原本的实现中](http://evasion.inrialpes.fr/~Eric.Bruneton/PrecomputedAtmosphericScattering2.zip)该映射使用了一些地球大气情况下特定的常数。这里我们用一个通用的映射，适用于任何大气，但是仍然需要在视界边缘提供更多的采样。改进后的映射是基于其[论文](https://hal.inria.fr/inria-00288758/en)对于 4D 纹理的参数化描述：我们对 $r$ 使用同样的映射方式，对于 $\mu$ 采用轻量化的映射，(仅考虑“视线和地面无交点”的情景)。更准确的描述是，我们映射 $\mu$ 为一个值 $x_{\mu}$ 在 0,1 之间，该映射通过到顶部大气的距离 $d$，和其最小值和最大值作比较 $d_{min}=r_{top}-r,d_{max}=\rho+H$（见[论文](https://hal.inria.fr/inria-00288758/en)中的符号标注及下图）  
![](/imgs/Atmosphere Ⅱ/DistanceMap.svg)
$$
\begin{aligned}
&H=\sqrt{r_{top}^2-r_{bottom}^2}\\
&\rho=\sqrt{r^2-r_{bottom}^2}\\
&\rho和H与bottom大气相切,勾股定理\\
&d_{min}=r_{top}-r\\
&d_{max}=\rho+H\\
&d=-r\mu+\sqrt{r_{top}^2-r^2\sin\theta}\;\;刚才推导的
\end{aligned}
$$
映射过程来自：
$$
\begin{aligned}
&对\rho,d进行mapping，\\
&\rho=\sqrt{r^2-r_{bottom}^2},
r\in[r_{bottom},r_{top}],\rho\in[0,H]\\
&d\in[d_{min},d_{max}]
\end{aligned}
$$


根据这些定义，从 $(r,\mu)$ 到纹理坐标 $(u,v)$ 的映射可以被实现为:

~~~c++
vec2 GetTransmittanceTextureUvFromRMu(IN(AtmosphereParameters) atmosphere,
    Length r, Number mu) {
  assert(r >= atmosphere.bottom_radius && r <= atmosphere.top_radius);
  assert(mu >= -1.0 && mu <= 1.0);
  // Distance to top atmosphere boundary for a horizontal ray at ground level.
  Length H = sqrt(atmosphere.top_radius * atmosphere.top_radius -
      atmosphere.bottom_radius * atmosphere.bottom_radius);
  // Distance to the horizon.
  Length rho =
      SafeSqrt(r * r - atmosphere.bottom_radius * atmosphere.bottom_radius);
  // Distance to the top atmosphere boundary for the ray (r,mu), and its minimum
  // and maximum values over all mu - obtained for (r,1) and (r,mu_horizon).
  Length d = DistanceToTopAtmosphereBoundary(atmosphere, r, mu);
  Length d_min = atmosphere.top_radius - r;
  Length d_max = rho + H;
  Number x_mu = (d - d_min) / (d_max - d_min);
  Number x_r = rho / H;
  return vec2(GetTextureCoordFromUnitRange(x_mu, TRANSMITTANCE_TEXTURE_WIDTH),
              GetTextureCoordFromUnitRange(x_r, TRANSMITTANCE_TEXTURE_HEIGHT));
}
~~~

逆映射的过程为：

~~~c++
void GetRMuFromTransmittanceTextureUv(IN(AtmosphereParameters) atmosphere,
    IN(vec2) uv, OUT(Length) r, OUT(Number) mu) {
  assert(uv.x >= 0.0 && uv.x <= 1.0);
  assert(uv.y >= 0.0 && uv.y <= 1.0);
  Number x_mu = GetUnitRangeFromTextureCoord(uv.x, TRANSMITTANCE_TEXTURE_WIDTH);
  Number x_r = GetUnitRangeFromTextureCoord(uv.y, TRANSMITTANCE_TEXTURE_HEIGHT);
  // Distance to top atmosphere boundary for a horizontal ray at ground level.
  Length H = sqrt(atmosphere.top_radius * atmosphere.top_radius -
      atmosphere.bottom_radius * atmosphere.bottom_radius);
  // Distance to the horizon, from which we can compute r:
  Length rho = H * x_r;
  r = sqrt(rho * rho + atmosphere.bottom_radius * atmosphere.bottom_radius);
  // Distance to the top atmosphere boundary for the ray (r,mu), and its minimum
  // and maximum values over all mu - obtained for (r,1) and (r,mu_horizon) -
  // from which we can recover mu:
  Length d_min = atmosphere.top_radius - r;
  Length d_max = rho + H;
  Length d = d_min + x_mu * (d_max - d_min);
  mu = d == 0.0 * m ? Number(1.0) : (H * H - rho * rho - d * d) / (2.0 * r * d);
  mu = ClampCosine(mu);
}
~~~

现在能够定义一个片元着色器函数去预计算 transmittance texture 的纹素：

~~~c++
DimensionlessSpectrum ComputeTransmittanceToTopAtmosphereBoundaryTexture(
    IN(AtmosphereParameters) atmosphere, IN(vec2) frag_coord) {
  const vec2 TRANSMITTANCE_TEXTURE_SIZE =
      vec2(TRANSMITTANCE_TEXTURE_WIDTH, TRANSMITTANCE_TEXTURE_HEIGHT);
  Length r;
  Number mu;
  GetRMuFromTransmittanceTextureUv(
      atmosphere, frag_coord / TRANSMITTANCE_TEXTURE_SIZE, r, mu);
  return ComputeTransmittanceToTopAtmosphereBoundary(atmosphere, r, mu);
}
~~~

### Lookup

通过预计算 texture 能够通过一张 lookup 纹理得到一点和大气顶部之间的 transmittance(假设不和地面相交)：

~~~c++
DimensionlessSpectrum GetTransmittanceToTopAtmosphereBoundary(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    Length r, Number mu) {
  assert(r >= atmosphere.bottom_radius && r <= atmosphere.top_radius);
  vec2 uv = GetTransmittanceTextureUvFromRMu(atmosphere, r, mu);
  return DimensionlessSpectrum(texture(transmittance_texture, uv));
}
~~~

并且我们还可以从下图推导，  
![](/imgs/Atmosphere Ⅱ/Transmittance.svg)
$$
\begin{aligned}
&O\;is\;Earth\;center.\\
&For\;point\;q,we\;get\\
&O是地球球心，对于q点我们能计算得到\\
&r_d=\rvert oq\lvert=\sqrt{d^2+2r\mu d+r^2}\\
&the\;view\; zenith\; angle\;for\;p\;is\;the\;angle\;between\;\vec{oq}\;and\;\vec{pi}\\
&p点的天顶角是\vec{oq}\;和\;\vec{pi}夹角\\
&\mu_d=\frac{oq\cdot pi}{\rvert oq\lvert\rvert pi\lvert}
\end{aligned}
$$
有了在点 p 的 $r,\mu$ 我们仅需要两次纹理查找就可以计算出在大气中任意两点 p,q 之间的 transmittance。(重新回顾一下，p,q 之间的 transmittance 是p和大气顶部的 transmittance 除以 q 和大气顶部的 transmittance，反之亦然，这里还是假定两点之间的线段不和地面相交)

(注：这里不和地面相交指的是不会有下图情况)  
![](/imgs/Atmosphere Ⅱ/SegmentInstersect.svg)

~~~c++
DimensionlessSpectrum GetTransmittance(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    Length r, Number mu, Length d, bool ray_r_mu_intersects_ground) {
  assert(r >= atmosphere.bottom_radius && r <= atmosphere.top_radius);
  assert(mu >= -1.0 && mu <= 1.0);
  assert(d >= 0.0 * m);

  Length r_d = ClampRadius(atmosphere, sqrt(d * d + 2.0 * r * mu * d + r * r));
  Number mu_d = ClampCosine((r * mu + d) / r_d);

  if (ray_r_mu_intersects_ground) {
    return min(
        GetTransmittanceToTopAtmosphereBoundary(
            atmosphere, transmittance_texture, r_d, -mu_d) /
        GetTransmittanceToTopAtmosphereBoundary(
            atmosphere, transmittance_texture, r, -mu),
        DimensionlessSpectrum(1.0));
  } else {
    return min(
        GetTransmittanceToTopAtmosphereBoundary(
            atmosphere, transmittance_texture, r, mu) /
        GetTransmittanceToTopAtmosphereBoundary(
            atmosphere, transmittance_texture, r_d, mu_d),
        DimensionlessSpectrum(1.0));
  }
}
~~~

如果 $r,\mu$ 定义的射线和地面相交，这里的 `ray_r_mu_intersects_ground` 应该为真。  
我们不会用 `RayIntersectsGround` 计算其是否和地面相交，因为当射线和地平线相当接近时，由于浮点数运算的精度有限以及舍入误差，得到结果可能是错误的。并且调用者往往有更可靠的方法来判断射线是否和地面相交。

最后我们也需要大气中一点和太阳之间的 transmittance。因为太阳并不是一个点光源，所以这是一个对太阳圆面(Sun disc) transmittance 的积分。这里我们认为圆面上的 transmittance 是固定的，除了在地平线以下的部分，地平线以下部分认为其 transmittance 是 0。因此到太阳的 transmittance 可以通过`GetTransmittanceToTopAtmosphereBoundary`计算，再乘以太阳圆面位于地平线上的比例可以得到。  
当太阳天顶角 $\theta_s$ 大于地平面天顶角 $\theta_h$ 和太阳半径角度 $\alpha_s$ 时这个比例为0，当 $\theta_s$ 小于 $\theta_h-\alpha_s$ 该比例为 1。  
![](/imgs/Atmosphere Ⅱ/SunZenith.svg)  
等价为，当 $\mu_s=\cos\theta_s$ 小于 $\cos(\theta_h+\alpha_s)\approx\cos\theta_h-\alpha_s\sin\theta_h$ (一阶泰勒展开$f(x)\approx f(\theta_h)+f'(\theta_h)(x-\theta_h)$) 比例为 0，当 $\mu_s$ 大于 $\cos(\theta_h-\alpha_s)\approx\cos\theta_h+\alpha_s\sin\theta_h$ 时比例为 1。在两者之间，太阳可见圆面比例近似为平滑阶跃函数(smoothstep, 这可以通过绘制[圆弧弓形](https://en.wikipedia.org/wiki/Circular_segment)面积关于[弓形高](https://en.wikipedia.org/wiki/Sagitta_(geometry))的函数关系来验证)。因此，有 $\sin\theta_h=r_{bottom}/r$，我们可以用以下方式估计到太阳的 transmittance：

~~~c++
DimensionlessSpectrum GetTransmittanceToSun(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    Length r, Number mu_s) {
  Number sin_theta_h = atmosphere.bottom_radius / r;
  Number cos_theta_h = -sqrt(max(1.0 - sin_theta_h * sin_theta_h, 0.0));
  return GetTransmittanceToTopAtmosphereBoundary(
          atmosphere, transmittance_texture, r, mu_s) *
      smoothstep(-sin_theta_h * atmosphere.sun_angular_radius / rad,
                 sin_theta_h * atmosphere.sun_angular_radius / rad,
                 mu_s - cos_theta_h);
}
// 上图左侧为，mu_s - cos_theta_h <= 0, 为 -sin_theta_h * atmosphere.sun_angular_radius / rad
// 右侧 mu_s - cos_theta_h >= 1, 为 sin_theta_h * atmosphere.sun_angular_radius / rad
~~~

$$
\cos\theta_h-\alpha_s\sin\theta_h,
\cos\theta_h+\alpha_s\sin\theta_h\\
\mu_s-\cos\theta_h
$$

## Single scattering

单次散射指的是从太阳出发的光线经过一次散射事件后到达大气中某一点(可能是由空气分子或气溶胶颗粒引起的；我们排除来自地面的反射，这部分单独计算)

## Latex Picture

- figure 1  

~~~latex
\begin{tikzpicture}

  % 画同心圆弧 (半径=2cm 和 3cm, 从 30° 到 120°)
  \draw[line width=0.5pt] (60:4cm) arc[start angle=60, end angle=120, radius=4cm];
  \draw[line width=0.5pt, name path=topArc] (60:5.5cm) arc[start angle=60, end angle=120, radius=5.5cm];
  \fill (0,2) circle (0.05);
  \node[font=\bfseries,scale=0.8] at (0,2) [above right] {o};

  % x axis
  \draw[line width=0.2pt,->] (0,2) -- (1,2);
  \node[scale=0.8] at (1,2) [above right] {x};
  
  % z axis
  \draw[line width=0.2pt] (0,2) -- (0,2.4);
  \draw[line width=0.2pt,->] (0,2.75) -- (0,3);
  \node[scale=0.8] at (0,3) [above left] {z};
  % symbol on z axis
  \draw[line width=0.2pt] (-0.2,2.4) -- (-0.05,2.55);
  \draw[line width=0.2pt] (-0.05,2.55) -- (0.1,2.4);
  \draw[line width=0.2pt] (0.1,2.4) -- (0.25,2.55);
  \draw[line width=0.2pt] (-0.2,2.55) -- (-0.05,2.7);
  \draw[line width=0.2pt] (-0.05,2.7) -- (0.1,2.55);
  \draw[line width=0.2pt] (0.1,2.55) -- (0.25,2.7);

  % p point
  \fill (0,4.6) circle (0.05);
  \node[font=\bfseries,scale=0.7] at (0,4.6) [above left] {p};
  \draw[line width=0.2pt] (0,3) -- (0,4.6);
  \draw[line width=0.2pt] (0,4.6) -- (0,5.2);

  % p i line
  \draw[line width=0.2pt,name path=pii] (0,4.6) -- (3,5);

  % angle mu
  \draw[line width=0.2pt,dash pattern=on 1.5pt off 1pt, name path=angle]
   (0.3,4.64) arc[start angle=10, end angle=90, radius=0.3cm]
  node [right, xshift=0.1cm, yshift=0.1cm,scale = 0.8] {$\mu=\cos(\theta)$};

  \fill[name intersections={of=topArc and pii}]%, 
                            %  by=inter,  % 交点命名
                            %  total=1}]  % 只取第一个交点
  (intersection-1) circle (0.05)
  node [above, scale=0.80, font=\bfseries] {i};
  
  % point q
  \fill (1.5,4.8) circle(0.05)
  node[below,scale=0.8, font=\bfseries] {q};

  % r
  \draw[line width = 0.2pt, dash pattern = on 1.5pt off 1pt,<->]
   (-1,2) -- (-1,4.6);
  \node at (-1,3.3) [left,scale=0.8] {r};
   
  \draw[line width = 0.2pt]
  (-1.1,2) -- (-0.9,2);
  \draw[line width = 0.2pt]
  (-1.1,4.6) -- (-0.9,4.6);

\end{tikzpicture}
~~~


## Reference

[ebruneton/precomputed_atmospheric_scattering: This project provides a new implementation of our EGSR 2008 paper "Precomputed Atmospheric Scattering".](https://github.com/ebruneton/precomputed_atmospheric_scattering)  
[Precomputed Atmospheric Scattering](https://inria.hal.science/file/index/docid/288758/filename/article.pdf)

[ebruneton.github.io/precomputed_atmospheric_scattering/atmosphere/functions.glsl.html](https://ebruneton.github.io/precomputed_atmospheric_scattering/atmosphere/functions.glsl.html)  
[Volumetric Atmospheric Scattering - Alan Zucconi](https://www.alanzucconi.com/2017/10/10/atmospheric-scattering-1/)

* Next Epic 的 Atmosphere Scattering  
  [Publications](https://sebh.github.io/publications/)  
  [Production Ready Atmosphere Rendering](https://sebh.github.io/publications/egsr2020.pdf)
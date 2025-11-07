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

单次散射指的是从太阳出发的光线经过一次散射事件后到达大气中某一点(可能是由空气分子或气溶胶颗粒引起的；我们排除来自地面的反射，这部分在[Ground irradiance](https://ebruneton.github.io/precomputed_atmospheric_scattering/atmosphere/functions.glsl.html#irradiance)单独计算)。下面小节将描述如何进行计算并存储在预计算纹理中，并如何进行读取。

### Computation

考虑下面情景，太阳光在 q 点在到达另一点 p 前因为空气分子发生散射。(如果因为气溶胶发生散射，理论和下面相同，只是把瑞利散射换为米氏散射 "Rayleigh" to "Mie"):  
![](/imgs/Atmosphere Ⅱ/SingleScattering.svg)  
到达 p 的 radiance 由以下部分组成：

* 大气顶部的太阳 irradiance，
* 太阳到点 $q$ 的 transmittance (即，大气顶部的太阳光，能够穿透大气到达点 $q$ 的比例)，
* $q$ 点的 Rayleigh 散射系数 (即，到达点 $q$ 的光被散射到任意方向的系数)，
* Rayleigh 散射的方程 (即，在 $q$ 点被散射后实际朝向点 $p$ 传播的比例)，
* 点 $q,p$ 之间的 transmittance (即，在点 $q$ 朝向 $p$ 并到达 $p$ 的比例)。

因此，记 $\omega_s$ 为指向太阳的单位方向向量:

* $r=\lvert \vec{op}\rvert$
* $d=\lvert \vec{pq}\rvert$
* $\mu=(\vec{op}\cdot\vec{pq})/rd$
* $\mu_s=(\vec{op}\cdot{\omega_s})/r$
* $\nu=(\vec{pq}\cdot\omega_s)/d$

点 $q$ 的 $r,\mu_s$ 值为：

* $r_d=\lvert \vec{oq}\rvert=\sqrt{d^2+2r\mu d+r^2}$
* $\mu_{s,d}=(\vec{oq\cdot\omega_s})/r_d=\big((\vec{op}+\vec{pq})\cdot\omega_s\big)/r_d=(r\mu_s+d\upsilon)/r_d$

Rayleigh 和 Mie 散射的单次散射分量可以按照下面所述方法进行 (需要注意的是，出于效率考虑，我们省略了太阳的 irradiance 和 相位方程项，以及大气底部的散射系数，我们在后续中进行补充)

~~~c++
void ComputeSingleScatteringIntegrand(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    Length r, Number mu, Number mu_s, Number nu, Length d,
    bool ray_r_mu_intersects_ground,
    OUT(DimensionlessSpectrum) rayleigh, OUT(DimensionlessSpectrum) mie) {
	Length r_d = ClampRadius(atmosphere, sqrt(d * d + 2.0 * r * mu * d + r * r));
	Number mu_s_d = ClampCosine((r * mu_s + d * nu) / r_d);
	// pq transmittance * q sun transmittance?
    DimensionlessSpectrum transmittance =
      GetTransmittance(
          atmosphere, transmittance_texture, r, mu, d,
          ray_r_mu_intersects_ground) *
      GetTransmittanceToSun(
          atmosphere, transmittance_texture, r_d, mu_s_d);
	rayleigh = transmittance * GetProfileDensity(
      atmosphere.rayleigh_density, r_d - atmosphere.bottom_radius);
	mie = transmittance * GetProfileDensity(
      atmosphere.mie_density, r_d - atmosphere.bottom_radius);
}
~~~

考虑从一个给定方向 $\omega$，到达点 $q$ 的仅经过一次散射事件的太阳光。散射事件可能发生在点 $p$ 和沿射线 $[p,\omega)$ 与最近顶部大气层交点 $i$ 之间的任意一点 $q$。因此，从方向 $\omega$ 到达点 $p$ 的单次散射是针对所有在点 $p,i$ 之间的点 $q$，从点 $q$ 到 $p$ 的单次散射 radiance 的合。为了计算它，我们需要先计算长度 $\lvert pi\rvert$:  
![](/imgs/Atmosphere Ⅱ/SingleScattering2.svg)

~~~c++
Length DistanceToNearestAtmosphereBoundary(IN(AtmosphereParameters) atmosphere,
    Length r, Number mu, bool ray_r_mu_intersects_ground) {
  if (ray_r_mu_intersects_ground) {
    return DistanceToBottomAtmosphereBoundary(atmosphere, r, mu);
  } else {
    return DistanceToTopAtmosphereBoundary(atmosphere, r, mu);
  }
}
~~~

单次散射合可以按如下计算(使用[Trapezoidal rule](https://en.wikipedia.org/wiki/Trapezoidal_rule))：

~~~c++
void ComputeSingleScattering(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    Length r, Number mu, Number mu_s, Number nu,
    bool ray_r_mu_intersects_ground,
    OUT(IrradianceSpectrum) rayleigh, OUT(IrradianceSpectrum) mie) {
    assert(r >= atmosphere.bottom_radius && r <= atmosphere.top_radius);
    assert(mu >= -1.0 && mu <= 1.0);
    assert(mu_s >= -1.0 && mu_s <= 1.0);
    assert(nu >= -1.0 && nu <= 1.0);

    // Number of intervals for the numerical integration.
    const int SAMPLE_COUNT = 50;
    // The integration step, i.e. the length of each integration interval.
	// |pi|/sample_count
    Length dx =
      DistanceToNearestAtmosphereBoundary(atmosphere, r, mu,
          ray_r_mu_intersects_ground) / Number(SAMPLE_COUNT);
    // Integration loop.
    DimensionlessSpectrum rayleigh_sum = DimensionlessSpectrum(0.0);
    DimensionlessSpectrum mie_sum = DimensionlessSpectrum(0.0);
    for (int i = 0; i <= SAMPLE_COUNT; ++i) {
        Length d_i = Number(i) * dx;
        // The Rayleigh and Mie single scattering at the current sample point.
        DimensionlessSpectrum rayleigh_i;
        DimensionlessSpectrum mie_i;
        ComputeSingleScatteringIntegrand(atmosphere, transmittance_texture,
            r, mu, mu_s, nu, d_i, ray_r_mu_intersects_ground, rayleigh_i, mie_i);
        // Sample weight (from the trapezoidal rule).
        Number weight_i = (i == 0 || i == SAMPLE_COUNT) ? 0.5 : 1.0;
        rayleigh_sum += rayleigh_i * weight_i;
        mie_sum += mie_i * weight_i;
    }
    rayleigh = rayleigh_sum * dx * atmosphere.solar_irradiance *
      atmosphere.rayleigh_scattering;
    mie = mie_sum * dx * atmosphere.solar_irradiance * atmosphere.mie_scattering;
}
~~~

这里`i==0 || i==SAMPLE_COUNT` 权重为 0.5，是因为  
![](/imgs/Atmosphere Ⅱ/SingleScattering3.svg)  
上面刻度为每个采样点的位置，下面表示每个采样点所代表的采样段，对于采样点0和i 其覆盖的距离为 dx 的一半。

注意在这里我们添加了在 `ComputeSingleScatteringIntergrand` 中忽略的 太阳 irradiance 和 散射系数，但是没有添加 相位函数项，为了在角度上有更高精度我们在 [渲染时](https://ebruneton.github.io/precomputed_atmospheric_scattering/atmosphere/functions.glsl.html#rendering) 添加。如下我们给出完整的相位函数：

~~~c++
InverseSolidAngle RayleighPhaseFunction(Number nu) {
  InverseSolidAngle k = 3.0 / (16.0 * PI * sr);
  return k * (1.0 + nu * nu);
}

InverseSolidAngle MiePhaseFunction(Number g, Number nu) {
  InverseSolidAngle k = 3.0 / (8.0 * PI * sr) * (1.0 - g * g) / (2.0 + g * g);
  return k * (1.0 + nu * nu) / pow(1.0 + g * g - 2.0 * g * nu, 1.5);
}
~~~

### Precomputation

`ComputeSingleScattering` 函数计算消耗大，并且计算多级散射需要大量的计算。因此我们希望将预计算结果存储在 texture 中，该纹理需要将函数的 4 个参数映射到纹理坐标。假设我们拥有一个 4D texture，需要将 $(r,\mu,\mu_s,\upsilon)$ 映射到 $(x,y,z,w)$。下属函数实现了[论文](https://hal.inria.fr/inria-00288758/en)定义的映射方法并进行了一些小改动(参阅论文和上述图标的标注)：

* $\mu$ 的映射会考虑到最近大气边界的最小距离，从而将 $\mu$ 映射到完整的 $[0,1]$ 区间 (原映射未覆盖完整的 $[0,1]$)。
* 对于 $\mu_s$ 的映射比论文中更通用 (原版的映射使用了针对地球大气的专用常量)。该映射基于(太阳光线)到顶部大气的距离，和 $\mu$ 的映射类似，并只使用了一个(可配置的)专有参数。同时和原本定义一样，能够在近地平线范围提供更高采样。

论文中的映射：
$$
\begin{aligned}
r:&\mu_r=\frac{\rho}{H}\\
\mu:&\mu_\mu=\frac{1}{2}+\frac{(r\mu+\sqrt{\Delta})}{2\rho}
.\;if\;r\mu<0\;and\;\Delta>0\\
&\frac{1}{2}-\frac{(r\mu-\sqrt{\Delta+H^2})}{2\rho+2H}.\;otherwise\\
\mu_s:&\mu_{\mu_s}=\frac{(1-e^{-3\mu_s-0.6})}{(1-e^{-3.6})}\\
\nu:&\mu_\nu=\frac{(1+\upsilon)}{2}\\
&\rho=(r^2-R_g^2)^{\frac{1}{2}},
H=(R_t^2-R_g^2)^{\frac{1}{2}},
\Delta=r^2\mu^2-\rho^2=R_g^2-r^2\sin\theta.\\
&\Delta+H^2=R_t^2-r^2\sin\theta
\end{aligned}
$$

$R_t$ 大气顶部半径，$R_g$ 大气底部半径，$r$ 采样点半径，$\mu$ 采样点视线方向的天顶角余弦。

新参数化为：

~~~c++
vec4 GetScatteringTextureUvwzFromRMuMuSNu(IN(AtmosphereParameters) atmosphere,
    Length r, Number mu, Number mu_s, Number nu,
    bool ray_r_mu_intersects_ground) {
    assert(r >= atmosphere.bottom_radius && r <= atmosphere.top_radius);
    assert(mu >= -1.0 && mu <= 1.0);
    assert(mu_s >= -1.0 && mu_s <= 1.0);
    assert(nu >= -1.0 && nu <= 1.0);

    // Distance to top atmosphere boundary for a horizontal ray at ground level.
    // 地平线上水平视线的到大气顶部距离
    Length H = sqrt(atmosphere.top_radius * atmosphere.top_radius -
      atmosphere.bottom_radius * atmosphere.bottom_radius);
    // Distance to the horizon.
    // 到地平线距离
    Length rho =
      SafeSqrt(r * r - atmosphere.bottom_radius * atmosphere.bottom_radius);
    Number u_r = GetTextureCoordFromUnitRange(rho / H, SCATTERING_TEXTURE_R_SIZE);

    // Discriminant of the quadratic equation for the intersections of the ray
    // (r,mu) with the ground (see RayIntersectsGround).
    // 射线（r, mu）与地面相交的二次方程的判别式（参见 RayIntersectsGround 函数）。
    Length r_mu = r * mu;
    Area discriminant =
      r_mu * r_mu - r * r + atmosphere.bottom_radius * atmosphere.bottom_radius;
    Number u_mu;
    if (ray_r_mu_intersects_ground) {
        // Distance to the ground for the ray (r,mu), and its minimum and maximum
        // values over all mu - obtained for (r,-1) and (r,mu_horizon).
        // 射线(r,mu) 到地面的距离，和其最大最小值在 在(r,-1) 和 (r,mu_Horizon)
        Length d = -r_mu - SafeSqrt(discriminant);
        Length d_min = r - atmosphere.bottom_radius;
        Length d_max = rho;
        // d_max == d_min 说明采样点在 大气底部
        u_mu = 0.5 - 0.5 * GetTextureCoordFromUnitRange(d_max == d_min ? 0.0 :
            (d - d_min) / (d_max - d_min), SCATTERING_TEXTURE_MU_SIZE / 2);
    } else {
        // Distance to the top atmosphere boundary for the ray (r,mu), and its
        // minimum and maximum values over all mu - obtained for (r,1) and
        // (r,mu_horizon).
        // 射线(r,mu) 到大气顶部距离，最大最小值 (r,1),(r,mu_Horizon)
        Length d = -r_mu + SafeSqrt(discriminant + H * H);
        Length d_min = atmosphere.top_radius - r;
        Length d_max = rho + H;
        u_mu = 0.5 + 0.5 * GetTextureCoordFromUnitRange(
            (d - d_min) / (d_max - d_min), SCATTERING_TEXTURE_MU_SIZE / 2);
    }

    Length d = DistanceToTopAtmosphereBoundary(
      atmosphere, atmosphere.bottom_radius, mu_s);
    Length d_min = atmosphere.top_radius - atmosphere.bottom_radius;
    Length d_max = H;
    Number a = (d - d_min) / (d_max - d_min);
    Length D = DistanceToTopAtmosphereBoundary(
      atmosphere, atmosphere.bottom_radius, atmosphere.mu_s_min);
    Number A = (D - d_min) / (d_max - d_min);
    // An ad-hoc function equal to 0 for mu_s = mu_s_min (because then d = D and
    // thus a = A), equal to 1 for mu_s = 1 (because then d = d_min and thus
    // a = 0), and with a large slope around mu_s = 0, to get more texture 
    // samples near the horizon.
    // 这是一个特设函数：当mu_s=mu_s_min时函数值为0（因为此时d=D，故a=A），
    // 当mu_s=1时函数值为1（因为此时d=d_min，故a=0），
    // 且在mu_s=0附近斜率较大，以便在地平线附近获取更多纹理采样。
    Number u_mu_s = GetTextureCoordFromUnitRange(
      max(1.0 - a / A, 0.0) / (1.0 + a), SCATTERING_TEXTURE_MU_S_SIZE);

    Number u_nu = (nu + 1.0) / 2.0;
    return vec4(u_nu, u_mu_s, u_mu, u_r);
}
~~~


对于采样点高度映射：
$$
\begin{aligned}
&H=\sqrt{R_t^2-R_g^2}\\
&\rho=\sqrt{r^2-R_g^2}\\
&u_r=\frac{\rho}{H}\\
\end{aligned}
$$
对于采样点视线的天顶角映射：
$$
\begin{aligned}

&\Delta=r^2\mu^2-r^2+R_g^2=R_g^2-r^2(\sin\theta)^2\\
&r\;\mu\;cross\;ground
\left\{
	\begin{aligned}
	&d=-r\mu-\sqrt{\Delta}\\
    &d_{min}=r-R_g\\
    &d_{max}=\rho\\
    &u_\mu=0.5-0.5\frac{d-d_{min}}{d_{max}-d_{min}}\\
	\end{aligned}
\right.\\
&no\;cross\;ground
\left\{
	\begin{aligned}
    &d=-r\mu+\sqrt{\Delta+H^2}\\
    &d_{min}=R_t-r\\
    &d_{max}=\rho+H\\
    &u_\mu=0.5+0.5\frac{d-d_{min}}{d_{max}-d_{min}}\\
	\end{aligned}
\right.\\
\end{aligned}
$$
对于太阳角度映射：
$$
\begin{aligned}
&d=distance\;to\;top\;with\;\mu_s\\
&d_{min}=R_t-R_g\\
&d_{max}=H\\
&a=\frac{d-d_{min}}{d_{max}-d_{min}}\\
&D=distance\;to\;top\;with\;\mu_{s_{min}}\\
&A=\frac{D-d_{min}}{d_{max}-d_{min}}\\
&u_{\mu_s}=\frac{A-a}{A(1+a)}
\end{aligned}
$$
对于视线角度映射：
$$
\begin{aligned}
u_{\nu}=\frac{\nu+1}{2}
\end{aligned}
$$
逆映射函数为：

~~~c++
void GetRMuMuSNuFromScatteringTextureUvwz(IN(AtmosphereParameters) atmosphere,
    IN(vec4) uvwz, OUT(Length) r, OUT(Number) mu, OUT(Number) mu_s,
    OUT(Number) nu, OUT(bool) ray_r_mu_intersects_ground) {
    assert(uvwz.x >= 0.0 && uvwz.x <= 1.0);
    assert(uvwz.y >= 0.0 && uvwz.y <= 1.0);
    assert(uvwz.z >= 0.0 && uvwz.z <= 1.0);
    assert(uvwz.w >= 0.0 && uvwz.w <= 1.0);

    // Distance to top atmosphere boundary for a horizontal ray at ground level.
    Length H = sqrt(atmosphere.top_radius * atmosphere.top_radius -
      atmosphere.bottom_radius * atmosphere.bottom_radius);
    // Distance to the horizon.
    Length rho =
      H * GetUnitRangeFromTextureCoord(uvwz.w, SCATTERING_TEXTURE_R_SIZE);
    r = sqrt(rho * rho + atmosphere.bottom_radius * atmosphere.bottom_radius);

    if (uvwz.z < 0.5) {
        // Distance to the ground for the ray (r,mu), and its minimum and maximum
        // values over all mu - obtained for (r,-1) and (r,mu_horizon) - from which
        // we can recover mu:
        Length d_min = r - atmosphere.bottom_radius;
        Length d_max = rho;
        Length d = d_min + (d_max - d_min) * GetUnitRangeFromTextureCoord(
            1.0 - 2.0 * uvwz.z, SCATTERING_TEXTURE_MU_SIZE / 2);
        // d == 0.0 意味着采样点在大气底部，并且视线向正下方。
        mu = d == 0.0 * m ? Number(-1.0) :
            ClampCosine(-(rho * rho + d * d) / (2.0 * r * d));
        ray_r_mu_intersects_ground = true;
    } else {
        // Distance to the top atmosphere boundary for the ray (r,mu), and its
        // minimum and maximum values over all mu - obtained for (r,1) and
        // (r,mu_horizon) - from which we can recover mu:
        Length d_min = atmosphere.top_radius - r;
        Length d_max = rho + H;
        Length d = d_min + (d_max - d_min) * GetUnitRangeFromTextureCoord(
            2.0 * uvwz.z - 1.0, SCATTERING_TEXTURE_MU_SIZE / 2);
        // d == 0, 意味着采样点在大气顶部，并且视线向正上方
        mu = d == 0.0 * m ? Number(1.0) :
            ClampCosine((H * H - rho * rho - d * d) / (2.0 * r * d));
        ray_r_mu_intersects_ground = false;
    }

    Number x_mu_s =
      GetUnitRangeFromTextureCoord(uvwz.y, SCATTERING_TEXTURE_MU_S_SIZE);
    Length d_min = atmosphere.top_radius - atmosphere.bottom_radius;
    Length d_max = H;
    Length D = DistanceToTopAtmosphereBoundary(
      atmosphere, atmosphere.bottom_radius, atmosphere.mu_s_min);
    Number A = (D - d_min) / (d_max - d_min);
    Number a = (A - x_mu_s * A) / (1.0 + x_mu_s * A);
    Length d = d_min + min(a, A) * (d_max - d_min);
    mu_s = d == 0.0 * m ? Number(1.0) :
     ClampCosine((H * H - d * d) / (2.0 * atmosphere.bottom_radius * d));

    nu = ClampCosine(uvwz.x * 2.0 - 1.0);
}
~~~

逆映射部分

对于采样点半径逆映射：
$$
\begin{aligned}
H&=\sqrt{R_t^2-R_g^2}\\
\rho&=u_rH\\
r&=\sqrt{(\rho)^2+R_g^2}
\end{aligned}
$$
对于采样点视线的天顶角逆映射：
$$
\begin{aligned}
&if\;u_\mu<0.5\\
&\left\{
	\begin{aligned}
	&d_{min}=r-R_g\\
	&d_{max}=\rho\\
	&d=d_{min}+(d_{max}-d_{min})(1-2u_\mu)\\
	&\mu=\left\{
		\begin{aligned}
		&-1,b==0\\
		&\frac{-(\rho*\rho+d*d)}{2rd}
		\end{aligned}
		\right.
    \end{aligned}
\right.\\
&余弦定理\\
&R_g^2=(r+d\mu)^2+d^2(\sin\theta)^2\\
&R_g^2=r^2+2rd\mu+d^2\mu^2+d^2(\sin\theta)^2\\
&R_g^2=r^2+2rd\mu+d^2\\
&\mu=\frac{R_g^2-r^2-d^2}{2rd}=\frac{-\rho^2-d^2}{2rd}\\
\\
&if\;u_\mu\ge0.5\\
&\left\{
	\begin{aligned}
	&d_{min}=R_{t}-r\\
	&d_{max}=\rho+H\\
	&d=d_{min}+(d_{max}-d_{min})(2u_\mu-1)\\
	&\mu=\left\{
		\begin{aligned}
		&1,b==0\\
		&\frac{H^2-\rho^2-d^2}{2rd}
		\end{aligned}
		\right.
	\end{aligned}
\right.\\
&(r+d\mu)^2+(d\sin\theta)^2=R_t^2\\
&r^2+2rd\mu+d^2=R_t^2\\
&\mu=\frac{R_t^2-r^2-d^2}{2rd}=
\frac{R_t^2-\rho^2-R_g^2-d^2}{2rd}=\frac{H^2-\rho^2-d^2}{2rd}
\end{aligned}
$$

左侧为视线和底部相交，右侧为视线和顶部相交。  
<img src="/imgs/Atmosphere Ⅱ/groundMapping.png" style="zoom:40%;" /><img src="/imgs/Atmosphere Ⅱ/TopMapping.png" style="zoom:40%;" />

对于太阳角度逆映射：
$$
\begin{aligned}
&d_{min}=R_t-R_g\\
&d_{max}=H\\
&D=distance\;to\;top\;with\;\mu_{s_{min}}\\
&A=\frac{D-d_{min}}{d_{max}-d_{min}}\\
&u_{\mu_s}=\frac{A-a}{A(1+a)}\\
&u_{\mu_s}A=(A-a)/(1+a)\\
&A-u_{\mu_s}A=(A+aA-A+a)/(1+a)=(aA+a)/(1+a)\\
&1+u_{\mu_s}A=(1+a+A-a)/(1+a)=(1+A)/(1+a)\\
&\frac{A-u_{\mu_s}A}{1+u_{\mu_s}A}=
\frac{a(1+A)}{(1+A)}=a\\
&d=d_{min}+\min(a,A)(d_{max}-d_{min})\\
&\mu_s=\frac{H^2-d^2}{2R_gd}\\
&(d\mu_s+R_g)^2+(d\sin\theta_s)^2=R_t^2\\
&d^2+2R_gd\mu_s+R_g^2=R_t^2\\
&\mu_s=\frac{R_t^2-R_g^2-d^2}{2R_gd}=\frac{H^2-d^2}{2R_gd}
\end{aligned}
$$
视线方向逆映射：
$$
\nu=u_\nu*2-1
$$
按照这种映射我们应该使用 4D Texture，实际上不是这样。因此需要建立在 3D纹理坐标 和 4D纹理坐标 间额外的映射。下述函数将一个 3D纹理坐标 拓展至 4D纹理坐标，进而转变为参数 $(r,\mu,\mu_s,\nu)$。这是通过从 $x$ 纹理坐标 “解包” 出两个纹理坐标实现的。注意在函数末尾对 $\nu$ 进行了钳制。这是因为 $\nu$ 并不是完全独立的变量，其取值范围取决于 $\mu,\mu_s$ (这点可以在从 天顶角、视线、太阳方向等单位向量的笛卡尔坐标计算 $\mu,\mu_s,\nu$ 观测到)，前面的函数隐式的假设了该约束(如果为满足该约束，其断言可能失败)

```c++
void GetRMuMuSNuFromScatteringTextureFragCoord(
    IN(AtmosphereParameters) atmosphere, IN(vec3) frag_coord,
    OUT(Length) r, OUT(Number) mu, OUT(Number) mu_s, OUT(Number) nu,
    OUT(bool) ray_r_mu_intersects_ground) {
	const vec4 SCATTERING_TEXTURE_SIZE = vec4(
        // 8-1, 32, 128, 32
        SCATTERING_TEXTURE_NU_SIZE - 1,
        SCATTERING_TEXTURE_MU_S_SIZE,
        SCATTERING_TEXTURE_MU_SIZE,
        SCATTERING_TEXTURE_R_SIZE);
    // x [0,n*mu_s_size]->[0,1,...,n]->[[0,1),[0,1),...,[0,1)]
  	Number frag_coord_nu =
          floor(frag_coord.x / Number(SCATTERING_TEXTURE_MU_S_SIZE));
	// x [0, n*mu_s_size]->[[0],[1],...,[n]]
    Number frag_coord_mu_s =
      	mod(frag_coord.x, Number(SCATTERING_TEXTURE_MU_S_SIZE));
  	vec4 uvwz =
      	vec4(frag_coord_nu, frag_coord_mu_s, frag_coord.y, frag_coord.z) /
          	SCATTERING_TEXTURE_SIZE;
  	GetRMuMuSNuFromScatteringTextureUvwz(
      	atmosphere, uvwz, r, mu, mu_s, nu, ray_r_mu_intersects_ground);
  	// Clamp nu to its valid range of values, given mu and mu_s.
  	nu = clamp(nu, mu * mu_s - sqrt((1.0 - mu * mu) * (1.0 - mu_s * mu_s)),
      	mu * mu_s + sqrt((1.0 - mu * mu) * (1.0 - mu_s * mu_s)));
}
```

$$
\begin{aligned}
&\cos(A-B)=\cos A\cos B+\sin A\sin B\\
&\cos(A+B)=\cos A\cos B-\sin A\sin B\\
&\mu\mu_s-\sqrt{(1-\mu^2)(1-\mu_s^2)},\mu\mu_s+\sqrt{(1-\mu^2)(1-\mu_s^2)}
\end{aligned}
$$

通过该映射我们能计算单次散射情况下 3D 纹理中的一个纹素：

~~~c++
void ComputeSingleScatteringTexture(IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture, IN(vec3) frag_coord,
    OUT(IrradianceSpectrum) rayleigh, OUT(IrradianceSpectrum) mie) {
  Length r;
  Number mu;
  Number mu_s;
  Number nu;
  bool ray_r_mu_intersects_ground;
  GetRMuMuSNuFromScatteringTextureFragCoord(atmosphere, frag_coord,
      r, mu, mu_s, nu, ray_r_mu_intersects_ground);
  ComputeSingleScattering(atmosphere, transmittance_texture,
      r, mu, mu_s, nu, ray_r_mu_intersects_ground, rayleigh, mie);
}
~~~

### Lookup

在预计算纹理的帮助下，我们可以通过两次纹理查找，获得一点和最近大气边界之间的散射。(需要进行两次 3D 纹理查找来模拟单次 4D 纹理查找的 quadrilinear interpolation —— 四线性插值。3D 纹理的坐标计算可以通过 `GetRMuMuSNuFromScatteringTextureFragCoord` 所用的 3D-4D映射 的逆过程来计算)：

~~~c++
TEMPLATE(AbstractSpectrum)
AbstractSpectrum GetScattering(
    IN(AtmosphereParameters) atmosphere,
    IN(AbstractScatteringTexture TEMPLATE_ARGUMENT(AbstractSpectrum))
        scattering_texture,
    Length r, Number mu, Number mu_s, Number nu,
    bool ray_r_mu_intersects_ground) {
  	vec4 uvwz = GetScatteringTextureUvwzFromRMuMuSNu(
      atmosphere, r, mu, mu_s, nu, ray_r_mu_intersects_ground);
  	Number tex_coord_x = uvwz.x * Number(SCATTERING_TEXTURE_NU_SIZE - 1);
  	Number tex_x = floor(tex_coord_x);
  	Number lerp = tex_coord_x - tex_x;
  	vec3 uvw0 = vec3((tex_x + uvwz.y) /
                     Number(SCATTERING_TEXTURE_NU_SIZE),
                     uvwz.z, uvwz.w);
  	vec3 uvw1 = vec3((tex_x + 1.0 + uvwz.y) /
                     Number(SCATTERING_TEXTURE_NU_SIZE),
                     uvwz.z, uvwz.w);
  	return AbstractSpectrum(
        texture(scattering_texture, uvw0) * (1.0 - lerp) +
      	texture(scattering_texture, uvw1) * lerp);
}
~~~

最后，我们提供一个在下一小节非常有用的，便利的查询函数。  
该函数返回两种结果：要么包含相位函数的单次散射，或是 n 级的散射(n>1)。它假设：若 `scattering_order` 严格大于 1，则 `multiple_scattering_texture` 需要对应此阶数的散射，且同时包含 Rayleigh, Mie 散射 和 相位函数项。

~~~c++
RadianceSpectrum GetScattering(
    IN(AtmosphereParameters) atmosphere,
    IN(ReducedScatteringTexture) single_rayleigh_scattering_texture,
    IN(ReducedScatteringTexture) single_mie_scattering_texture,
    IN(ScatteringTexture) multiple_scattering_texture,
    Length r, Number mu, Number mu_s, Number nu,
    bool ray_r_mu_intersects_ground,
    int scattering_order) {
  if (scattering_order == 1) {
    IrradianceSpectrum rayleigh = GetScattering(
        atmosphere, single_rayleigh_scattering_texture, r, mu, mu_s, nu,
        ray_r_mu_intersects_ground);
    IrradianceSpectrum mie = GetScattering(
        atmosphere, single_mie_scattering_texture, r, mu, mu_s, nu,
        ray_r_mu_intersects_ground);
    return rayleigh * RayleighPhaseFunction(nu) +
        mie * MiePhaseFunction(atmosphere.mie_phase_function_g, nu);
  } else {
    return GetScattering(
        atmosphere, multiple_scattering_texture, r, mu, mu_s, nu,
        ray_r_mu_intersects_ground);
  }
}
~~~

## Multiple scattering

多级散射是从太阳出发的光经过两次或更多弹射后，到达大气中某点(这里一次弹射既指一次散射事件也包括从地面反射回来)。下面小节将描述如何计算它，如何存储它到一张预计算纹理中，并且如何回读它。

注意，对于单次散射，我们忽略了最后弹射是地面反射的那些光线。这些光路的贡献在实时渲染时单独计算，以便考虑实际的地面反射率(对于预计算的地面中间反射，我们使用平均、统一的反照率)。

### Computation

多重散射可被分解为双重散射、三重散射等的和，其中每一项代表从太阳出发的光线经过精确的2，3次弹射后到达大气中的某点，而且每一项都可以由前面一项计算得到。实际上，方向 $\omega$ 经过 $n$ 次弹射到达点 $p$ 的光，需要对最后一次弹射中所有可能的 $q$ 点进行积分，这涉及到在 $n-1$ 次弹射后从任意方向到达点 $q$ 的光。

上述表示每个散射都需要从上一级散射中进行三重积分计算得到(一次积分覆盖了从点 $p$ 出发沿方向 $\omega$ 到最近大气边界路径上的所有点 $q$，以及嵌套的在每个点 $q$ 所有方向的双重积分)。因此如果我们想从零开始计算”每一阶“，对双重散射我们需要进行三重积分(<font color=#ADADAD>$p$ 在方向 $\omega$ 和最近大气边界上所有点——1重，该路径上某点的所有方向——2重，某点的某方向至最近大气上所有点——3重</font>)，对于三重散射需要进行六重积分，以此类推。这显然效率低下，因为存在大量冗余计算(第 $n$ 阶计算会重复前一阶的计算，导致总计算量随着阶数呈现二次增长)，更高效的方法是：

* 在 texture 预计算单次散射(就像上面描述那样)，
* 对于阶数 $n\ge2$:
  * 用三重积分预计算 $n$ 阶散射到 texture 中，被积函数用 $(n-1)$ 阶的散射纹理查找。

这种策略避免了大量的荣誉计算但是并没有消除全部的冗余。考虑下图情况，点 $p,p{'}$ 以及计算经过 $n$ 次弹射后，从方向 $\omega$ 到达两点的光线，所必须进行的计算。这些必须计算特别涉及对辐照度 $L$ 的评估，$L$是从各个方向经过 $n-1$ 次弹射后，从点 $q$ 沿方向 $-\omega$ 被散射的辐照度:  
![](/imgs/Atmosphere Ⅱ/NoNecessaryComputation.svg)  
因此，如果我们用上述的三重积分方法计算第 $n$ 阶的散射，会重复计算 $L$ 部分(实际上，对于在 $q$ 和沿方向 $-\omega$ 最近大气边界上的所有点 $p$ 都会重复计算 $L$ 部分)。为了避免这情况，提高多重散射的计算效率，我们通过下面办法改进上述的算法：

* 预计算单次散射在 texture 中(和上面一样)
* 对于 $n\ge2$:
  * 对每个点 $q$ 和方向 $\omega$，预计算经过 $n-1$ 次弹射后来自各个方向的，在点 $q$ 被散射到方向 $-\omega$ 的光(这仅包含一个被积函数使用 $(n-1)$ 散射纹理的二重积分<font color=A3A3A3>——在点 $q$ 对各个方向的积分——1重，沿被积的每个方向上各个点进行积分——2重 </font>)
  * 对于每个点 $p$ 和方向 $\omega$，预计算经过 $n$ 次弹射来自方向 $\omega$ 的光线(这仅包含一重积分，被积函数使用上一行计算的texture查询结果)

为了完善算法，现在需要指定如何执行上述循环中的两个步骤。这就是我们下一小节的工作。

#### First step

第一步计算被散射到大气中某一点 $q$，朝着某一方向 $-\omega$ 的 radiance。此外假设该散射事件是 $n$ 阶散射。

该 radiance 是所有可能入射方向 $\omega_i$ 的积分，积分项是下述几项的乘积：

* 经过 $n-1$ 次弹射后从方向 $\omega_i$ 到达点 $q$ 的 入射 radiance $L_i$ (这是下面两项之和)：
  * $(n-1)$ 阶预计算散射纹理项，

  * 如果射线 $[q,\omega_i)$ 和地面在 $r$ 相交，要考虑经过 $n-1$ 次弹射的光线并且最后一次弹射发生在 $r$ (即地面上的情况) 的光路贡献 (根据定义，这些路径被我们排除在预计算纹理之外，但是我们必须在这里将它们考虑在内，因为地面上的弹射后紧接着在点 $q$ 会发生一次弹射)。  
    这一贡献恰好是以下因素的乘积：

    * $q,r$ 之间的 transmittance

    * 平均的地面 albedo

    * [Lambertian BRDF](https://www.cs.princeton.edu/~smr/cs348c-97/surveypaper.html) $1/\pi$

    * 经过 $n-2$ 次弹射地面接受的 irradiance。在[下一小节](https://ebruneton.github.io/precomputed_atmospheric_scattering/atmosphere/functions.glsl.html#irradiance) 我们讲解如何在 texture 中预计算它。现在，假设可以用下述函数检索预计算纹理中的 irradiance：

      ~~~c++
      IrradianceSpectrum GetIrradiance(
          IN(AtmosphereParameters) atmosphere,
          IN(IrradianceTexture) irradiance_texture,
          Length r, Number mu_s);
      ~~~

* $q$ 点的散射系数

* 从方向 $\omega,\omega_i$ 间的相位方程。

 引出下面实现方法(其中，`multiple_scattering_texture` 包含 $(n-1)$ 阶的散射，如果 $n>2$，`irradiance_texture` 是经过 $n-2$ 次弹射后地面接受的 irradiance，并且 `scattering_order` 为 $n$):

~~~c++
// 第一步计算被散射到大气中某一点 q，朝着某一方向 -\omega 的 radiance
RadianceDensitySpectrum ComputeScatteringDensity(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    IN(ReducedScatteringTexture) single_rayleigh_scattering_texture,
    IN(ReducedScatteringTexture) single_mie_scattering_texture,
    IN(ScatteringTexture) multiple_scattering_texture,
    IN(IrradianceTexture) irradiance_texture,
    Length r, Number mu, Number mu_s, Number nu, int scattering_order)
{
    assert(r >= atmosphere.bottom_radius && r <= atmosphere.top_radius);
    assert(mu >= -1.0 && mu <= 1.0);
    assert(mu_s >= -1.0 && mu_s <= 1.0);
    assert(nu >= -1.0 && nu <= 1.0);
    assert(scattering_order >= 2);

    // 计算天顶角，视线omega和太阳方向omega_s的单位方向
    // 比如视线和天顶角的夹角mu的cos,
    // 太阳方向和天顶角夹角mu_s的cos
    // 视线和太阳方向夹角nu的余弦，目的是为了简化下面计算
    // Compute unit direction vectors for the zenith, the view direction omega and
    // and the sun direction omega_s, such that the cosine of the view-zenith
    // angle is mu, the cosine of the sun-zenith angle is mu_s, and the cosine of
    // the view-sun angle is nu. The goal is to simplify computations below.
    vec3 zenith_direction = vec3(0.0, 0.0, 1.0);
    vec3 omega = vec3(sqrt(1.0 - mu * mu), 0.0, mu);
    Number sun_dir_x = omega.x == 0.0 ? 0.0 : (nu - mu * mu_s) / omega.x;
    Number sun_dir_y = sqrt(max(1.0 - sun_dir_x * sun_dir_x - mu_s * mu_s, 0.0));
    vec3 omega_s = vec3(sun_dir_x, sun_dir_y, mu_s);

    const int SAMPLE_COUNT = 16;
    const Angle dphi = pi / Number(SAMPLE_COUNT);
    const Angle dtheta = pi / Number(SAMPLE_COUNT);
    RadianceDensitySpectrum rayleigh_mie =
      RadianceDensitySpectrum(0.0 * watt_per_cubic_meter_per_sr_per_nm);

    // 对所有入射方向 \omega_i 的积分使用嵌套循环
    // Nested loops for the integral over all the incident directions omega_i.
    for (int l = 0; l < SAMPLE_COUNT; ++l)
    {
        Angle theta = (Number(l) + 0.5) * dtheta;
        Number cos_theta = cos(theta);
        Number sin_theta = sin(theta);
        bool ray_r_theta_intersects_ground =
            RayIntersectsGround(atmosphere, r, cos_theta);

        // 到地面的 distance 和 transmittance 只取决于 theta，
        // 所以我们为了效率我们在外侧循环中计算
        // The distance and transmittance to the ground only depend on theta, so we
        // can compute them in the outer loop for efficiency.
        Length distance_to_ground = 0.0 * m;
        DimensionlessSpectrum transmittance_to_ground = DimensionlessSpectrum(0.0);
        DimensionlessSpectrum ground_albedo = DimensionlessSpectrum(0.0);
        if (ray_r_theta_intersects_ground)
        {
            distance_to_ground =
              DistanceToBottomAtmosphereBoundary(atmosphere, r, cos_theta);
            transmittance_to_ground =
              GetTransmittance(atmosphere, transmittance_texture, r, cos_theta,
                  distance_to_ground, true /* ray_intersects_ground */);
            ground_albedo = atmosphere.ground_albedo;
        }

        for (int m = 0; m < 2 * SAMPLE_COUNT; ++m)
        {
            Angle phi = (Number(m) + 0.5) * dphi;
            vec3 omega_i =
            vec3(cos(phi) * sin_theta, sin(phi) * sin_theta, cos_theta);
            SolidAngle domega_i = (dtheta / rad) * (dphi / rad) * sin(theta) * sr;

            // 从方向\omega_i经过n-1次弹射的 radiance L_i 是
            // 以下列出项的和:
            // n-1阶预计算散射纹理项
            // The radiance L_i arriving from direction omega_i after n-1 bounces is
            // the sum of a term given by the precomputed scattering texture for the
            // (n-1)-th order:
            Number nu1 = dot(omega_s, omega_i);
            RadianceSpectrum incident_radiance = GetScattering(atmosphere,
            single_rayleigh_scattering_texture, single_mie_scattering_texture,
            multiple_scattering_texture, r, omega_i.z, mu_s, nu1,
            ray_r_theta_intersects_ground, scattering_order - 1);
			
            // n-1次弹射并且最后一次弹射在地面的光路贡献
            // 该贡献是到地面的transmittance，地面albedo，地面brdf和
			// 经过 n-2次弹射后接受的irradiance这几项的乘积
            // and of the contribution from the light paths with n-1 bounces and whose
            // last bounce is on the ground. This contribution is the product of the
            // transmittance to the ground, the ground albedo, the ground BRDF, and
            // the irradiance received on the ground after n-2 bounces.
            vec3 ground_normal =
              normalize(zenith_direction * r + omega_i * distance_to_ground);
            IrradianceSpectrum ground_irradiance = GetIrradiance(
            atmosphere, irradiance_texture, atmosphere.bottom_radius,
            dot(ground_normal, omega_s));
            incident_radiance += transmittance_to_ground *
            ground_albedo * (1.0 / (PI * sr)) * ground_irradiance;

            // 从方向omega_i被散射到方向-omega的radiance是
            // 接受的radiance，散射系数，方向omega和omega_i的相位函数这三项的乘积
            // (对所有的粒子类型求合，包括Rayleigh，Mie)
            // The radiance finally scattered from direction omega_i towards direction
            // -omega is the product of the incident radiance, the scattering
            // coefficient, and the phase function for directions omega and omega_i
            // (all this summed over all particle types, i.e. Rayleigh and Mie).
            Number nu2 = dot(omega, omega_i);
            Number rayleigh_density = GetProfileDensity(
            atmosphere.rayleigh_density, r - atmosphere.bottom_radius);
            Number mie_density = GetProfileDensity(
            atmosphere.mie_density, r - atmosphere.bottom_radius);
            rayleigh_mie += incident_radiance * (
            atmosphere.rayleigh_scattering * rayleigh_density *
                RayleighPhaseFunction(nu2) +
            atmosphere.mie_scattering * mie_density *
                MiePhaseFunction(atmosphere.mie_phase_function_g, nu2)) *
            domega_i;
        }
    }
    return rayleigh_mie;
}
~~~

>watt_per_cubic_meter_per_sr_per_nm 定义在 [Pysical units](https://ebruneton.github.io/precomputed_atmospheric_scattering/atmosphere/definitions.glsl.html) 中可以找到。

#### Second step

计算 n 阶散射的第二步是，针对每一点 $p$ 和方向 $\omega$，使用先前函数计算的纹理，计算经过 n 次弹射后来自方向 $\omega$ 的 radiance。

该 radiance 是下述内容的积分，  
在点 $p$ 和方向 $\omega$ 上到大气边界最近点之间所有点 $q$ 做下面的乘积：

* 由先前函数预计算的纹理提供的项——即来自任意方向经过 $n-1$ 次弹射，在点 $q$ 向点 $p$ 散射的 radiance
* $p,q$ 之间的 transmittance

注意，这里特意排除了经过 $n$ 次弹射且最后一次弹射发生在地面的光路。选择将这些路径排除在我们预计算纹理中是为了在实时渲染中用真实的地面 albedo 计算。

第二步实现过程：

~~~c++
RadianceSpectrum ComputeMultipleScattering(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    IN(ScatteringDensityTexture) scattering_density_texture,
    Length r, Number mu, Number mu_s, Number nu,
    bool ray_r_mu_intersects_ground)
{
    assert(r >= atmosphere.bottom_radius && r <= atmosphere.top_radius);
    assert(mu >= -1.0 && mu <= 1.0);
    assert(mu_s >= -1.0 && mu_s <= 1.0);
    assert(nu >= -1.0 && nu <= 1.0);

    // 数值积分的积分区间数量
    // Number of intervals for the numerical integration.
    const int SAMPLE_COUNT = 50;
    // 积分步进，即每个积分区间的长度
    // The integration step, i.e. the length of each integration interval.
    Length dx =
        DistanceToNearestAtmosphereBoundary(
            atmosphere, r, mu, ray_r_mu_intersects_ground) /
                Number(SAMPLE_COUNT);
    // 积分循环
    // Integration loop.
    RadianceSpectrum rayleigh_mie_sum =
        RadianceSpectrum(0.0 * watt_per_square_meter_per_sr_per_nm);
    for (int i = 0; i <= SAMPLE_COUNT; ++i)
    {
        Length d_i = Number(i) * dx;

        // r,mu,mu_s 变量在当前积分点
        // (详情请回顾之前的单次散射部分)
        // The r, mu and mu_s parameters at the current integration point (see the
        // single scattering section for a detailed explanation).
        Length r_i =
            ClampRadius(atmosphere, sqrt(d_i * d_i + 2.0 * r * mu * d_i + r * r));
        Number mu_i = ClampCosine((r * mu + d_i) / r_i);
        Number mu_s_i = ClampCosine((r * mu_s + d_i * nu) / r_i);
		// 在当前采样点的 Rayleigh 和 Mie 多重散射
        // The Rayleigh and Mie multiple scattering at the current sample point.
        RadianceSpectrum rayleigh_mie_i =
            GetScattering(
                atmosphere, scattering_density_texture, r_i, mu_i, mu_s_i, nu,
                ray_r_mu_intersects_ground) *
            GetTransmittance(
                atmosphere, transmittance_texture, r, mu, d_i,
                ray_r_mu_intersects_ground) *
            dx;
        // Sample weight (from the trapezoidal rule).
        Number weight_i = (i == 0 || i == SAMPLE_COUNT) ? 0.5 : 1.0;
        rayleigh_mie_sum += rayleigh_mie_i * weight_i;
    }
    return rayleigh_mie_sum;
}
~~~

### Precomputation

如多阶散射整体[计算逻辑](https://ebruneton.github.io/precomputed_atmospheric_scattering/atmosphere/functions.glsl.html#multiple_scattering)所述，我们需要预计算每阶散射结果到纹理中，以便在下一阶运算时节省运算量。并且为了将函数映射存进一张纹理，我们需要将函数参数映射到纹理坐标。  
幸运的是，所有阶的散射都依赖相同的参数组合$(r,\mu,\mu_s,\nu)$就和单阶散射的参数一样，因此我们可以复用单阶散射中定义的映射过程。这立刻引出了函数，用于预计算纹理中一个纹素，用于以反弹次数为维度的每次迭代中的 [first step](https://ebruneton.github.io/precomputed_atmospheric_scattering/atmosphere/functions.glsl.html#multiple_scattering_first_step) 和 [second step](https://ebruneton.github.io/precomputed_atmospheric_scattering/atmosphere/functions.glsl.html#multiple_scattering_second_step)：

~~~c++
RadianceDensitySpectrum ComputeScatteringDensityTexture(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    IN(ReducedScatteringTexture) single_rayleigh_scattering_texture,
    IN(ReducedScatteringTexture) single_mie_scattering_texture,
    IN(ScatteringTexture) multiple_scattering_texture,
    IN(IrradianceTexture) irradiance_texture,
    IN(vec3) frag_coord, int scattering_order) {
    Length r;
    Number mu;
    Number mu_s;
    Number nu;
    bool ray_r_mu_intersects_ground;
    // 从texcrood映射到物理参数
    GetRMuMuSNuFromScatteringTextureFragCoord(atmosphere, frag_coord,
      r, mu, mu_s, nu, ray_r_mu_intersects_ground);
	// First setp,计算被散射到大气中某一点 q
    // 朝着某一方向 -\omega 的 radiance
    return ComputeScatteringDensity(atmosphere, transmittance_texture,
      single_rayleigh_scattering_texture, single_mie_scattering_texture,
      multiple_scattering_texture, irradiance_texture, r, mu, mu_s, nu,
      scattering_order);
}

RadianceSpectrum ComputeMultipleScatteringTexture(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    IN(ScatteringDensityTexture) scattering_density_texture,
    IN(vec3) frag_coord, OUT(Number) nu) {
    Length r;
    Number mu;
    Number mu_s;
    bool ray_r_mu_intersects_ground;
    GetRMuMuSNuFromScatteringTextureFragCoord(atmosphere, frag_coord,
      r, mu, mu_s, nu, ray_r_mu_intersects_ground);
    return ComputeMultipleScattering(atmosphere, transmittance_texture,
      scattering_density_texture, r, mu, mu_s, nu,
      ray_r_mu_intersects_ground);
}
~~~

### Lookup

同样的，我们可以轻易复用在单阶散射中的查找函数 `GetScattering` ，用以从预计算纹理中读取多重散射的值。实际上，这正是我们在上述函数 `ComputeScatteringDensity` 和 `ComputeMultipleScattering` 中所做的。

## Ground irradiance

Ground irradiance 是经过 $n\ge0$ 次弹射后在地面接受的太阳光(这里一次弹射即是一次散射事件也可以是在地面的反射)。我们需要将其用在以下两个场景：

* 当预计算 $n$ 阶散射时，$n\ge2$，为了计算在地面上 $(n-1)$ 阶弹射光路的贡献。(这需要 $n-2$ 次弹射后地面的 irradiance，可以查看[Multiple scattering](https://ebruneton.github.io/precomputed_atmospheric_scattering/atmosphere/functions.glsl.html#multiple_scattering_computation) 小节)。
* 在渲染时，计算最后一次弹射在地面的光路贡献(根据定义，这些光路不会被预计算纹理覆盖)。

在第一种情况中，我们只需要大气底部水平表面的地面辐照度(在预计算时我们假设大地是拥有均匀 albedo 的完美球体)。  
在第二种情况中，我们需要任意高度和任意法线的地面辐照度，并且为了效率想对其进行预计算。实际上，就像我们在[论文](https://hal.inria.fr/inria-00288758/en)中描述的那样，我们仅针对任意海拔下的水平表面进行预计算该辐照度(这只需要2D texture，其余情况需要4D texture)，并且用近似方法处理非水平表面。

下面小节描述了如何预计算地面辐照度，如何存储，如何读取。

### Computation

地面辐照度的计算和，直接辐照度(从太阳直接接受的，没有任何反射的光)以及间接辐照度(至少有一次弹射)的计算是不一样的，我们先从直接辐照度开始。

irradiance 是 incident radiance 在半球上的积分，乘以余弦因子。针对地面的直接 irradiance，incident radiance 是大气顶部接受的的 Sun radiance，乘上穿过大气而衰减的 transmittance 系数。并且由于太阳的立体角非常小，可以认为该 transmittance 是一个常数，就是说我们可以将它移到 irradiance 积分外面，可以将积分的半球简化为(可见的)太阳圆盘。于是，积分等价于一个球体引起的环境光遮蔽，也叫角系数，详见[Radiative view factors](http://webserver.dmt.upm.es/~isidoro/tc3/Radiation%20View%20factors.pdf)(10页)。对于一个小立体角，这些复杂的等式可以被简化为：

~~~c++
IrradianceSpectrum ComputeDirectIrradiance(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    Length r, Number mu_s) {
    assert(r >= atmosphere.bottom_radius && r <= atmosphere.top_radius);
    assert(mu_s >= -1.0 && mu_s <= 1.0);

    Number alpha_s = atmosphere.sun_angular_radius / rad;
    // Approximate average of the cosine factor mu_s over the visible fraction of
    // the Sun disc.
    // （推导：对太阳圆盘上 cosθ 做积分，
    // 利用“小角度近似”和“圆盘均匀亮度”假设，化简得到此式。）
    Number average_cosine_factor =
    mu_s < -alpha_s ? 0.0 : (mu_s > alpha_s ? mu_s :
        (mu_s + alpha_s) * (mu_s + alpha_s) / (4.0 * alpha_s));

    return atmosphere.solar_irradiance *
      GetTransmittanceToTopAtmosphereBoundary(
          atmosphere, transmittance_texture, r, mu_s) * average_cosine_factor;

}
~~~

对于地面的 indirect irradiance(非直接辐照度)，在半球上的积分一定使用数值方法计算。准确来讲，我们需要计算在半球上所有方向 $\omega$ 上如下乘积的积分：

* 经过 $n$ 次弹射后来自方向 $\omega$ 到达的 radiance
* 余弦因子，比如 $\omega_z$

实现如下(这里如果 $n>1$，且 `scattering_order` 等于 $n$， `multiple_scattering_texture` 是包含 $n$ 阶散射的)：

~~~c++
IrradianceSpectrum ComputeIndirectIrradiance(
    IN(AtmosphereParameters) atmosphere,
    IN(ReducedScatteringTexture) single_rayleigh_scattering_texture,
    IN(ReducedScatteringTexture) single_mie_scattering_texture,
    IN(ScatteringTexture) multiple_scattering_texture,
    Length r, Number mu_s, int scattering_order) {
  assert(r >= atmosphere.bottom_radius && r <= atmosphere.top_radius);
  assert(mu_s >= -1.0 && mu_s <= 1.0);
  assert(scattering_order >= 1);

    const int SAMPLE_COUNT = 32;
    const Angle dphi = pi / Number(SAMPLE_COUNT);
    const Angle dtheta = pi / Number(SAMPLE_COUNT);

    IrradianceSpectrum result =
    	IrradianceSpectrum(0.0 * watt_per_square_meter_per_nm);
    vec3 omega_s = vec3(sqrt(1.0 - mu_s * mu_s), 0.0, mu_s);
    for (int j = 0; j < SAMPLE_COUNT / 2; ++j) {
    	Angle theta = (Number(j) + 0.5) * dtheta;
    	for (int i = 0; i < 2 * SAMPLE_COUNT; ++i) {
      	Angle phi = (Number(i) + 0.5) * dphi;
      	vec3 omega =
        	vec3(cos(phi) * sin(theta), sin(phi) * sin(theta), cos(theta));
      	SolidAngle domega = (dtheta / rad) * (dphi / rad) * sin(theta) * sr;

      	Number nu = dot(omega, omega_s);
      	result += GetScattering(atmosphere,
            single_rayleigh_scattering_texture,
          	single_mie_scattering_texture, multiple_scattering_texture,
          	r, omega.z, mu_s, nu, false /* ray_r_theta_intersects_ground */,
          	scattering_order) * omega.z * domega;
    	}
    }
    return result;
}
~~~

### Precomputation

为了预计算纹理中的 ground irradiance，需要从 ground irradiance 参数映射到纹理坐标。因为只预计算水平表面的 ground irradiance，所以 irradiance 取决于 $r,\mu_s$，所以要从 $(r,\mu_s)$ 映射到纹理坐标 $(u,v)$。最简单的仿射变换就足够了，因为 ground irradiance 足够平滑：

~~~c++
vec2 GetIrradianceTextureUvFromRMuS(IN(AtmosphereParameters) atmosphere,
    Length r, Number mu_s) {
    assert(r >= atmosphere.bottom_radius && r <= atmosphere.top_radius);
    assert(mu_s >= -1.0 && mu_s <= 1.0);
    Number x_r = (r - atmosphere.bottom_radius) /
      (atmosphere.top_radius - atmosphere.bottom_radius);
    Number x_mu_s = mu_s * 0.5 + 0.5;
    return vec2(GetTextureCoordFromUnitRange(x_mu_s, IRRADIANCE_TEXTURE_WIDTH),
              GetTextureCoordFromUnitRange(x_r, IRRADIANCE_TEXTURE_HEIGHT));
}
~~~

逆映射：

~~~c++
void GetRMuSFromIrradianceTextureUv(IN(AtmosphereParameters) atmosphere,
    IN(vec2) uv, OUT(Length) r, OUT(Number) mu_s) {
    assert(uv.x >= 0.0 && uv.x <= 1.0);
    assert(uv.y >= 0.0 && uv.y <= 1.0);
    Number x_mu_s = GetUnitRangeFromTextureCoord(uv.x, IRRADIANCE_TEXTURE_WIDTH);
    Number x_r = GetUnitRangeFromTextureCoord(uv.y, IRRADIANCE_TEXTURE_HEIGHT);
    r = atmosphere.bottom_radius +
      x_r * (atmosphere.top_radius - atmosphere.bottom_radius);
    mu_s = ClampCosine(2.0 * x_mu_s - 1.0);
}
~~~

现在定义函数用以预计算一个纹素的 ground irradiance，关于 direct irradiance 部分：

~~~c++
const vec2 IRRADIANCE_TEXTURE_SIZE =
    vec2(IRRADIANCE_TEXTURE_WIDTH, IRRADIANCE_TEXTURE_HEIGHT);

IrradianceSpectrum ComputeDirectIrradianceTexture(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    IN(vec2) frag_coord) {
  Length r;
  Number mu_s;
  GetRMuSFromIrradianceTextureUv(
      atmosphere, frag_coord / IRRADIANCE_TEXTURE_SIZE, r, mu_s);
  return ComputeDirectIrradiance(atmosphere, transmittance_texture, r, mu_s);
}
~~~

indirect irradiance 部分：

~~~c++
IrradianceSpectrum ComputeIndirectIrradianceTexture(
    IN(AtmosphereParameters) atmosphere,
    IN(ReducedScatteringTexture) single_rayleigh_scattering_texture,
    IN(ReducedScatteringTexture) single_mie_scattering_texture,
    IN(ScatteringTexture) multiple_scattering_texture,
    IN(vec2) frag_coord, int scattering_order) {
  Length r;
  Number mu_s;
  GetRMuSFromIrradianceTextureUv(
      atmosphere, frag_coord / IRRADIANCE_TEXTURE_SIZE, r, mu_s);
  return ComputeIndirectIrradiance(atmosphere,
      single_rayleigh_scattering_texture, single_mie_scattering_texture,
      multiple_scattering_texture, r, mu_s, scattering_order);
}
~~~

### Lookup

使用一个 lookup 查找 groud irradiance： 

~~~c++
IrradianceSpectrum GetIrradiance(
    IN(AtmosphereParameters) atmosphere,
    IN(IrradianceTexture) irradiance_texture,
    Length r, Number mu_s) {
  vec2 uv = GetIrradianceTextureUvFromRMuS(atmosphere, r, mu_s);
  return IrradianceSpectrum(texture(irradiance_texture, uv));
}
~~~

## Rendering

假设我们已经预计算过 transmittance，scattering 和 irradiance texture，并且实现了使用这些纹理计算天空颜色，空中透视和 ground radiance 的函数。  
更准确地说，我们假设单瑞利散射，不含相位函数，加上多个散射项(除以瑞利散射的相位函数保持量纲一致)存储在 `scattering_texture`。我们也假设单米氏散射不含相位方程被存储：

* 要么单独的存储在 `single_mie_scattering_texture` 中(我们[最初的实现](http://evasion.inrialpes.fr/~Eric.Bruneton/PrecomputedAtmosphericScattering2.zip)没有提供这个选项。)
* 或者，如果定义了 `COMBINED_SCATTERING_TEXTURE` 预处理宏，存储在 `scattering_texture` 中。这种情况下，只能使用 GLSL 编辑器，Rayleigh 散射和多重散射被存储在 RGB 通道，单 Mie 散射的红色分量被存储在 alpha 通道。

在第二中情况中，单 Mie 散射的绿色和蓝色通道进行[论文](https://hal.inria.fr/inria-00288758/en)中所描述的外推，其函数如下：

~~~c++
#ifdef COMBINED_SCATTERING_TEXTURES
vec3 GetExtrapolatedSingleMieScattering(
    IN(AtmosphereParameters) atmosphere, IN(vec4) scattering) {
  // Algebraically this can never be negative, but rounding errors can produce
  // that effect for sufficiently short view rays.
  if (scattering.r <= 0.0) {
    return vec3(0.0);
  }
  return scattering.rgb * scattering.a / scattering.r *
	    (atmosphere.rayleigh_scattering.r / atmosphere.mie_scattering.r) *
	    (atmosphere.mie_scattering / atmosphere.rayleigh_scattering);
}
#endif
~~~

$$
\begin{aligned}
&\frac{S_{rgb}S_a}{S_r}*
\frac{A_R.r}{A_M.r}*
\frac{A_M}{A_R}\\
&\frac{S_a}{S_r}:Mie散射和Rayleigh散射r通道比\\
&\frac{A_R.r}{A_M.r}:Rayleigh和Mie散射系数的r通道比\\
&\frac{A_M}{A_S}:Mie和Rayleigh的散射系数比\\
&假设散射的结果比值也满足散射系数的比例\\
&(并用结果的r通道做比值修正)，\\
&进而外推Mie散射的结果
\end{aligned}
$$

然后我们可以检索所有的散射分量(一部分是Rayleigh + Multiple Scattering，另一部分是单 Mie Scattering)使用基于 [`GetScattering`](https://ebruneton.github.io/precomputed_atmospheric_scattering/atmosphere/functions.glsl.html#single_scattering_lookup) 的函数(在这里使用一些重复代码，而不是调用两次 `GetScattering`，以确保纹理坐标在 `scattering_texture` 和 `single_mie_scattering_texture` 的查询之间共享)：

~~~c++
IrradianceSpectrum GetCombinedScattering(
    IN(AtmosphereParameters) atmosphere,
    IN(ReducedScatteringTexture) scattering_texture,
    IN(ReducedScatteringTexture) single_mie_scattering_texture,
    Length r, Number mu, Number mu_s, Number nu,
    bool ray_r_mu_intersects_ground,
    OUT(IrradianceSpectrum) single_mie_scattering) {
  vec4 uvwz = GetScatteringTextureUvwzFromRMuMuSNu(
      atmosphere, r, mu, mu_s, nu, ray_r_mu_intersects_ground);
  Number tex_coord_x = uvwz.x * Number(SCATTERING_TEXTURE_NU_SIZE - 1);
  Number tex_x = floor(tex_coord_x);
  Number lerp = tex_coord_x - tex_x;
  vec3 uvw0 = vec3((tex_x + uvwz.y) / Number(SCATTERING_TEXTURE_NU_SIZE),
      uvwz.z, uvwz.w);
  vec3 uvw1 = vec3((tex_x + 1.0 + uvwz.y) / Number(SCATTERING_TEXTURE_NU_SIZE),
      uvwz.z, uvwz.w);
#ifdef COMBINED_SCATTERING_TEXTURES
  vec4 combined_scattering =
      texture(scattering_texture, uvw0) * (1.0 - lerp) +
      texture(scattering_texture, uvw1) * lerp;
  IrradianceSpectrum scattering = IrradianceSpectrum(combined_scattering);
  single_mie_scattering =
      GetExtrapolatedSingleMieScattering(atmosphere, combined_scattering);
#else
  IrradianceSpectrum scattering = IrradianceSpectrum(
      texture(scattering_texture, uvw0) * (1.0 - lerp) +
      texture(scattering_texture, uvw1) * lerp);
  single_mie_scattering = IrradianceSpectrum(
      texture(single_mie_scattering_texture, uvw0) * (1.0 - lerp) +
      texture(single_mie_scattering_texture, uvw1) * lerp);
#endif
  return scattering;
}
~~~

### Sky

为了渲染天空，我们只需要显示天空的 radiance，该 radiance 可以通过预计算的散射纹理，乘上预计算中省略的相位函数得到。我们也能返回大气的 transmittance(我们可以在预计算的 transmittance 纹理中进行一次 lookup获得)，这需要正确地渲染太空中的物体(比如太阳和月亮)。这导致了下面的函数其中大部分计算用于正确处理观察者在大气层外的情况，以及光线穿透的情况：

~~~c++
RadianceSpectrum GetSkyRadiance(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    IN(ReducedScatteringTexture) scattering_texture,
    IN(ReducedScatteringTexture) single_mie_scattering_texture,
    Position camera, IN(Direction) view_ray, Length shadow_length,
    IN(Direction) sun_direction, OUT(DimensionlessSpectrum) transmittance) {
    // Compute the distance to the top atmosphere boundary along the view ray,
    // assuming the viewer is in space (or NaN if the view ray does not intersect
    // the atmosphere).
    Length r = length(camera);
    Length rmu = dot(camera, view_ray);
    Length distance_to_top_atmosphere_boundary = -rmu -
      sqrt(rmu * rmu - r * r + atmosphere.top_radius * atmosphere.top_radius);
    // If the viewer is in space and the view ray intersects the atmosphere, move
    // the viewer to the top atmosphere boundary (along the view ray):
    if (distance_to_top_atmosphere_boundary > 0.0 * m)
    {
        camera = camera + view_ray * distance_to_top_atmosphere_boundary;
        r = atmosphere.top_radius;
        rmu += distance_to_top_atmosphere_boundary;
    }
    else if (r > atmosphere.top_radius)
    {
        // If the view ray does not intersect the atmosphere,
        // simply return 0.
        transmittance = DimensionlessSpectrum(1.0);
        return RadianceSpectrum(0.0 * watt_per_square_meter_per_sr_per_nm);
    }
    // Compute the r, mu, mu_s and nu parameters
    // needed for the texture lookups.
    Number mu = rmu / r;
    Number mu_s = dot(camera, sun_direction) / r;
    Number nu = dot(view_ray, sun_direction);
    bool ray_r_mu_intersects_ground =
        RayIntersectsGround(atmosphere, r, mu);

    transmittance = ray_r_mu_intersects_ground ?
        DimensionlessSpectrum(0.0) :
      GetTransmittanceToTopAtmosphereBoundary(
          atmosphere, transmittance_texture, r, mu);
    IrradianceSpectrum single_mie_scattering;
    IrradianceSpectrum scattering;
    if (shadow_length == 0.0 * m)
    {
        scattering = GetCombinedScattering(
            atmosphere, scattering_texture, single_mie_scattering_texture,
            r, mu, mu_s, nu, ray_r_mu_intersects_ground,
            single_mie_scattering);
    }
    else
    {
        // Case of light shafts (shadow_length is
        // the total length noted l in our
        // paper): we omit the scattering between the camera and the point at
        // distance l, by implementing 
        // Eq. (18) of the paper (shadow_transmittance
        // is the T(x,x_s) term, scattering is the S|x_s=x+lv term).
        Length d = shadow_length;
        Length r_p =
            ClampRadius(atmosphere, sqrt(d * d + 2.0 * r * mu * d + r * r));
        Number mu_p = (r * mu + d) / r_p;
        Number mu_s_p = (r * mu_s + d * nu) / r_p;

        scattering = GetCombinedScattering(
            atmosphere, scattering_texture, single_mie_scattering_texture,
            r_p, mu_p, mu_s_p, nu, ray_r_mu_intersects_ground,
            single_mie_scattering);
        DimensionlessSpectrum shadow_transmittance =
            GetTransmittance(atmosphere, transmittance_texture,
                r, mu, shadow_length, ray_r_mu_intersects_ground);
        scattering = scattering * shadow_transmittance;
        single_mie_scattering = single_mie_scattering * shadow_transmittance;
    }
    return scattering * RayleighPhaseFunction(nu) + single_mie_scattering *
      MiePhaseFunction(atmosphere.mie_phase_function_g, nu);
}
~~~

### Aerial perspective

为了渲染空中透视，我们需要两点之间的 transmittance 和 散射量(例如观察者和地面上一点，该点可以在任意海拔高度)。我们已经拥有计算两点之间 transmittance 的函数(在预计算的纹理上进行两次查找，该纹理即只包含到大气顶部 transmittance 的那张)，但是我们没有能够计算两点之间 散射量的函数。幸运的是，两点之间的散射量可以通过两次纹理查找计算得来，就像计算 transmittance 一样(和用除法计算 transmittance 不同，这里需要使用减法)，该纹理需要存储着距离大气边界最近点的散射量。这就是我们下面函数所实现的(函数开头的计算用于正确处理“观测者位于大气层外”的场景)：

~~~c++
RadianceSpectrum GetSkyRadianceToPoint(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    IN(ReducedScatteringTexture) scattering_texture,
    IN(ReducedScatteringTexture) single_mie_scattering_texture,
    Position camera, IN(Position) point, Length shadow_length,
    IN(Direction) sun_direction, OUT(DimensionlessSpectrum) transmittance) {
    // Compute the distance to the top atmosphere boundary along the view ray,
    // assuming the viewer is in space (or NaN if the view ray does not intersect
    // the atmosphere).
    Direction view_ray = normalize(point - camera);
    Length r = length(camera);
    Length rmu = dot(camera, view_ray);
    Length distance_to_top_atmosphere_boundary = -rmu -
      sqrt(rmu * rmu - r * r + atmosphere.top_radius * atmosphere.top_radius);
    // If the viewer is in space and the view ray intersects the atmosphere, move
    // the viewer to the top atmosphere boundary (along the view ray):
    if (distance_to_top_atmosphere_boundary > 0.0 * m)
    {
        camera = camera + view_ray * distance_to_top_atmosphere_boundary;
        r = atmosphere.top_radius;
        rmu += distance_to_top_atmosphere_boundary;
    }

    // Compute the r, mu, mu_s and nu parameters for the first texture lookup.
    Number mu = rmu / r;
    Number mu_s = dot(camera, sun_direction) / r;
    Number nu = dot(view_ray, sun_direction);
    Length d = length(point - camera);
    bool ray_r_mu_intersects_ground = RayIntersectsGround(atmosphere, r, mu);

    transmittance = GetTransmittance(atmosphere, transmittance_texture,
      r, mu, d, ray_r_mu_intersects_ground);

    IrradianceSpectrum single_mie_scattering;
    IrradianceSpectrum scattering = GetCombinedScattering(
      atmosphere, scattering_texture, single_mie_scattering_texture,
      r, mu, mu_s, nu, ray_r_mu_intersects_ground,
      single_mie_scattering);

    // Compute the r, mu, mu_s and nu parameters
    // for the second texture lookup.
    // If shadow_length is not 0 (case of light shafts),
    // we want to ignore the
    // scattering along the last shadow_length meters of the view ray,
    // which we do by subtracting shadow_length from d (this way scattering_p is equal to
    // the S|x_s=x_0-lv term in Eq. (17) of our paper).
    d = max(d - shadow_length, 0.0 * m);
    Length r_p = ClampRadius(atmosphere, sqrt(d * d + 2.0 * r * mu * d + r * r));
    Number mu_p = (r * mu + d) / r_p;
    Number mu_s_p = (r * mu_s + d * nu) / r_p;

    IrradianceSpectrum single_mie_scattering_p;
    IrradianceSpectrum scattering_p = GetCombinedScattering(
      atmosphere, scattering_texture, single_mie_scattering_texture,
      r_p, mu_p, mu_s_p, nu, ray_r_mu_intersects_ground,
      single_mie_scattering_p);

    // Combine the lookup results to
    // get the scattering between camera and point.
    DimensionlessSpectrum shadow_transmittance = transmittance;
    if (shadow_length > 0.0 * m)
    {
        // This is the T(x,x_s) term in 
        // Eq. (17) of our paper, for light shafts.
        shadow_transmittance = GetTransmittance(atmosphere, transmittance_texture,
            r, mu, d, ray_r_mu_intersects_ground);
    }
    scattering = scattering - shadow_transmittance * scattering_p;
    single_mie_scattering =
      single_mie_scattering - shadow_transmittance * single_mie_scattering_p;
    #ifdef COMBINED_SCATTERING_TEXTURES
    single_mie_scattering = GetExtrapolatedSingleMieScattering(
      atmosphere, vec4(scattering, single_mie_scattering.r));
    #endif

    // Hack to avoid rendering artifacts when the sun is below the horizon.
    single_mie_scattering = single_mie_scattering *
      smoothstep(Number(0.0), Number(0.01), mu_s);

    return scattering * RayleighPhaseFunction(nu) + single_mie_scattering *
      MiePhaseFunction(atmosphere.mie_phase_function_g, nu);
}
~~~

### Ground

为了渲染地面，我们需要到达地面的 irradiance，这些 irradiance 在大气中或在地面上经过 0 次或者更多次弹射。direct irradiance 在 transmittance 纹理中进行一次查找就能够计算，通过 `GetTransmittanceToSun` 函数，而 indirect irradiance 通过查找预计算 irradiance 纹理得到(该纹理只包含水平表面的 irradiance；对于其他情况我们使用定义在[论文](https://hal.inria.fr/inria-00288758/en)中的近似)。下面函数分别返回 direct 和 indirect irradiance：

~~~c++
IrradianceSpectrum GetSunAndSkyIrradiance(
    IN(AtmosphereParameters) atmosphere,
    IN(TransmittanceTexture) transmittance_texture,
    IN(IrradianceTexture) irradiance_texture,
    IN(Position) point, IN(Direction) normal, IN(Direction) sun_direction,
    OUT(IrradianceSpectrum) sky_irradiance) {
    Length r = length(point);
    Number mu_s = dot(point, sun_direction) / r;

    // Indirect irradiance (approximated if the surface is not horizontal).
    sky_irradiance = GetIrradiance(atmosphere, irradiance_texture, r, mu_s) *
      (1.0 + dot(normal, point) / r) * 0.5;

    // Direct irradiance.
    return atmosphere.solar_irradiance *
      GetTransmittanceToSun(
          atmosphere, transmittance_texture, r, mu_s) *
      max(dot(normal, sun_direction), 0.0);
}
~~~

## Definitions

### Atmosphere parameters

~~~c++
struct AtmosphereParameters {
  // 大气层顶部的太阳辐照度。
  IrradianceSpectrum solar_irradiance;
  // 太阳的角半径。警告：该实现使用的近似值仅在太阳角半径小于0.1弧度时有效。
  Angle sun_angular_radius;
  // 行星中心到大气层底部的距离。
  Length bottom_radius;
  // 行星中心到大气层顶部的距离。
  Length top_radius;
  // 空气分子的密度分布曲线，即海拔高度到无量纲值（0代表无密度，1代表最大密度）的函数。
  DensityProfile rayleigh_density;
  // 在空气分子密度最大高度（通常为大气层底部）处的瑞利散射系数，以波长函数表示。
  // 高度h处的散射系数 = rayleigh_scattering × 该高度对应的rayleigh_density值。
  ScatteringSpectrum rayleigh_scattering;
  // 气溶胶的密度分布曲线，即海拔高度到无量纲值（0代表无密度，1代表最大密度）的函数。
  DensityProfile mie_density;
  // 在气溶胶密度最大高度（通常为大气层底部）处的气溶胶散射系数，以波长函数表示。
  // 高度h处的散射系数 = mie_scattering × 该高度对应的mie_density值。
  ScatteringSpectrum mie_scattering;
  // 在气溶胶密度最大高度（通常为大气层底部）处的气溶胶消光系数，以波长函数表示。
  // 高度h处的消光系数 = mie_extinction × 该高度对应的mie_density值。
  ScatteringSpectrum mie_extinction;
  // Cornette-Shanks相位函数中气溶胶的各向异性因子。
  Number mie_phase_function_g;
  // 吸收光线的空气分子（如臭氧）的密度分布曲线，即海拔高度到无量纲值（0代表无密度，1代表最大密度）的函数。
  DensityProfile absorption_density;
  // 在吸收光线的分子（如臭氧）密度最大高度处的消光系数，以波长函数表示。
  // 高度h处的消光系数 = absorption_extinction × 该高度对应的absorption_density值。
  ScatteringSpectrum absorption_extinction;
  // 地面的平均反照率。
  DimensionlessSpectrum ground_albedo;
  // 需预先计算大气散射的最大太阳天顶角余弦值（为达到最高精度，应取可忽略天空光辐射的最小太阳天顶角。例如地球案例中取102度，对应mu_s_min = -0.2）。
  Number mu_s_min;
};
~~~



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

[[Elek09] 实时渲染参数化的、有多次散射的行星大气 - 知乎](https://zhuanlan.zhihu.com/p/345916725)

[ebruneton/precomputed_atmospheric_scattering: This project provides a new implementation of our EGSR 2008 paper "Precomputed Atmospheric Scattering".](https://github.com/ebruneton/precomputed_atmospheric_scattering)  
[Precomputed Atmospheric Scattering](https://inria.hal.science/file/index/docid/288758/filename/article.pdf)

翻译来源：[ebruneton.github.io/precomputed_atmospheric_scattering/atmosphere/functions.glsl.html](https://ebruneton.github.io/precomputed_atmospheric_scattering/atmosphere/functions.glsl.html)  
[Volumetric Atmospheric Scattering - Alan Zucconi](https://www.alanzucconi.com/2017/10/10/atmospheric-scattering-1/)

* Next Epic 的 Atmosphere Scattering  
  [Publications](https://sebh.github.io/publications/)  
  [Production Ready Atmosphere Rendering](https://sebh.github.io/publications/egsr2020.pdf)
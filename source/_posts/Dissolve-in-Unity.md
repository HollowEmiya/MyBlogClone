---
title: Dissolve in Unity
date: 2022-04-09 12:35:18
tags:
---

# Dissolve in Unity

参考冯乐乐所著《Unity Shader 入门精要》在unity里实现简单的消融(Dissolve)效果，主要思想是利用噪声纹理，进行透明度剔除。(图片在GitHub加载失败可能是网络问题)

## 结果预览

![Alt pic](https://raw.githubusercontent.com/HollowEmiya/DissolveShader/main/DissolveResult.png)

<!--more-->

## 思路

我们先思考我们所需的消融效果是什么样的，需要物体被部分烧毁的痕迹，物体会被烧得镂空，燃烧的边缘有着颜色变化，大体而言就是这样。所以我们只需要利用噪声纹理，对物体表面进行依据透明度的剔除(Clip)来模拟镂空就好，再根据由噪声纹理得到的**透明度**和我们所定义**消融系数**对颜色进行插值就可以模拟边缘烧焦的效果(因为透明度更高的部分被clip了，所以我们这样简单的模拟就能得到很好的效果)。

## Word is weak, show me your code

<div align="center">
<img title="" src="https://raw.githubusercontent.com/HollowEmiya/DissolveShader/main/DissolveMat.png" alt=""></div>

<center>Shader 面版</center>

    Properties
        {
            _BurnAmount ("Burn Amount", Range(0.0, 1.0)) = 0.0
            _LineWidth ("Burn Line Width", Range(0.0, 0.2)) = 0.1
            _MainTex ("Base (RGB)", 2D) = "white" { }
            _BumpMap ("Normal Map", 2D) = "bump" { }
            _BurnFirstColor ("Burn First Color", Color) = (1, 0, 0, 1)
            _BurnSecondColor ("Burn Second Color", Color) = (1, 0, 0, 1)
            _BurnMap ("Burn Map", 2D) = "white" { }
        }

我们首先需要定义所要用到的消融系数`_BurnAmount`，控制消融效果边缘宽度的`_LineWidth`，物体的主纹理、法线纹理、噪声纹理，以及用以模拟烧焦边缘的两个颜色`FirstColor`,`SecondColor`……

### Pass

然后来看第一个Pass部分，由于我们需要镂空效果所以我们要关闭Cull，来保证背面的片元也被成功渲染，渲染模式设置为普通的前向渲染即可。

    Tags {“LightMode” = “ForwardBase”}
    Cull Off

### 顶点着色器

关于顶点着色器，我们计算好视体空间坐标、所用的纹理坐标、世界坐标及切线下的光线方向，填充v2f即可。

    v2f vert(a2v v) {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
    
                o.uvMainTex = TRANSFORM_TEX(v.texcoord, _MainTex);
                o.uvBumpMap = TRANSFORM_TEX(v.texcoord, _BumpMap);
                o.uvBurnMap = TRANSFORM_TEX(v.texcoord, _BurnMap);
    
                TANGENT_SPACE_ROTATION;
                o.lightDir = mul(rotation, ObjSpaceLightDir(v.vertex)).xyz;
    
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
    
                TRANSFER_SHADOW(o);
    
                return o;
            }

因为我们使用了法线纹理，所以需要利用切线空间下的光照进行漫反射的计算。

### 片元着色器

接下来是重点的片元着色器

    fixed4 frag(v2f i) : SV_Target {
                fixed3 burn = tex2D(_BurnMap, i.uvBurnMap).rgb;
    
                clip(burn.r - _BurnAmount);
    
                float3 tangentLightDir = normalize(i.lightDir);
                fixed3 tangentNormal = UnpackNormal(tex2D(_BumpMap, i.uvBumpMap));
    
                fixed3 albedo = tex2D(_MainTex, i.uvMainTex).rgb;
    
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
    
                fixed3 diffuse = _LightColor0.rgb * albedo * saturate(dot(tangentLightDir, tangentNormal));
    
                fixed t = 1 - smoothstep(0.0, _LineWidth, burn.r - _BurnAmount);
    
                fixed3 burnColor = lerp(_BurnFirstColor, _BurnSecondColor, t);
                burnColor = pow(burnColor, 5);
    
                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);
                fixed3 finalColor = lerp(ambient + diffuse * atten, burnColor, t * step(0.0001, _BurnAmount));
    
                return fixed4(finalColor, 1);
            }

可以看到我们先对噪声纹理进行采样，之后运用clip函数(剔除传入参数小于0的片元)根据纹理结果和我们设置的消融系数来进行片元剔除。

利用smoothstep函数计算系数t，smoothstep是根据第三个参数对第一、二个参数空间进行平滑过渡，区间左侧为零，右侧为1，区间做过度。

系数t越大越接近消融部分，越小越远离消融部分，t为0即像素显示正常颜色，t为1表明像素在燃烧边界。

根据t对FirstColor和SecondColor来进行lerp插值，模拟边缘的烧焦效果，因为“烧毁的部分”已经被clip掉了，所以我们的模拟不用担心透明度大于消融系数的部分。

同时使用pow函数对burnColor处理，使其更真实。

再用系数t对光照效果(ambient+diffuse)和烧焦颜色插值，这里使用一个step函数保证我们消融系数为0时，可以正常返回物体本身的颜色。

到这里其实主要部分已经完成了，不过我们还需要处理阴影，因为我们使用了透明效果，只要涉及到透明效果我们就一定要对阴影十分留意，如果不做处理，阴影还依照原本物体的样子投射阴影那么我们的模拟就穿帮了，这肯定不是我们想看到的，还好利用Unity我们不需要太复杂的处理。

### 关于阴影

我们需要做的就是把我们在消融模拟时剔除的片元，“真正的剔除”。也就是真的把物体烧了(笑。

开玩笑的，其实还是依照纹理采样来的透明度在生成阴影的pass剔除对应的片元。

先设置Pass的渲染模式

    Tags { "LightMode"="ShadowCaster" }

#### 顶点着色器

顶点着色器及v2f结构体

    v2f vert(appdata_base v) {
                    v2f o;
    
                    TRANSFER_SHADOW_CASTER_NORMALOFFSET(o);
    
                    o.uvBurnMap = TRANSFORM_TEX(v.texcoord, _BurnMap);
    
                    return o;
                }

#### 片元着色器

片元着色器

    fixed4 frag(v2f i) : SV_Target {
                    fixed3 burn = tex2D(_BurnMap, i.uvBurnMap).rgb;
    
                    clip(burn.r - _BurnAmount);
    
                    SHADOW_CASTER_FRAGMENT(i);
                }

clip用法和消融模拟中一样，而 V2F_SHADOW_CASTER、TRANSFER_SHADOW_CASTER_NORMALOFFSET、SHADOW_CASTER_FRAGMENT 是老常客了，投射阴影重点就在于我们需要按照正常模拟那样对片元进行处理(是保留？还是剔除？)，利用Unity提供的这三个老常客我们可以省去很多很多的工作。

在结构体 v2f 中 V2F_SHADOW CASTER 来定义阴影投射需要定义的变量，在顶点着
色器中，使用 TRANSFER_SHADOW_ CASTER _NORMALOFFSET 来填充 V2F_SHADOW_ 
ASTER在背后声明的变量，这是Unity帮我们完成的。。在片元着
色器中，SHADOW_ CASTER_FRAGMENT 来让 Unity 为我们完成阴影投射的部分，把结果输出到深度图和阴影映射纹理中。

注意，在我们使用Unity的内置宏时，我们的命名以及变量使用需要符合Unity的规范，例如， TRANSFER_SHADOW_ CASTER_NORMALOFFSET 会使用名称`v`作为输入结构体，v需要包含顶点位置 v.vertex 和顶点法线 v.normal，这些变量的appdata_base包含的，而当我们需要顶点动画时，可以在顶点着色器修改vertex，再传递给TRANSFER_SHADOW_ CASTER_NORMALOFFSET，这是可以的。

最后，也可以编写一个脚本根据时间修改Mat的_BurnAmount属性，来模拟基于时间的消融效果。

<a href="https://github.com/HollowEmiya/DissolveShader" target="_blank">代码地址</a>

[演示视频](https://www.bilibili.com/video/BV1BY4y1i7a8?spm_id_from=333.999.0.0)

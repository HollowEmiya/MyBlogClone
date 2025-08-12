---
title: 关于 SRP Batcher
math: true
tags: [Unity,渲染]
index_img: /imgs/Hexo主题变更/Shiki&Tsukihime.png
banner_img: /imgs/Hexo主题变更/Shiki&Tsukihime.png
date: 2025-02-22 14:05:36
typora-root-url: ../
---

关于 SRP Batcher 的理解和内容

# 关于 SRP Batcher

## Shader 和 Material

首先我们得对 Shader 和 Material 以及渲染过程有一个基础的了解。

* Shader：运行在 **GPU** 上的一段程序，
  将输入的 Mesh 和 Texture 以及 其他 Properties 进行计算，然后输出结果。
  <font color= #999999>当然也有利用 GPU 并行能力计算的 Computer Shader 主要是做数据计算和处理，比如计算 LUT，或者滤波、图像处理等等，这个不是本文重点。</font>
* Material：描述(存储) 模型/物体 表面细节信息的资源。
  定义外观属性 Color，Tex，Smoothness……可以看作是对 Shader的实例。

材质可以看作是一个配置Shader属性数值的面板。通过修改材质球里的属性数值，实际上就是在修改Shader的属性数值。
当我们创建一个材质时，我们实际上是在创建一个Shader的实例。这个实例包含了Shader的所有属性，并且我们可以为这些属性提供具体的值。
在Unity中，每个材质都关联了一个Shader。当我们将材质应用到一个物体上时，实际上是在告诉Unity使用这个材质所关联的Shader来渲染这个物体。

## 简单的渲染过程

就现在的设计，简单来讲渲染物体主要分为四个步骤：
暂时用 DX 举例吧，我记不得太多了……

1. **初始化**：

   * **创建设备** (ID3D11Device) 和 **设备上下文**(ID3D11DeviceContext)，以及交换链管理 Front Buffer 和 Back Buffer

     创建渲染视图并绑定到输出合并阶段。

2. **设置渲染状态**：

   * **定义顶点格式和输入布局**：根据需要渲染的几何体定义顶点结构，并创建相应的输入布局（`ID3D11InputLayout`）描述顶点数据的格式和语义。
   * **创建顶点和索引缓冲区**：将几何体的顶点数据存储在顶点缓冲区（`ID3D11Buffer`）中，如果有重复顶点，可以使用索引缓冲区（`ID3D11Buffer`）来优化内存使用和渲染效率。
   * **创建着色器**：编写顶点着色器（Vertex Shader）和像素着色器（Pixel Shader），并使用`D3DCompile`函数编译成字节码，然后创建着色器对象（`ID3D11VertexShader`和`ID3D11PixelShader`）

3. **绘制调用**：

   - **绑定着色器和常量缓冲区**：将顶点着色器、像素着色器和常量缓冲区绑定到渲染管线的相应阶段。
   - **设置图元拓扑**：指定要渲染的图元类型，如三角形列表、线列表等。
   - **设置混合状态、深度/模板状态等**：根据需要设置渲染管线的其他状态，如混合模式、深度测试和模板测试等。
   - (可选)进行ClearRenderTargetView
   - **绘制几何体**：使用`DrawIndexed`或`Draw`函数绘制几何体，根据索引缓冲区（如果有）或顶点缓冲区中的数据生成图元，
   - 重复渲染过程。

4. **结果展示**：

   前后帧交替显示渲染结果

###  一点关于dx的概念，不是很重要……

* `ID3D11Device`接口，它用于创建资源和枚举显示适配器的功能，用以应用程序直接与图形硬件进行交互。

* 设备上下文[^设备上下文]是设备的一个接口，用于设置管道状态、将资源绑定到图形管线和生成渲染命令。
  设备上下文在`ID3D11DeviceContext`接口中实现，它提供了对图形管线的直接控制，允许应用程序执行各种渲染操作。
  * **即时上下文（Immediate Context）**：即时上下文直接与图形硬件进行交互，它允许应用程序立即执行渲染命令。每个设备都有一个且只有一个即时上下文。
  * **延迟上下文（Deferred Context）**：延迟上下文将渲染命令记录到命令列表中，它主要用于多线程渲染。延迟上下文可以由工作线程使用，而即时上下文通常由主线程使用

### Draw Call

**Draw Call** 是 CPU 调用图形 API（如 OpenGL 或 DirectX）命令 GPU 进行渲染的操作。具体来说，Draw Call 是 CPU 向 GPU 发送的绘制命令，告诉 GPU 如何渲染场景的一部分，包括使用哪些顶点、纹理、着色器等。

### Draw Call 的工作原理

1. **命令缓冲区**：为了实现 CPU 和 GPU 的并行工作，需要一个命令缓冲区（Command Buffer），它包含一个命令队列。CPU 向其中添加命令，GPU 从中读取命令并执行。
2. **Draw Call 命令**：Draw Call 是 **Command Buffer 的一种命令**，用于告诉 GPU 进行绘制操作。除了 Draw Call，Command Buffer 中还有其他命令，如改变渲染状态等。
3. **CPU 和 GPU 的交互**：CPU 通过图形 API 向命令缓冲区添加命令，GPU 从中读取命令并执行。当 CPU 需要渲染一个对象时，它会向命令缓冲区添加一个 Draw Call 命令，GPU 完成上一个渲染任务后，会从命令缓冲区中取出这个命令并执行。

### 为什么 Draw Call 多了会影响帧率？

1. **CPU 的准备工作**：在每次调用 Draw Call 之前，CPU 需要向 GPU 发送很多内容，包括数据、状态和命令等。CPU 需要完成很多准备工作，如数据准备，资源绑定，命令提交，检查渲染状态等。
2. **GPU 的渲染能力**：GPU 的渲染能力很强，渲染速度往往快于 CPU 提交命令的速度。如果 Draw Call 的数量太多，CPU 就会把大量时间花费在提交 Draw Call 命令上，造成 CPU 的过载。
   可以粗浅地理解为 DrawIndexed 或Draw函数就是一次 Draw Call
3. **性能瓶颈**：由于 CPU 的准备工作和 GPU 的渲染能力不匹配，过多的 Draw Call 会导致 CPU 成为性能瓶颈，影响帧率。

### 如何减少 Draw Call？

1. **批处理（Batching）**：将多个小的 Draw Call 合并成一个大的 Draw Call，减少 CPU 提交 Draw Call 的次数和时间。
   - **静态批处理**：适合静态物体，如不会移动的大地、石头等。将这些物体标记为静态，引擎会自动合并网格，减少 Draw Call。
   - **动态批处理**：适合动态物体，引擎每帧都会重新合并网格，但限制较多，如顶点数和着色器复杂度等。
2. **减少材质使用**：相同的材质会方便合并网格，减少 Draw Call。
3. **GPU 实例化（GPU Instancing）**：将 mesh 信息存储在 GPU 内存缓冲区中，并进行渲染而无需额外的 Draw Call。
4. **减少渲染状态切换**：改变渲染状态比渲染模型更耗时，尽量减少渲染状态切换。

#### 渲染状态

**改变渲染状态**是指在图形渲染过程中，通过设置或修改渲染器（如Direct3D或OpenGL）的各种参数，来控制渲染行为和效果的过程。这些参数通常被称为渲染状态，它们定义了渲染器在处理图形数据时的行为方式。

##### 渲染状态的类型

渲染状态有很多种，常见的包括：

1. **填充模式（Fill Mode）**：决定多边形是被填充还是只绘制轮廓。例如，线框模式（Wireframe）只绘制多边形的边框，而实心模式（Solid）会填充整个多边形。
2. **剔除模式（Cull Mode）**：决定是否剔除（不渲染）某些面。例如，可以设置为剔除背面（Backface Culling），只渲染正面的三角形。
3. **深度测试（Depth Test）**：决定是否根据深度值（Z值）来决定是否更新像素的颜色。如果开启深度测试，只有当新像素的深度值小于当前像素的深度值时，才会更新像素的颜色。
4. **混合模式（Blend Mode）**：决定如何将新像素的颜色与现有像素的颜色进行混合。例如，可以设置为Alpha混合，根据新像素的Alpha值来决定如何混合颜色。
5. **模板测试（Stencil Test）**：决定是否根据模板缓冲区中的值来决定是否更新像素的颜色。模板缓冲区可以用于实现各种高级渲染效果，如阴影、轮廓等。
6. **光照模式（Lighting Mode）**：决定是否使用光照计算，以及如何计算光照。
7. **纹理过滤器（Texture Filter）**：决定如何对纹理进行采样和过滤，以获得更好的图像质量。例如，可以设置为线性过滤（Linear Filtering）或各向异性过滤（Anisotropic Filtering）

## Unity And Material

<img src="/imgs/关于SRP Batcher/OldMaterial.jpg">
CPU 收集 Material 信息，设置 CBUFFER

## SRP Batcher

<img src="/imgs/关于SRP Batcher/SRPPipeline.jpg">
在渲染循环里把材质信息保留在 GPU 内存中，只有材质内容改变才会重新 setup material data。
在此流程中，CPU 仅负责处理内置引擎属性（如*对象矩阵变换*）。所有材质的 CBUFFER 均以持久化形式驻留在 GPU 内存中，随时可供调用。

1. **材质数据持久化存储于 GPU 内存**，若材质内容未发生变更，则无需重复配置和上传缓冲区至 GPU，显著减少数据传输开销。
2. **专用代码管理全局「按对象」GPU 常量缓冲区**，通过独立优化的代码路径，高效更新 GPU 中存储大规模对象相关数据（如内置属性）的 CBUFFER，进一步降低 CPU-GPU 通信负载。

<img src="/imgs/关于SRP Batcher/SRPPipeline2.jpg">
左图为标准的 SRP（可编程渲染管线）渲染循环，右图为 SRP Batcher 的渲染循环。在 SRP Batcher 的机制中，一个“批次”仅由连续的 `Bind`（绑定资源）、`Draw`（绘制）、`Bind`、`Draw`... 等 GPU 命令序列构成。

**SRP Batcher 的核心优化逻辑**：

- **不减少 DrawCall 数量** SRP Batcher 并不会直接降低 DrawCall 的总数，其优化重点在于 **减少 DrawCall 之间的 GPU 资源设置开销**。
- **通过持久化与批次合并降低开销** 借助材质数据在 GPU 内存中的持久化存储，以及针对同着色器变体的批次合并机制，大幅减少重复的资源绑定与状态切换操作，从而提升整体渲染效率。

### 补充

#### “Per Object” 变量

<img src="/imgs/关于SRP Batcher/SRPPerObject.jpg">

## 关于

* [^设备上下文]: [Direct3D 11 中的设备简介 - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/direct3d11/overviews-direct3d-11-devices-intro)

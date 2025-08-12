---
title: GPU Driven And Hierarchical Z-Bufferr
math: true
tags: [图形学,渲染]
index_img: /imgs/Hexo主题变更/Shiki&Tsukihime.png
banner_img: /imgs/Hexo主题变更/Shiki&Tsukihime.png
date: 2025-02-23
typora-root-url: ../

---

关于 GPU Driven And Hierarchical Z-Buffer

# GPU Driven And Hierarchical Z-Buffer

## Hierarchical Z-Buffer

### Question

为什么**遮挡查询**，要2x2的像素。
首先我们的做法是把包围盒投影到屏幕空间采样对应等级的Mipmap，那这个投影过程可能就会有精度缺失。
如果之采样1个像素就会降低命中率，尤其是我们找寻最高LOD层级才能大概率找到对应的 1x1 像素，这命中率是不高的。
---
title: Visual Studio 解决项目默认的配置奇奇怪怪的问题
date: 2023-08-28 20:24:22
tags: 
typora-root-url: ./..
---
  
因为之前年轻配环境的时候不懂事，那时候应该是配OpenCv 直接配到用户默认的项目配置里面去了，不小心修改了默认的配置。
  
<!--more-->
  
# 修改默认配置
  
## 第一步：
  
打开：视图->属性管理器  
<img src="/imgs/VisualStudioPro/Vs1.png">
  
顺利的话会在右侧看见对应的窗口，然后修改 对应的 Microsoft.Cpp.x64.user(这个对应你自己要用的啊，我这里只用了 Debug|x64)  
<img src="/imgs/VisualStudioPro/Vs2.png">
  
后面应该不用教了吧，该配置里的库引用啊，lib还有连接器什么的……  
这个都不会的话，可以多做几个需要外部库的项目，比如LearnOpenGL  
然后可以理解一些编译，链接的东西，不是说去啃编译原理，我之前看过也不懂，做几个项目，或者看看视频，比如b站奇乐的有一期讲make还是cmake的我忘了，了解大概就行。  
这个没什么技术难度，就是项目见的多少。  
---
title: 使用shader加速三维点源阵仿真
date: 2022-01-17 20:29:33
tags:
  - Web
  - WebGL
categories:
  - Web
  - WebGL
---

[项目地址](https://github.com/MrThanlon/array-field/)/[预览1](https://mrthanlon.github.io/array-field/)/[预览2](https://codepen.io/mrthanlon/pen/KKXYVXN)

# 引

之前做了一个二维的点源阵仿真，还是挺简单的，D3.js直接画就完事了，但是这远远不够，工程上更多会考虑三维的情况。之前看了闫令琪老师的GAMES101和GAMES202，想着能不能趁着寒假在家用webgl做一个三维版本的。

<!--more-->

# 前置知识

## 二维情况

虽然代码里写的应该挺清楚了，不过这里还是稍微整理一下，公式推导比较复杂，二维的也不是关键，所以这里直接给出结果，

点源阵方向图可以认为是一个半径变化的圆，我们需要知道所有点源的坐标（为了方便使用极坐标系）$(d_i,\theta_i)$和相位$\Phi_i$，对于任意方向$\phi$，方向图在这个方向的半径由以下公式给出
$$
r_\phi=|\exp(j\cdot\sum d_i\cos(\phi-\theta_i)+\Phi_i)| \\
=\sqrt{\cos^2(\sum d_i\cos(\phi-\theta_i)+\Phi_i)+\sin^2(\sum d_i\cos(\phi-\theta_i)+\Phi_i)}
$$
其中$j$为虚数单位，

显然，当有$N$个点源和$M$个方向要计算的时候，复杂度为$O(MN)$，对于二维的情况$M$只需要几百即可达到肉眼看不出瑕疵的效果，对于CPU来说可以算是小菜一碟了。

## 三维情况

三维情况会比较复杂些，考虑使用球面坐标系，那么点源的坐标需要表示为三元组$(d_i,\phi_i,\theta_i)$，相位还是$\psi_i$，方向将变为二元组$(\phi,\theta)$，

现在我们计算$(\phi_i,\theta_i)$与$(\phi,\theta)$的夹角，用中学学过的向量知识我们知道，设他们的夹角是$\psi_i$那么有
$$
\cos\psi_i=\frac{\vec{u}\cdot\vec{v}}{|\vec{u}|\cdot|\vec{v}|}
$$
其中$\vec{u}$表示方向$(\phi_i,\theta_i)$的向量，$\vec{v}$表示$(\phi,\theta)$方向的向量，为了方便乘法计算我们变换到空间直角坐标系，并令$\vec{u}$和$\vec{v}$都为单位向量，得到
$$
\cos\psi_i=\sin\theta_i\cos\phi_i\sin\theta\cos\phi+\sin\theta_i\sin\phi_i\sin\theta\sin\phi+\cos\theta_i\cos\theta
$$
于是我们可以直接带入到之前二维的半径公式上，注意到二维中计算夹角很简单，只需要$\phi-\theta_i$，即现在我们将它用$\psi_i$进行替换，可得到三维情况下的半径公式
$$
r_{\phi,\theta}=|\exp(j\cdot \sum d_i\cos\psi_i+\Phi_i)| \\
=\sqrt{\cos^2(\sum d_i\cos\psi_i+\Phi_i)+\sin^2(\sum d_i\cos\psi_i+\Phi_i)}
$$

# GPU编程与三维模型展示

为了方便我就直接用three.js了，它能够很方便地利用WebGL展示三维模型，同时也支持自行编写着色器程序（ShaderMaterial）。

按我自己实验，要想达到比较好的效果，采样点会上万，这样的计算量对于CPU来说很难做到实时计算了，因此我们需要通过并行计算更强大的GPU来完成。

## TODO: 使用顶点着色器（vertex shader）

在三维渲染中，顶点着色器主要完成MVP(Model, View, Perspect)矩阵变换，当然我们也可以夹一些私货进去，例如修改顶点的位置坐标。

为了进行计算和渲染，我们需要一个合适的三维模型，有些同学想既然二维是圆，那么三维自然就是球了，但是我们不能使用球面坐标，因为它并不均匀，为了使得模型更加均匀，我们使用的是正二十面体细分为球体，好在three.js中刚好提供了正二十面体的构造器（IcosahedronGeometry），而且支持细分，所以我们直接用它就好了。（关于为什么用正二十面体而不是正四/六/八/十二面体，因为正四/八面体细分之后还是不均匀，而正六/十二面体的面不是三角形，WebGL只能渲染三角形）。

## TODO: 使用计算着色器（compute shader）

只是得到一个图案可不能叫仿真，我们有时候还要关心点源阵的实际参数，例如主波束宽度，定向性等。

可惜的是顶点着色器在计算完成后结果就直接传到片段着色器（fragment shader）中了，CPU是取不回来的，而如果既要使用GPU加速，又能够对计算结果进行分析的话，现代GPU编程给我们提供了一个更自由的工具，也就是计算着色器。关于计算着色器我不打算在本文中详细介绍，有兴趣的同学可以自行搜索相关内容。

但是比较可惜的是，目前只有WebGL2.0能支持计算着色器，而且苹果的浏览器Safari直到15版本才支持WebGL2.0，而且还刚好不支持计算着色器。

## TODO: 使用WebGPU

前面提到了Safari15的WebGL2.0并不支持计算着色器，所以对于iOS设备我们得想其他办法，例如WebGPU，不过截止本文发表，WebGPU在Safari15上还是试验性功能，需要在设置中开启，Chrome只有金丝雀测试版支持WebGPU。

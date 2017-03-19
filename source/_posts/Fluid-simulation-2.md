---
title: 电影工业中的流体模拟（二）- 数学基础
date: 2017-03-19 18:26:20
tags:
---
在本章之中，我们对流体模拟中需要用到的数学概念进行简单的介绍，其中包括梯度、散度、旋度、拉普拉斯算子与高斯定理等。为了便于阅读，以下对这些概念以及涉及的公式统一使用流体模拟相关文献常用的记号方式。
<!--more-->

**梯度**
一个函数的梯度(gradient)用于描述这个函数在给定空间中上升最快的方向。如图：
![梯度](https://i.imgur.com/TlGdOJy.png)
这张图中蓝色网格便是函数 $f(x,y) = −(cos^2x + cos^2y)^2$ 的图像，下面的红色箭头则绘制了该函数在二维空间中毎点的梯度场。
计算一个函数的梯度就是计算它所有分量上的空间偏导数，组合为一个向量，即函数在该点的梯度。比如，在三维空间中，函数 $f(x, y, z)$ 的梯度即写成：
$$\nabla f(x,y,z)=(\frac{\partial f}{\partial x}, \frac{\partial f}{\partial y}, \frac{\partial f}{\partial z})$$
其中$$\frac{\partial f}{\partial x}$$ 即是函数$f$ 在$x$方向的偏导数。

有些时候，我们需要知道函数在特定方向上的变化率，通过将梯度与该方向的向量点乘，我们就可以计算出函数在这一点的方向导数，如函数 $f$ 在方向 $\boldsymbol{n}$ 上的方向导数记为：
$$\frac{\partial f}{\partial \boldsymbol{n}}=\nabla f\cdot\boldsymbol{n}$$

**散度**
散度(divergence)算子用于描述向量场中在某点的聚集或发散程度，如图：
![散度](https://i.imgur.com/RUggoZ8.jpg)
左边的向量场在中心点散度为正，中间的向量场在中心点散度为负，右边的向量场在各点散度为零。

在三维空间中，向量场$\boldsymbol{u}$的散度可以写成（这里我们用粗体的$\boldsymbol{u}$表示向量场，普通的$u$则表示在第一个维度上的分量）：
$$\nabla\cdot\boldsymbol{u}=\nabla\cdot(u,v,w)=\frac{\partial u}{\partial x}+\frac{\partial v}{\partial x}+\frac{\partial w}{\partial x}$$
注意虽然散度仅仅对向量场才有意义，但它本身是标量而不是向量。另外，散度符号之所以记成这样的形式是因为它也可以理解为梯度算子与向量场的点乘：
$$\nabla\cdot\boldsymbol{u}=(\frac{\partial}{\partial x}, \frac{\partial}{\partial y}, \frac{\partial}{\partial z})\cdot (u,v,w)=\frac{\partial u}{\partial x}+\frac{\partial v}{\partial x}+\frac{\partial w}{\partial x}$$
在流体模拟中，由于水、油等液体通常具有难以压缩(incompressible)的性质，因此这些液体的速度场通常被认为散度为零，这些散度为零的向量场也被称为无散场。尽管在实际计算过程中，实现这一约束通常是最耗时的一步，但通常认为，这种无散的特性，是在视觉上让模拟具有真实感的最重要的一步。

比如下面的图，展示了一个变化中的无散度的随机速度场。即便是这样一个随机场，在散度为零的时候，顺着这些速度运动的粒子都能在视觉上表现出很强的流体特征：
![随机无散场](https://i.imgur.com/dzunYiq.gif)
对于无散场与流体模拟的更深刻的关系，以及如何在计算中保证速度场散度为零，我们将在后续的章节中进行深入介绍。

**旋度**
旋度(curl)算子，顾名思义，用于描述向量场绕某点旋转的剧烈程度及方向。
![旋度](https://i.imgur.com/9jAsHWd.png)
在三维空间中旋度是一个向量，它的计算方式为：
$$\nabla\times\boldsymbol{u}=\nabla\times (u,v,w)=(\frac{\partial w}{\partial y}-\frac{\partial v}{\partial z}, \frac{\partial u}{\partial z}-\frac{\partial w}{\partial x}, \frac{\partial v}{\partial x}-\frac{\partial u}{\partial y})$$
要注意的是，一个向量场的旋度场是无散的，即我们总有：
$$\nabla\cdot\nabla\times\boldsymbol{u}=0$$
因此，在制作特效中，一些不需要太多真实感的流体模拟，可以用一种称为**旋度噪声**（[Curl Noise](http://www.cs.ubc.ca/~rbridson/docs/bridson-siggraph2007-curlnoise.pdf)）的技术完成。具体说来，便是生成一些根据时间变化的三维[Perlin噪声](https://zhuanlan.zhihu.com/p/22337544)，然后取其旋度作为速度场，并将粒子顺着速度场移动，即可得到如下效果：
![旋度噪声](https://i.imgur.com/Z8aaI3L.jpg)
有兴趣的读者可以在这里看到WebGL实现的可交互效果和代码：
http://graphics.thew.nu/curl/
以及其他的一些[实现细节](http://prideout.net/blog/?p=63)

在接下来的章节，我们将利用以上的数学知识，从描述流体的最基本方程（纳维-斯托克斯方程）讲到如何生成细节丰富的特效。敬请期待。

\_(:3」∠)\_ \_(・ω・”∠)\_ \_(:з)∠)\_ ∠( ᐛ 」∠)＿ \_(:зゝ∠)\_
请毫不犹豫地关注我们：
我们的网站：[https://graphicon.io](https://graphicon.io)
知乎专栏：[GraphiCon图形控](https://zhuanlan.zhihu.com/graphicon)
公众号：GraphiCon
![qr code](https://ooo.0o0.ooo/2017/03/14/58c7de193d7ac.png)
如果你有什么想法，建议，或者想加入我们，你可以：
给我们发邮件：[hi@graphicon.io](mailto:hi@graphicon.io)
加入我们的QQ群：SIQGRAPH（342086343）
加入我们的slack群：[GraphiCon](https://graphicon.herokuapp.com/)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">知识共享署名-非商业性使用-禁止演绎 4.0 国际许可协议</a>进行许可。


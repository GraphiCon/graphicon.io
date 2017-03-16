---
title: mesh-deformation-1
date: 2017-03-16 09:54:21
tags:
---

![teaser.png](https://ooo.0o0.ooo/2017/03/10/58c196b927ffd.png)

# 用数学编辑3D模型（一）- Mesh Deformation with Laplacian coordinates

## 引言

想必在我们之前的[“光影之下”系列第一篇“从Logo谈起”](https://zhuanlan.zhihu.com/p/25766738)中，大家已经隐约感受到了ARAP（as-rigid-as-possible）这个能够尽量保持局部形状不变的全局变形法的强大功能。那么作为文中提到的M神，今天，我暂时还不能给大家介绍ARAP算法。因为凡事需从基础开始，所以，今天先来谈一谈ARAP所基于的最基本的一种3D模型形变方法——Mesh Deformation with Laplacian coordinates。注意，前方高能（公式有点多）！

## 背景介绍

在CG虚拟世界中，不同于以粒子表示的流体，固体与软体等有明确形状的实体物体通常是由包裹在其表面的三角形面片网格（triangle mesh）来离散化表示的。triangle mesh包括物体表面上的多个定义在三维空间中的离散采样点，以及连接它们所组成的三角形面片。采样点越密集，模型就越精细（如下图）。

![meshRes.png](https://ooo.0o0.ooo/2017/03/10/58c196bf3e45d.png)

要想对triangle mesh这类3D模型进行编辑，一般需要给离散点赋予新的三维位置坐标。显然，手动去设置每一个离散点的位置既不够直观又非常麻烦。所以，智能化的模型编辑工具对艺术家来说十分重要。

事实上，在大多数编辑过程中，我们只是想去改变一个模型的姿势或者说大体形态（如下图所示的一种编辑），而不是模型的表面细节（比如这只大鸟的眼睛、肌肉突起）和组成部分（比如大鸟的四肢、头和尾）。

![meshEdit.png](https://ooo.0o0.ooo/2017/03/10/58c196bd5209d.png)

那么，有没有一种智能化的方法，能够让用户只需设置个别离散点的新位置来表达他所想要的形变，就能自动根据所需保持的形体信息来计算出剩余离散点应有的位置呢？我们今天要说的mesh deformation with Laplacian coordinates就是这样一种智能的模型编辑工具。

## 基本方法

该算法通过在满足用户设置的部分离散点新位置的情况下，最小化一个表征编辑前后模型表面细节差异的函数来求出新模型各离散点的位置，并由原有的连接构成新的triangle mesh。这种解决问题的方法叫做**数学优化**，我们在高中时学的求一元实函数的最值问题就是它的最简单的形式。这里我们的函数有$3n$个自变量（$n$为离散点个数，常见的从几百到上十万不等，每个离散点都定义在三维空间），属于多元函数，所以优化它还需要用到**线性代数**和**多元微积分**的知识。

对每一个离散点$x_i \in R^3$，其细节信息可以用定义在triangle mesh上的离散拉普拉斯算子（discrete Laplacian operator）来描述，即连接$x_i$到与$x_i$相连的所有离散点$x_j$，$j \in N(i)$的中心位置的向量$l_i = \sum_j^{N(i)}w_{ij}x_j - x_i$（如下图c）。这里算子中的权重$w_{ij}$可以是均匀权重，或者[cotangent权重](https://zhuanlan.zhihu.com/p/25496167)：$w_{ij} \propto \frac{1}{2}(\cot \alpha_j + \cot \beta_j)$（如下图b），且 $\sum_j^{N(i)}w_{ij} = 1$ [[Desbrun et al. 1999]](http://w.multires.caltech.edu/pubs/ImplicitFairing.pdf)。后者考虑到了离散采样点分布的不均匀性，能更好的描述该处的细节信息。

![discreteLap.png](https://ooo.0o0.ooo/2017/03/10/58c196af20e2d.png)

*\*注1：我们将会在[“电影工业中的流体模拟”系列](https://zhuanlan.zhihu.com/p/25774747)的第三篇“纳维斯托克斯方程”中给出拉普拉斯算子在连续意义上的定义，以及它在均匀网格（uniform grid，常用来求偏微分方程数值解）上的计算式。*

有了这个具有描述模型局部细节的算子，我们就可以写出需要最小化的目标函数了：

$$f(x_1, x_2, ..., x_n) = \sum_{i=1}^n \| (\sum_j^{N(i)} w_{ij}x_j - x_i) - l'_i \|^2$$

即最小化编辑前后模型间各对应点细节差异之和（这里$l'_i$为编辑前模型上点$i$处的discrete Laplacian，表示编辑前该点处的细节信息；向量间的差异用欧式距离度量）。同时还要注意用户输入的部分点所需满足的位置限定（position constraints）：

$$x_k = p_k, k \in M$$

其中$M$为用户所选定的预先限定位置的离散点所组成的集合，$p_k$为对点$k$所指定的位置。为了在满足position constraints的同时最小化函数$f$，在mesh deformation的情境中我们一般用penalty method把这一constrained optimization problem转化为unconstrained optimization problem，即定义新的目标函数：

$$f_p(x_1, x_2, ..., x_n) = \sum_{i=1}^n \| (\sum_j^{N(i)} w_{ij}x_j - x_i) - l'_i \|^2 + \alpha\sum_{k \in M} \| x_k - p_k\|^2$$

其中$\alpha$为限制条件的严格程度，$\alpha$越大，条件将被满足得越好；当$\alpha$趋近于无穷时，条件将被严格满足。这样一来，优化$f_p$这个二次型就相当于在满足position constraints的情况下优化$f$了。由于二次型只有一个极值点（local optimum），所以这个极值点同时也是个最值点（global optimum），故可以直接令其梯度（gradient）为零以求出目标函数值最小时的自变量，也就是新模型各点的位置坐标。由于我们不希望强行以牺牲细节为代价去严格满足position constraints，故一般无需将$\alpha$设置太大，在$[1, 10]$区间内效果就不错。

*\*注2：当需要在一个数学优化问题中严格地满足一个等式限定条件（equality constraints）时，通常还可采用拉格朗日乘子法（Lagrange Multipliers）将它转化成一个unconstrained问题求解。如果限定条件由不等式定义，则可考虑barrier method或Interior point method。详见[convex optimization [Boyd and Vandenberghe 2004]](https://web.stanford.edu/~boyd/cvxbook/bv_cvxbook.pdf)。*

前文提到的线性代数和多元微积分在这里的作用就是指导对多元函数的各种操作，比如$f_p$其实可以用矩阵的形式写成：

$$f_p(x) = \|Lx - l'\|^2 + \alpha \| Sx - p \|^2$$

也就是一个Linear least squares。于是其梯度可以写成$g_p(x) = 2(L^TLx - L^T l') + 2\alpha (S^TSx - S^Tp)$，那么问题就由此转化成了求解$x$，使得$g_p(x) = 0$，即求解线性方程组：

$$(L^TL+\alpha S^TS)x = L^Tl' + \alpha S^Tp$$

用计算机求解线性方程组需要用到**数值计算**工具，通常情况下我们可以直接调用现成的库（比如[Eigen](http://eigen.tuxfamily.org/)）。这里由于系数矩阵$L^TL+\alpha S^TS$是稀疏的（sparse，因为每个点只有几个相邻的点，故非零系数个数$m$与方针列数$3n$满足$m \sim O(3n)$）对称正定矩阵（symmetric positive definite matrix），所以我们可以选用sparse Cholesky decomposition或者conjugate gradient method来求解。关于这个线性方程组是否有解的问题，结论是当每个连通的triangle mesh上至少有一个离散点被用户选定并指定position constraint时，此方程组有唯一解，故不用担心。

在实际求解过程中，由于目标函数中离散点坐标的三个维度之间并没有耦合，故可以分别对$x$、$y$、$z$三个维度进行求解。于是先选取sparse Cholesky decomposition对它们共同的系数矩阵进行分解，再对三个不同的RHS来做back-substitution就会非常快了。

*\*注3：求解线性方程组的方法主要分为直接法（direct method）和迭代法（iterative method）两大类，后者处理起大规模稀疏系统更有优势（详见[A First Course In Numerical Methods [Ascher and Greif 2011]](http://www.siam.org/books/cs07/)）。通常拿到一个线性方程组，若其系数矩阵不是方阵，一般可考虑先用normal equation将其转化为方阵线性系统。然后，我们首先要分析它是否有解：*

 - *若有唯一解，则可先使用direct method中求解一般线性系统的LU分解方法对其进行求解。有了这个保底选项后，再充分利用当前问题以及系数矩阵的性质来选取更合适的方法（比如上文选取的sparse Cholesky decomposition和conjugate gradient method）。*
 - *当遇到无解的情况时，可以使用normal equation把等式右边向量（RHS，right hand side）投影到系数矩阵的列向量空间（column space）中再进一步分析（投影后系统一定有解，但解不一定唯一）。*
 - *当遇到有无穷多个解的情况时，可以考虑使用truncated SVD求出模（norm）最小的那个解，或者根据实际问题的性质添加specific regularizer（也就是先验知识）告诉计算机根据你的需求找出最合适的那个解。*

*在使用计算机求解线性方程组数值解的过程中，还可能遇到ill-condition的情况，即系统对误差非常敏感，求解过程十分不稳定，此时可考虑用preconditioning来解决。如果问题本身是一个linear Least squares，则还可考虑直接用QR分解来求解。*

## 分析与改进

该方法的实现过程主要就是根据原始模型数据，通过编程计算出上述系数矩阵和RHS，然后调用[Eigen](http://eigen.tuxfamily.org/)库求解线性方程组得到新的3D模型。至于如何获取用户设定的position constraints，则涉及到一些UI编程。

那接下来我们就来测试一下这个方法！我们将一个章鱼模型的一条触须的根部和它身体的其余部分固定，再将该触须顶部向右下方拖拽（如下图a）。用上述方法求解后就可以得到下图b所示的结果。我们可以看到虽然大体形态基本是我们想要的，但所编辑触须上的细节并没有得到很好的保持，而是有所扭曲。相比之下，我们更希望得到下图c所示的结果。那么，问题究竟出在哪呢？

![lapEdit.png](https://ooo.0o0.ooo/2017/03/10/58c196bad9cc1.png)

问题就在于我们的discrete Laplacian并无法将只是经过了旋转的离散点局部细节判定为和之前相同，而事实上这种旋转在编辑过程中并不会改变局部细节。如下图所示的一个离散点和其邻近点所组成的局部，b由a旋转所得，我们认为a和b其实是具有一样的细节的，但他们两者的discrete Laplacian之差的欧氏距离很大。而c反而会被当前的算法认为和a很相似，但其实它已经是被扭曲了。

![lapRot.png](https://ooo.0o0.ooo/2017/03/10/58c196afde4df.png)

所以我们需要改进当前的目标函数，让它包含这种对旋转的容忍。假设我们想要的新模型上各点$x_i$局部相对于原始模型相应点$x'_i$局部之间的旋转关系为$R_il'_i=l_i$（这里$R_i$是一个$3\times3$的旋转矩阵），那么我们就可以很容易的把目标函数改写成：

$$f_{pr}(x_1, x_2, ..., x_n, R_1, R_2, ..., R_n) = \sum_{i=1}^n \| (\sum_j^{N(i)} w_{ij}x_j - x_i) - R_il'_i \|^2 + \\ \alpha\sum_{k \in M} \| x_k - p_k\|^2$$

以及对$R_i$必须是旋转矩阵的约束条件：

$$R_i^TR_i = I, det(R_i) = 1$$

这样就可以把上图a和b细节相同的考虑包含进来。至于上图c这种被扭曲反而不改变discrete Laplacian的情况，在对整个triangle mesh上的所有离散点同时求解时基本是不会出现的。虽然新的目标函数还是一个二次型，约束条件也还是等式，但这里的约束条件可不像之前的position constraints那么好对付了，因为这些constraints是nonlinear（非线性）的。即使再使用penalty method或者使用Lagrange Multipliers把问题转化为unconstrained problem，然后令其梯度等于零，得到的方程组也是非线性的，这使得我们之前用来求解线性方程组的工具都不够用，更别说如果这个非线性函数非凸（non-convex）的话，其极值点不一定都是最值点。

*\*注4：一般解非线性问题常用的是gradient descent（梯度下降法），Newton's method（牛顿法）等几类hill climbing method和它们的变种（详见[convex optimization [Boyd and Vandenberghe 2004]](https://web.stanford.edu/~boyd/cvxbook/bv_cvxbook.pdf)）。在工程应用中，人们时常也会想一些trick（比如change of variables、local/global）来绕过问题中的nonlinearity或间接对其求解。对于找到的极值点不一定是最值点的问题，有一种解决办法是去找到一个合适的自变量起始点（initial guess），从这个起始点开始优化目标函数，得到的极值点即使不是最值点也是在应用场景中可接受的结果。*

为了保持求解的简洁性，[[Lipman et al. 2004]](https://igl.ethz.ch/projects/Laplacian-mesh-processing/Laplacian-mesh-editing/diffcoords-editing.pdf)根据优化$f_p$得到的初步结果将$R_i$估算出来，然后直接代入$f_{pr}$再次优化求出最终的解，得到的结果也就是之前那张章鱼示意图中的图c了。然而我们很容易注意到，虽然图c的小圆圈变得没有那么扭曲了，但它们还是和图a中原始模型上的小圆圈差别很大，并不自然，毕竟估算出的$R_i$不一定准确。所以，我们还是不得不直接去面对这个nonlinear optimization problem。不过此刻，让我们继续来欣赏一些这个简单可行的方法的更多结果吧：

![moreResults.jpg](https://ooo.0o0.ooo/2017/03/16/58ca18a3b021d.jpg)

## 结语

我们将会在本系列的第二篇中给大家介绍倍受业界欢迎的ARAP算法[[Sorkine and Alexa 2007]](https://www.igl.ethz.ch/projects/ARAP/arap_web.pdf)，它采用了一种叫local/global的方法来求解这个非线性优化问题。

*\*注5：本文所讨论的优化问题，其目标函数都是光滑的，至少存在一阶导。那些不光滑（可以考虑使用[proximal algorithm [Parikh and Boyd 2014]](http://web.stanford.edu/~boyd/papers/prox_algs.html)来处理）甚至离散（比如graph search）的目标函数，常见于机器学习和人工智能中的优化问题。*

## 参考文献

[Sheffer, A. (2016). Computer Graphics: Modeling. [Powerpoint slides]](http://www.cs.ubc.ca/~sheffa/dgp/)

[[Desbrun et al. 1999]](http://w.multires.caltech.edu/pubs/ImplicitFairing.pdf) Desbrun, M., Meyer, M., Schröder, P., & Barr, A. H. (1999, July). Implicit fairing of irregular meshes using diffusion and curvature flow. In Proceedings of the 26th annual conference on Computer graphics and interactive techniques (pp. 317-324). ACM Press/Addison-Wesley Publishing Co..

[[Boyd and Vandenberghe 2004]](https://web.stanford.edu/~boyd/cvxbook/bv_cvxbook.pdf) Boyd, S., & Vandenberghe, L. (2004). Convex optimization. Cambridge university press.

[[Ascher and Greif 2011]](http://www.siam.org/books/cs07/) Ascher, U. M., & Greif, C. (Eds.). (2011). A First Course on Numerical Methods. Society for Industrial and Applied Mathematics.

[[Lipman et al. 2004]](https://igl.ethz.ch/projects/Laplacian-mesh-processing/Laplacian-mesh-editing/diffcoords-editing.pdf) Lipman, Y., Sorkine, O., Cohen-Or, D., Levin, D., Rossi, C., & Seidel, H. P. (2004, June). Differential coordinates for interactive mesh editing. In Shape Modeling Applications, 2004. Proceedings (pp. 181-190). IEEE.

[[Sorkine and Alexa 2007]](https://www.igl.ethz.ch/projects/ARAP/arap_web.pdf) Sorkine, O., & Alexa, M. (2007, July). As-rigid-as-possible surface modeling. In Symposium on Geometry processing (Vol. 4).

[[Parikh and Boyd 2014]](http://web.stanford.edu/~boyd/papers/prox_algs.html) Parikh, N., & Boyd, S. (2014). Proximal algorithms. Foundations and Trends® in Optimization, 1(3), 127-239.

<br/>

\_(:3」∠)\_ \_(・ω・”∠)\_ \_(:з)∠)\_ ∠( ᐛ 」∠)＿ \_(:зゝ∠)\_
GrapiCon图形控:有趣的图形学
请毫不犹豫地关注我们：
我们的网站：[https://graphicon.io](https://graphicon.io)
知乎专栏：[GraphiCon图形控](https://zhuanlan.zhihu.com/graphicon)
公众号：GraphiCon
![qr code](https://ooo.0o0.ooo/2017/03/14/58c7de193d7ac.png)
如果你有什么想法，建议，或者想加入我们，你可以：
给我们发邮件：[hi@graphicon.io](mailto:hi@graphicon.io)
加入我们的QQ群：SIQGRAPH（342086343）
加入我们的slack群：[GraphiCon](https://graphicon.slack.com/)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">知识共享署名-非商业性使用-禁止演绎 4.0 国际许可协议</a>进行许可。

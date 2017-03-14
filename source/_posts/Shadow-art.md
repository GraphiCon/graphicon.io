---
title: 光影之下（一）- 从Logo谈起
date: 2017-03-13 01:28:09
tags:
---
计算机图形学可以说是一门研究光与影的科学（当然不只研究光影），在渲染，3D重建，甚至3D打印，数字制造都有和光影相关的分支。不过今天我们要谈一个略微奇怪的方向，就从我们的LOGO说起吧！
<!--more-->
相信小伙伴们也注意到了吧，我们的LOGO是3D渲染的！
什么？没注意？再show一下：
![logo](https://ooo.0o0.ooo/2017/03/13/58c5e9d32d311.png)

它是怎么做的呢，其实我们造了一个模型。就像这个动图里这样，C字形加了一个小尾巴，看起来像G。一打光，就有一个C形的影子了。也就是我们GraphiCon的两个开头字母G，C。

![logo.gif](https://ooo.0o0.ooo/2017/03/10/58c298ad8760d.gif)

有这个想法也不是我们哪天突然开了个脑洞想出来的。其实在神书「哥德尔、艾舍尔、巴赫:集异璧之大成」的封面上就出现过这种效果，图中这个几何形状从3个方向看分别投影出G（Gödel），E（Escher）和B（Bach）3个字母。
![GEB.jpg](https://ooo.0o0.ooo/2017/03/14/58c6ef95c06ac.jpg)

这时候问题就来了，这时候问题就来了，**假如我们给出三个方向的影子作为输入，怎么生成这样一个几何形状呢？**还真有人想过这个事，2009年，现在都已经成为超级大牛的Niloy Mitra和Mark Pauly就发了这样一篇文章，名叫Shadow Art[[1]](#references)，从几何处理的角度解决了这个问题。

我们先从头来看，这问题真的很难吗？拿三幅图在三个角度投个影求个交不就完事了？事实是这样吗？我们来试试！

先从「集异璧」开始吧：
我们在OpenSCAD里画一个长长的「G」和它的影子：
![G](https://i.imgur.com/yubQ8LQ.png)
再画个「E」和「B」还有他们的影子：
![E](https://i.imgur.com/pA2LL96.png)![B](https://i.imgur.com/pdNMF82.png)
再对它们求个交：
![GEB](https://i.imgur.com/VXQCjuK.png)
Bingo！成了，这不很简单嘛～
唔，我们把「B」调大点试试：
![B_big](https://i.imgur.com/ZnrJwai.png)
求个交：
![bigGEB](https://i.imgur.com/MAbl3Fk.png)
这什么鬼，怎么和说好的不一样？是巧合吧，难道这模型太复杂了？
我们来试试简单的形状，比如三角形，方形和圆形：

![triangle](https://i.imgur.com/bN8pqLY.png)![square](https://i.imgur.com/0GR7Pk4.png)
![circle](https://i.imgur.com/6NdKLOj.png)
开心地求个交：
![enter image description here](https://i.imgur.com/uO4NYuK.png)
什么？还是不行？看来「GEB」能work才是巧合啊。。。看来这事不简单啊！
两位大牛也发现了这个问题，不过人家是大牛我是菜鸡就是因为人家想到了解法我只是在这发呆。
让我们来看看这该怎么办。
![Analysis](https://i.imgur.com/ABlkDSh.png)
图里面绿色，粉色和紫色的投影部分就是本来属于一个图像但求交过程中被削去的的部分，或者叫做投影不一致的区域（inconsistent region），图中的红点就是一个投影不一致的像素（inconsistent pixel）。这样一看就明白啦，我们的目标就是消除投影不一致的像素，那么要么我们在原图里砍掉这些投影不一致的像素（我们当然不想要这种），要么就把其它视角的原图稍微变个形让这些投影不一致像素落到其它视角的图像内，让这种坏像素变成好像素好了。


就这么简单？就这么简单！


好吧，小伙伴们把家伙们都拿出来，变形嘛，就用最常用的As-rigid-as-possible deformation（ARAP）好了，这个方法用一句话说起来就是尽量保持局部形状不变的全局变形法，我们的M神会专门写一篇文章好好讲讲As-rigid-as-possible deformation（ARAP）。

Anyway，「汽车人，变形！」
![Optimization](https://i.imgur.com/fAhb1rL.png)
比如我们想把中间米老鼠脚上的投影不一致像素变成好像素，我们选一圈坏像素（红色部分），找到他们在史努比视角和大力水手视角的对应点，然后用ARAP变形史努比和大力水手，使这些坏像素变成好像素，循环往复，最后就生成了新的史努比，米老鼠和大力水手。那么他们再求个交，就成了这样：
![shadow_cartoon.gif](https://ooo.0o0.ooo/2017/03/14/58c7440edff04.gif)
Done！

这时候或许有人会问（没人问我自己问）：你讲半天为啥一个公式也没有？
怪我咯？这篇文章里面就一个公式也没有！还是篇SIGGRAPH Asia
![fule](https://i.imgur.com/iPN4p8E.jpg)

或许你觉得这种文章没什么用，可人家还真有几篇follow up，借用投影和轮廓概念来建模的3D Modeling with Silhouettes[[2]](#references)，还有去年一篇算怎么用人来摆影子的paper：Shadow Theatre: Discovering Human Motion from a Sequence of Silhouettes[[3]](#references)

大家看的是不是意犹未尽？反正我是写的停不下来。。。下一期「光影之下」会讲讲这篇文章里的另一个大牛Mark Pauly的代表性项目：Computaional caustics，敬请期待！（一大波公式正在接近中！！！）
## References:
[1]: [Shadow Art](http://vecg.cs.ucl.ac.uk/Projects/SmartGeometry/shadowArt/shadowArt_sigA_09.html); Niloy J. Mitra, Mark Pauly; ACM SIGGRAPH Asia 2009.
[2]: [Shadow Theatre: Discovering Human Motion from a Sequence of Silhouettes](http://mrl.snu.ac.kr/research/ProjectShadowTheatre/ShadowTheatre.htm); Jungdam Won, Jehee Lee; SIGGRAPH 2016.
[3]: [3D Modeling with Silhouettes](http://www.alecrivers.com/3dmodelingwithsilhouettes/); Alec Rivers, Frédo Durand, Takeo Igarashi; SIGGRAPH 2010.

\_(:3」∠)\_ \_(・ω・”∠)\_ \_(:з)∠)\_ ∠( ᐛ 」∠)＿ \_(:зゝ∠)\_
GrapiCon图形控:有趣的图形学
请毫不犹豫地关注我们：
我们的网站：[https://graphicon.io](https://graphicon.io)
知乎专栏：[GraphiCon图形控](https://zhuanlan.zhihu.com/graphicon)
公众号：GraphiCon
![qr code](https://ooo.0o0.ooo/2017/03/12/58c52755a9463.jpg)
如果你有什么想法，建议，或者想加入我们，你可以：
给我们发邮件：[hi@graphicon.io](mailto:hi@graphicon.io)
加入我们的QQ群：SIQGRAPH（342086343）
加入我们的slack群：[GraphiCon](https://graphicon.slack.com/)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">知识共享署名-非商业性使用-禁止演绎 4.0 国际许可协议</a>进行许可。

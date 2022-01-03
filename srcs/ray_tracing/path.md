## 蒙特卡罗积分
为什么我们需要蒙特卡罗积分？对于一个简单的函数，比如 $f(x) = x^2$ ，我们可以轻易地算出其积分的结果。
但是对于我们的渲染方程，我们很难给出一个积分结果表达式。这个时候我们不得不放弃对表达式的求解，转而尝试求出其数值解。

我们先从一个非常简单的例子入手：

![Screenshot from 2022-01-03 12-57-59.png](https://s2.loli.net/2022/01/03/NRTnvbXBhAm1edz.png)

假定此时的你并不知道圆形的面积为 $\pi r^2$ ，我们如何求出内切圆的面积（假设正方形的边长为 $a$ ）？
大家应该都听说过这个做法：将一把豆子均匀随机地撒到正方形内，并求出落在圆内的数量。
假设 $f(i)$ 表示第i粒豆子是否落在圆内（圆内取值为1,否则为0），那么圆形面积就可以近似地表示为 $S\sum_i^N\frac{f(i)}{N}$ 。

这是通过概率的方法来对面积进行近似。如果我们投入的豆子的数量足够多， $\frac{f(i)}{N}$ 就会越来越接近 $\frac{S'}{S}$ ，其中 $S'$ 代表圆形的面积。
只要 $N$ 足够大，$S\sum_i^N\frac{f(i)}{N}$ 就能任意地逼近 $S'$ 。

现在，我们做一些微小的变形，将 $S$ 放入到求和式中，并将 $N$ 拿出来： $S' \approx \frac{1}{N}\sum_i^N \frac{f(i)}{\frac{1}{S}}$ 。
此时，求和式两部分组成：

+ $f(i)$ ：这个函数只有当豆子落在圆内的时候才会取1，否则就是0。我们可以认为这是对正方形内部情况的一个采样
+ $\frac{1}{S}$ ：这部分是原本式子外部的正方形面积 $S$ ，我们将它写为这个形式并放在了分母上，这样就可以将其看作一个概率 $p(i) = \frac{1}{S}$ ，这个概率恰好是对正方形内部任意一个点进行采样的概率

因此，我们可以整理表达式 $S' \approx \frac{1}{N}\sum_i^N \frac{f(i)}{p(i)}$ 。这就是这个例子的蒙特卡罗积分求解。
对于一般的形式，我们有 $\int_S f(x)dx = \frac{1}{N}\sum_i^N\frac{f(X_i)}{P(X_i)}$ 。

你可能很惊讶，但事实确实如此，我们已经完成了蒙特卡罗积分的推导（非严格地）。我们可以再从概率论的角度来看这个式子：

![Screenshot from 2022-01-03 13-23-00.png](https://s2.loli.net/2022/01/03/VMS2BPcutEbpXks.png)

我们如何对这个函数在区间上求积分？如果我们不知道函数的式子，我们可以使用最传统的求和来做： $\int_a^b f(x)dx \approx \frac{1}{N}\sum_i^Nf(x)dx$ 。
如果 $N$ 越大，这个结果就越准确。或者换一个视角：如果 $N$ 越大，那么每个可能的取值被取到且恰好取到一次的概率就越接近于 $1$ 。

然而对于计算机来说，让 $N$ 趋近于无穷是不可能实现的。因此，在每一次取值的时候，每个点都会有一个被取到的概率 $p(x)$ ，并且 $p(x)$ 越大，被重复取到的机会也就越多。
为了减少重复取值对函数估计造成的影响，我们在每次取值下面除以一个 $p(x)$ ，以减少该值的权重。这就是蒙特卡罗积分。

## 路径追踪
我们u已经知道如何使用Whitted风格的光线追踪了，但是这样做的效果往往很不好。根据渲染方程，每一个点我们都需要对周围进行大量的采样，这会导致（图片来自于GAMES101，下同）：

![Screenshot from 2022-01-03 14-16-48.png](https://s2.loli.net/2022/01/03/fKBOduzLMaDRVvJ.png)

是的，我们最后需要模拟的光线将成千上万，这不是计算机可以负担的起的。因此，我们需要将采样次数降低为1次。
我们降低了 $N$ ，效果当然会变得糟糕。我们需要使用蒙特卡罗积分来帮助我们进行优化。

对于需要渲染的一个像素，我们打出N束光线，即进行N次采样，并使用蒙特卡罗积分进行求和。在弹射的过程中，我们就不再需要弹射出多根光线：

![Screenshot from 2022-01-03 14-26-56.png](https://s2.loli.net/2022/01/03/HNeBZU7sbrv8yum.png)

我们依然需要限制弹射的次数。如果递归的次数过多，我们也无法负担过高的性能开销。当递归达到一定次数时，我们就认为光线的热情与斗志，都已经消耗殆尽了，
说什么它都不会再返回一个颜色了：

```c++
// If we've exceeded the ray bounce limit, no more light is gathered.
if (depth <= 0)
    return color(0,0,0);
```

由于每次射出的光线只有唯一的一个弹射方向，光线在空间中形成了一条条路径，因此这个算法被称为路径追踪（Path Tracing）。

这样的路径追踪在某些情况下往往有着较大的噪声。比如光源较小的情况下，可能很多可以被光照直接照明的地方由于随机的缘故无法将光线打到光源上。
为了避免这种情况，我们可以考虑将直接光照和间接光照分别采样。

如果光源是面光源，我们还是不能逐个计算每一条光线。为此，我们需要对光源也应用蒙特卡罗积分。注意到我们的渲染方程是对 $d\omega$ 进行积分，
而面光源是一个平面，求其面积是对 $dS$ 进行积分。为此我们要将 $d\omega$ 写为立体角的定义 $\frac{dScos\theta'}{||x - x'||^2}$ ，然后再进行积分：

![Screenshot from 2022-01-03 14-49-10.png](https://s2.loli.net/2022/01/03/HfT2arMyxveNiZC.png)   

## 俄罗斯轮盘赌
刚才的ROTK式的光线依然会存在问题：提前停止弹射对结果一定会产生影响。我们如何削减这种影响，同时又不会产生过多的额外开销呢？答案就是使用俄罗斯轮盘赌算法（Russian Roulette, RR）。

我们假定每一束光线都会有一定的概率 $P$ 停止传播，那么也就会有 $1 - P$ 的概率会继续传播。这样为什么能保证效果呢？

假定一个点计算的结果是 $L_o$ ，我们认为光线该点继续弹射的概率为 $P$ ，并且将返回值变为 $\frac{L_o}{P}$ 。此时我们可以分别讨论这个离散随机变量：

+ 光线确实弹射出去了，该部分的期望是 $\frac{L_o}{P} \times P = L_o$
+ 光线没有弹射，此时的期望是 $0$

因此，我们最终求出的光亮度的期望是不变的。最终我们得到的路径追踪伪代码为（图片来自于GAMES101）：

![Screenshot from 2022-01-03 14-39-27.png](https://s2.loli.net/2022/01/03/xb96MnSrVCTsGgY.png)

## 参考资料
+ [GAMES101-现代计算机图形学入门-闫令琪](https://www.bilibili.com/video/BV1X7411F744?p=16)
+ [Ray Tracing in One Weekend](https://raytracing.github.io/books/RayTracingInOneWeekend.html#rays,asimplecamera,andbackground/sendingraysintothescene)
+ [rOtK故事会：热情和斗志已经被消耗殆尽，决定暂时休息一段时间 ](https://www.sohu.com/a/423413074_338155)
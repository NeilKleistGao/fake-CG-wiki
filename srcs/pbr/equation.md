## 微表面模型
PBR模型必须要能满足三个条件，其中第一个便是微表面模型。该模型基于一个事实：世界上并不存在100%光滑的物体，即便是发生镜面反射的镜子，也只能将80%的内容反射回来。

![](https://learnopengl-cn.github.io/img/07/01/microfacets_light_rays.png)

如图所示（图片来自于Learn OpenGL），如果一个平面上的微表面越光滑，那么反射出的光线就会更趋近于同一个方向；反之，微表面越粗糙，
光线就会向各个不同的方向发散开，形成漫反射。

由于微表面过于“微小”，甚至小于一个像素，我们甚至很难用纹理来进行表示。因此，我们不得不舍弃具体的表面法线参数，而是泛泛地使用一个“粗糙度”变量，
来控制微表面法线的分布。我们再次请出半程向量： $\vec h = \frac{\vec L + \vec V}{|\vec L + \vec V|}$ ，如果微表面法线与半程向量方向一致的概率越大，
则说明效果越接近于镜面反射（图片来自于Learn OpenGL）：

![](https://learnopengl-cn.github.io/img/07/01/ndf.png)

常用的微表面模型有Blinn模型和Beckmann模型。Blinn模型的表达式为 $D = ce^{-\frac{\alpha}{m}}$ ，
Beckmann模型的表达式为 $D = \frac{1}{m^2cos^4\alpha}e^{-(\frac{tan\alpha}{m})^2}$ 。
这两个模型中， $\alpha$ 表示法线和半程向量之间的夹角， $m$ 表示粗糙度。

## 能量守恒
能量守恒是19世纪最伟大的发现之一，在图形学上也有重大的应用。为了真实模拟光线，我们规定：**出射光线的能量永远不能超过入射光线的能量和自身发射出的光线的能量**。

我们将光线分为折射部分和反射部分，反射光线不会进入到物体中，这就是镜面光；折射光线会穿过物体（半透明）或干脆在物体内被吸收，最终形成漫反射。

此外，对于金属的表面，所有的折射光会被直接吸收而不会继续散射开；而对于非金属（也叫做电介质，比如玻璃或者蜡），折射光线会继续散开。

## 渲染方程
在介绍PBR的最后一个条件之前，我们需要先介绍一下渲染方程。在说明渲染方程前，我们先介绍一下反射方程。

渲染的本质就是处理光与材质之间的交互，而这些交互中最重要的莫过于反射。对一个像素进行渲染的过程，就等同于从空间中接收来自四面八方的光线，
然后经过一定的计算，得到我们视角方向的反射光线。把这个过程用数学的方式表示出来就是：

$$L_r(p, \omega_r) = \int_{H^2} f_r(p, \omega_i, \omega_r)L_i(p, \omega_i)cos\theta_id\omega_i$$

在这个式子中，我们计算了对于一个点 $p$ ，我们从立体角 $\omega_r$ 看去时我们眼睛所接受的光亮度。这个值等于在该点接受了来自各个不同方向 $d\omega_i$
的光亮度 $L_i(p, \omega_i)$ 后，反射出的结果。其中， $f_r(p, \omega_i, \omega_r)$ 表示光线与材质交互的属性。

这个方程只能表示反射的光线，如果一个物体自身可以发射出光线，这个方程将无法使用。最简单的修正方法就是将发光项添加到方程中：

$$L_o(p, \omega_o) = L_e(p, \omega_o) + \int_{\Omega^{+}} f_r(p, \omega_i, \omega_o)L_i(p, \omega_i)(n \cdot \omega_i)d\omega_i$$

这就是大名鼎鼎的**渲染方程**。现在我们只需要处理 $f_r$ 项即可。

## BRDF
PBR的最后一个条件，就是需要一个函数，能够尽可能真实地表述出入射光线和出射光线的关系。这个函数就被称为**双向反射分布函数（Bidirectional Reflectance Distribution Function， BRDF）**。现在多数实时的渲染管线所使用的是Cook-Torrance BRDF模型。

Cook-Torrance BRDF包含两个部分，分别是漫反射和镜面反射：

$$f_r = k_df_{lambert} + k_sf_{cook−torrance}$$

其中 $f_{lambert} = \frac{c}{\pi}$ ，表示漫反射， $c$ 表示材质本身的颜色。为了遵守能量守恒，我们需要保证 $\int_{\Omega^{+}}f_{lambert}L(n \cdot \omega_i)d\omega_i \le L$ 。我们将积分无关项移出积分外并进行化简 $c\int_{\Omega^{+}}cos\theta d\omega_i \le 1$ 。
将立体角展开，我们可以得到二重积分 $c\int_{0}^{2\pi}\int_{0}^{\frac{\pi}{2}}cos\theta sin\theta d\theta d\phi \le 1$ ，解出不等式我们可以得到 $c \le \frac{1}{\pi}$ 。因此我们需要在漫反射项上除以一个 $\pi$ 。

镜面反射项 $f_{cook−torrance} = \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}$ 包含三个函数，分别是：

+ D：表面分布函数，我们在微表面部分已经处理过了
+ F：菲涅尔项
+ G：几何函数

### 菲涅尔现象
同一个地方的湖面，在不同的位置去观察它，总能看到它在不同位置所呈现的效果是不一样的，近处看可以看到清澈见底的湖水，远处看却是波光粼粼的湖面。
这就是菲涅尔现象的真实体验。掠夺角（视线和平面法线的夹角）越大，那么反射就越强，菲涅尔现象就越明显：

![](https://learnopengl-cn.github.io/img/07/01/fresnel.png)

由于菲涅尔方程本身非常复杂，我们在计算的过程中通常使用resnel-Schlick近似法进行表示：

$$F(h, v, F_0) = F_0 + (1 - F_0)(1 - (h \cdot v))^5$$

其中， $F_0$ 表示材质的基础反射率。你可以在参考资料中找到常见物体的基础反射率。

### 几何函数
几何函数考虑了微表面模型下的自遮挡关系。如果一个物体表面非常粗糙，那么凸起的部分就有可能会挡住其他部分。
与微表面类似地，我们也使用一个概率函数来表示几何遮挡：

$$G(n, v, k) = \frac{n \cdot v}{(n \cdot v)(1 - k) + k}$$

这个函数被称为Schlick-GGX函数，它是几何函数的一种表示。函数中的 $k$ 是关于粗糙度 $\alpha$ 的映射。
对于直接光照来说，  $k = \frac{(\alpha + 1)^2}{8}$ 。对于其他不同的光照，这个映射也会不同。

为了同时考虑光照的遮蔽和观察的遮蔽，我们可以使用史密斯法(Smith’s method)合并二者，即假定两个遮挡相互独立：

$$G(n, v, l, k) = G(n, v, k)G(n, l, k)$$

## 参考资料
+ [PBR理论 - LearnOpenGL](https://learnopengl-cn.github.io/07%20PBR/01%20Theory/)
+ [【1080P+/BD】续 终物语 上卷（1 2 3话）【物语系列圈字幕组】](https://www.bilibili.com/video/BV1wb411z7J4?spm_id_from=333.999.0.0)
+ [微表面模型－PBR渲染管线的材质](https://zhuanlan.zhihu.com/p/25421091)
+ [GAMES101-现代计算机图形学入门-闫令琪](https://www.bilibili.com/video/BV1X7411F744?p=15)
+ [为什么PBR中Lambert光照要除PI?](https://zhuanlan.zhihu.com/p/29837458)
+ [Shader实验室:菲涅尔效应](https://zhuanlan.zhihu.com/p/151375798)
## 模型空间、世界空间、观察空间、裁剪空间和屏幕空间
### 模型空间
每个模型都有自己的局部坐标空间（你可以假定一个正方体模型，在其正中心建立一个坐标系），这个坐标空间被称为模型空间。

模型空间是一个局部的、相对的空间，也是我们建立模型时使用的空间（建模师可不知道这个模型会被摆放到场景中的哪个位置）。
但这个空间下是无法完成渲染的：所有的物体会被堆叠到原点上。此时我们需要建立一个世界空间。

### 世界空间
我们可以建立一个更大的坐标空间，这个空间容纳了我们所有的模型。这个空间被称为世界空间。每个物体在世界空间中都有其放置的坐标，旋转角度，缩放大小。

![Screenshot from 2021-09-19 12-20-16.png](https://i.loli.net/2021/09/19/lhYwm87ORnZJS6z.png)

上图是Unity3D中的Transform组建的各个属性，包括位置，旋转和缩放。如果该对象没有其他父对象，那么Transform中所显示的就是世界空间下的坐标。

### 观察空间
世界空间是一个很自然的表示方法，但它也会给渲染带来不少麻烦。我们的摄像机（或者说观察点）也可以有其自己的世界空间，这样我们的视平面（最终物体投影到的平面）可能与xy平面，xz平面和yz平面均不平行，物体在投射的过程中需要经过大量的计算才能找到对应的像素。

我们有一个美好的设想：我们希望摄像机被放在原点，上方朝向Y轴正半轴，摄像机对着Z轴负半轴。这样的摄像机在渲染过程中很容易处理。
既然我们无法将摄像机的位置固定，我们不妨将摄像机连同整个世界一起变换，放到一个新的坐标空间中。这样，设计师在摆放时使用世界空间，而我们在渲染过程中可以使用一个全新的坐标空间：观察空间。

### 裁剪空间
观察空间中的物体依然不能进行渲染！我们的视野范围是有限的，并不是所有的物体我们都可以看得到（想象一下你在教室里能够同时看到前后两块黑板）。
因此，我们需要把看不到的东西裁剪掉，只保留视野范围内的物体。这个过程我们依然使用变换来完成，变换后的空间被称为裁剪空间。

裁剪空间一般是一个边长为2的正方体，所有的坐标都会被投影到 $[-1, 1]$ 。

### 屏幕空间
裁剪后的图像还需要进行变换，最终才能被显示到屏幕上。裁剪空间中对应的点需要被投影到屏幕中某个坐标上，这样硬件设备才能知道具体应该在哪显示这个像素。
屏幕上的坐标空间被称为屏幕空间。OpenGL中使用`glViewPort`（视口函数）来制定屏幕空间。

这里就是空间变换的终点了！在这个过程中我们经历了五个空间坐标，四次坐标变换。其中又以前三次变换最为复杂和重要，这三次变换被称为MVP变换（模型，观察和投影的缩写）。
接下来，我们首先介绍变换的方法，然后讲解MVP变换的做法。

## 空间中坐标的变换
接下来的部分我们将探讨坐标变换的计算方式。所有的计算都以2D为例，你可以很容易地推广到三维空间。

我们将用到大量线性代数的知识。如果你对这些内容还不是很熟悉，你可以先阅读[线性代数](https://neilkleistgao.github.io/fake-CG-wiki/linear_algebra/)方面的内容。

### 缩放
![Screenshot from 2021-09-19 13-01-48.png](https://i.loli.net/2021/09/19/QynkI42UZ8mqNRO.png)

我们需要将小的正方形放大两倍，变成外面那个更大的正方形。我们以点A到点F为例： $A(2, 2), F(4, 4)$ 。
我们不难发现坐标的变换满足如下的关系： $x' = 2x, y' = 2y$ 。更一般地，假定缩放的系数变为 $m, n$ ，那么变换公式为 $x' = mx, y' = ny$ 。

我们可以将坐标写为列向量的形式，这样变换就可以表示为矩阵乘法：

$$
\begin{bmatrix}
    x' \\ y'
\end{bmatrix}
=
\begin{bmatrix}
    m & 0 \\
    0 & n
\end{bmatrix}
\begin{bmatrix}
    x \\ y
\end{bmatrix}
$$

对于三维空间，你只需要将矩阵扩展为三维即可。

### 旋转
![Screenshot from 2021-09-19 13-24-48.png](https://i.loli.net/2021/09/19/LNZz6aYXIvmiFB8.png)

我们需要将C点旋转到B点，旋转角度为 $\alpha$ 。设向量的长度为 $L$ ，我们可以找到如下的几何关系：

$$
\begin{array}{l}
B_y = Lsin(\alpha + \beta) \\
B_x = Lcos(\alpha + \beta) \\
C_y = Lsin\beta \\
C_x = Lcos\beta
\end{array}
$$

将点B的式子展开后：

$$
\begin{array}{l}
B_y = L(sin\alpha cos\beta + sin\beta cos\alpha) \\
B_x = L(cos\alpha cos\beta - sin\alpha sin\beta)
\end{array}
$$

将点C的坐标带入式子：

$$
\begin{array}{l}
B_y = sin\alpha C_x + cos\alpha C_y \\
B_x = cos\alpha C_X - sin\alpha C_y
\end{array}
$$

那么写成矩阵形式就是：

$$
\begin{bmatrix}
    x' \\ y'
\end{bmatrix}
=
\begin{bmatrix}
    cos\alpha & -sin\alpha \\
    sin\alpha & cos\alpha
\end{bmatrix}
\begin{bmatrix}
    x \\ y
\end{bmatrix}
$$

三维的情况要略微复杂一些，因为我们需要确定一个“旋转轴”，然后才能像二维情况一样进行操作。以xyz分别为旋转轴的旋转矩阵分别如下：

$$
R_x(\alpha) = 
\begin{bmatrix}
    1 & 0 & 0 \\
    0 & cos\alpha & -sin\alpha \\
    0 & sin\alpha & cos\alpha
\end{bmatrix}
$$

$$
R_y(\alpha) = 
\begin{bmatrix}
    cos\alpha & 0 & sin\alpha \\
    0 & 1 & 0 \\
    -sin\alpha & 0 & cos\alpha
\end{bmatrix}
$$

$$
R_z(\alpha) = 
\begin{bmatrix}
    cos\alpha & -sin\alpha & 0 \\
    sin\alpha & cos\alpha & 0 \\
    0 & 0 & 1
\end{bmatrix}
$$

你可以发现，旋转 $\alpha$ 度和 旋转 $-\alpha$ 度的矩阵正好互为转置。也就是说**旋转矩阵的逆矩阵为原矩阵的转置**。

三维空间中的旋转被拆解为以上三个旋转轴上的旋转，旋转的计算需要将三个矩阵乘起来。这种旋转方式被称为欧拉角旋转。

> 可能你已经听说过了万向节死锁和四元数，我们在这个部分暂时不会涉及这方面的内容

除了将三个矩阵相乘，我们还可以使用罗德里格斯旋转公式：

$$
\vec v‘ = cos\alpha \vec v + (1 - cos\alpha)(\vec k \cdot \vec v) + sin\alpha \vec k \times \vec v
$$

其中 $\vec k$ 为单位旋转轴。

我们将 $\vec v$ 投影到 $\vec k$ 上，这样我们就有两个向量：一个与 $\vec k$ 同直线，另一个与 $\vec k$ 正交：

$$
\vec v_{\parallel} = (\vec v \cdot \vec k)\vec k \\
\vec v_{\perp} = \vec v - \vec v_{\parallel} = -\vec k \times (\vec k \times \vec v)
$$

由于 $\vec k \times \vec v = \vec k \times \vec v_{\perp}$ ，我们再叉乘一个 $\vec k$ 得到正交的分量。
此外，共线的向量不会受到旋转的影响，我们只需要考虑正交的分量。

以 $\vec v_{\perp}, \vec k \times \vec v, \vec k$ 为局部坐标系，旋转后的正交分量为： $cos\alpha \vec v_{\perp} + sin\alpha \vec k \times \vec v$ 。
将其与平行分量相加即可得到上面的公式。你也可以通过将叉乘写为矩阵形式，从而得到旋转矩阵。

### 平移
![Screenshot from 2021-09-19 15-03-52.png](https://i.loli.net/2021/09/19/6znPGmcBldq8QhY.png)

平移其实是最简单的变换，你甚至不需要经过过多的思考就可以得出下面的公式：

$$
\begin{array}{l}
x' = x + dx \\
y' = y + dy
\end{array}
$$

但是平移的问题在于：**上述的公式你无法用二维的矩阵进行表示**。这种不统一会使得我们在进行变换的过程中难以操作，计算效率低下。
为此，我们引入了齐次坐标。此时二维平面内的一个点被写为 $[x, y, 1]$ 。这样我们就可以使用矩阵乘法计算平移了：

$$
\begin{bmatrix}
    x' \\ y' \\ 1
\end{bmatrix}
=
\begin{bmatrix}
    1 & 0 & dx \\
    0 & 1 & dy \\
    0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
    x \\ y \\ 1
\end{bmatrix}
$$

除了允许我们用更统一的形式完成平移操作外，齐次坐标还有很多好处：

+ 当齐次项为0时，平移矩阵无法产生影响，我们可以方便地用这种方式区分点和向量
+ 我们可以定义坐标的加法，在这个定义下，向量加向量还是向量，点加向量还是点，符合我们的预期
+ 只要齐次项非0,我们都可以用它来表示一个点，如 $[6, 12, 3]$ 实际表示的坐标是 $[2, 4]$ 。这个步骤我们称为透视除法。这样我们也能定义点和点的相加，得到的结果是两个点的终点

## MVP变换
### 模型变换
模型变换需要将坐标从模型空间转换到世界空间。由于每个模型都有其在世界空间中的位置，缩放大小和旋转角度，只需要将相关矩阵相乘即可。
需要注意模型锚点的位置！如果需要对模型进行旋转和缩放，需要先将模型的锚点平移到原点，然后才能进行旋转和缩放。

### 观察变换
观察变换需要将相机连带整体场景变换到原点，并指向特殊的方向。我们定义摄像机的三个变量： `position`， `lookat` 和 `vup`,分别表示摄像机的位置，镜头方向和向上朝向。

平移部分只需要一个平移矩阵即可完成，我们重点讨论旋转。将 $\vec {x'} = \vec {lookat} \times \vec {vup}$ ， $\vec {y'} = \vec {vup}$ 和 $\vec {z'} = \vec {lookat}$ 旋转到 $\vec x, \vec y, \vec z$ 并不容易思考。
但我们可以使用逆向思维：

$$
\begin{bmatrix}
    \vec {x'}_x & \vec {x'}_y & \vec {x'}_z \\
    \vec {y'}_x & \vec {y'}_y & \vec {y'}_z \\
    \vec {z'}_x & \vec {z'}_y & \vec {z'}_z
\end{bmatrix}
\begin{bmatrix}
    1 & 0 & 0 \\
    0 & 1 & 0 \\
    0 & 0 & 1
\end{bmatrix}
=
\begin{bmatrix}
    \vec {x'}_x & \vec {x'}_y & \vec {x'}_z \\
    \vec {y'}_x & \vec {y'}_y & \vec {y'}_z \\
    \vec {z'}_x & \vec {z'}_y & \vec {z'}_z
\end{bmatrix}
$$

我们现在只需要求出该矩阵的逆矩阵即可。根据上面旋转矩阵的推论，我们只需要将这个矩阵转置：

$$
\begin{bmatrix}
    \vec {x'}_x & \vec {x'}_y & \vec {x'}_z \\
    \vec {y'}_x & \vec {y'}_y & \vec {y'}_z \\
    \vec {z'}_x & \vec {z'}_y & \vec {z'}_z
\end{bmatrix}^T
=
\begin{bmatrix}
    \vec {x'}_x & \vec {y'}_x & \vec {z'}_x \\
    \vec {x'}_y & \vec {y'}_y & \vec {z'}_y \\
    \vec {x'}_z & \vec {y'}_z & \vec {z'}_z
\end{bmatrix}
$$

### 投影变换
投影的方式分为两种：正交投影和透视投影。正交投影相对简单，因为正交投影的视锥体是一个长方体：

![Screenshot from 2021-09-20 15-26-36.png](https://i.loli.net/2021/09/20/1O5Bf4V3Tl7gYEu.png)

我们需要将这个长方体压缩到边长为2,且中心在原点。我们很容易可以写出这个矩阵：

$$
\begin{bmatrix}
    \frac{2}{r - l} & 0 & 0 & -\frac{r + l}{r - l} \\
    0 & \frac{2}{t - b} & 0 & -\frac{t + b}{t - b} \\
    0 & 0 & -\frac{2}{f - n} & -\frac{f + n}{f - n} \\
    0 & 0 & 0 & 1
\end{bmatrix}
$$

这个矩阵由两部分组成：首先将长方体中心平移到原点，然后再进行缩放操作。

比较麻烦的是透视投影。由于透视投影的视锥体是一个棱台，我们需要将其压缩为一个长方体，然后再按照正交投影的方式进行计算。

我们从侧面来观察视锥体（向X轴负半轴看去）：

![Screenshot from 2021-09-20 15-37-33.png](https://i.loli.net/2021/09/20/ClfdNHevu6XEKDg.png)

假定我们在视锥体边缘上取出某个点，我们可以得到他的Y轴和Z轴的坐标。我们需要知道“压缩”后，这个点的Y轴坐标会变成多少。 
注意到场景中存在一个相似三角形的关系： $\frac{n}{z} = \frac{y'}{y}$ ，我们可以得到 $y' = \frac{n}{z}y$ ，同理也可以得到 $x' = \frac{n}{z}x$ 。

现在我们可以填出矩阵的一部分：

$$
\begin{bmatrix}
    \frac{n}{z} & 0 & 0 & 0 \\
    0 & \frac{n}{z} & 0 & 0 \\
    ? & ? & ? & ? \\
    0 & 0 & 0 & 1
\end{bmatrix}
$$

由于我们使用齐次坐标，我们的变换也可以写为这个形式：

$$
\begin{bmatrix}
    n & 0 & 0 & 0 \\
    0 & n & 0 & 0 \\
    ? & ? & ? & ? \\
    0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
    x \\ y \\ z \\ 1
\end{bmatrix}
=
\begin{bmatrix}
    nx \\ ny \\ ? \\ z
\end{bmatrix}
$$

我们需要求出剩下的未知数。对于近平面上的点，无论如何变换，其Z值是不会变化的。于是我们可以得到：

$$
\begin{bmatrix}
    n & 0 & 0 & 0 \\
    0 & n & 0 & 0 \\
    0 & 0 & A & B \\
    0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
    x \\ y \\ n \\ 1
\end{bmatrix}
=
\begin{bmatrix}
    nx \\ ny \\ n^2 \\ n
\end{bmatrix}
$$

由于 $n^2$ 与 $x, y$ 无关，我们可以将前两项补为0。并设最后的未知数为 $A, B$ 。
除了近平面上的点以外，远屏幕上的中心点也不会发生任何变化。我们得到方程组：

$$
\begin{array}{l}
An + B = n^2 \\
Af + B = f^2
\end{array}
$$

解得： $A = n + f, B = -nf$ 。综上，我们得到的透视矩阵为：

$$
\begin{bmatrix}
    n & 0 & 0 & 0 \\
    0 & n & 0 & 0 \\
    0 & 0 & n + f & -nf \\
    0 & 0 & 0 & 1
\end{bmatrix}
$$

## 参考资料
+ [Learn OpenGL CN](https://learnopengl-cn.github.io/)
+ [GAMES101-现代计算机图形学入门-闫令琪](https://www.bilibili.com/video/BV1X7411F744?p=3)
+ [罗德里格斯旋转公式（Rodrigues' rotation formula）推导](https://www.cnblogs.com/wtyuan/p/12324495.html)
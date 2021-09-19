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
### 观察变换
### 投影变换
## 参考资料
+ [Learn OpenGL CN](https://learnopengl-cn.github.io/)
+ [GAMES101-现代计算机图形学入门-闫令琪](https://www.bilibili.com/video/BV1X7411F744?p=3)
+ [罗德里格斯旋转公式（Rodrigues' rotation formula）推导](https://www.cnblogs.com/wtyuan/p/12324495.html)
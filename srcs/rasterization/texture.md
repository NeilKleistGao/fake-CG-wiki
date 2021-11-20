## 什么是纹理
我们渲染出的模型到现在为止只有一个骨架（即网格），即便我们为他上色，也只是简单的颜色变换。虽然我们可以进行复杂的计算，从而让他实现逼真的效果，但是这样的计算量是非常巨大的。
如果我们可以像画家一样，在这些模型的表面“画”出我们想要的效果（而不是“算”出），我们能节约不少时间和人力。不过，这样的花纹保存在3D模型上非常麻烦，我们不妨将这些花纹“撕下来”，
保存在一张2D图像中。当我们需要显示的时候，再将这些图画粘贴回去。这些2D图像就是纹理（又称贴图，材质贴图）。

为了把纹理粘贴回模型上，我们需要指定网格上每一个点对应的纹理坐标。这些纹理坐标通常用uv表示（和空间坐标的xyz区分开来），为了适应不同的纹理大小，这些uv坐标的取值范围通常在 $[0, 1]$ 之间。
对于大小为 $w \times h$ 的纹理，我们会在 $[w \times u, h \times v]$ 点处进行采样，采样结果被用于该点的着色。

如果我们真的给每个点都指定一个uv坐标的话，这就和直接指定每个点的颜色没有太大区别了。通常来说，我们只给定网格顶点处的uv坐标（通常是三角形或四边形的网格，多边形的网格可以拆分为三角形），而其他内部点的uv坐标我们可以通过插值来完成。

## 三角形重心坐标
给定一个三角形的三个顶点的uv坐标，我们如何才能知道三角形内部任意一个点的uv坐标？如果三角形内的点都可以由三个顶点经过某种运算得到，我们就能想办法用同样的方法计算uv坐标。

![Screenshot from 2021-11-20 13-18-05.png](https://i.loli.net/2021/11/20/3AdoOWknqZCD8yL.png)

我们不妨将三角形平移，使得点A与原点重合：

![Screenshot from 2021-11-20 13-23-41.png](https://i.loli.net/2021/11/20/MiQ8YdlBCetpOZq.png)

由于向量 $\vec{AB}, \vec{AC}$ 不共线，这两个向量一定可以作为二维空间中的一组基。
由于三角形内的点也一定在平面内，我们就可以将点表示为 $\beta\vec{AB} + \gamma\vec{AC}$ 。只要我们保证 $\beta \ge 0, \gamma \ge 0$ ，
所有的点就都会落在 $\vec{AB}, \vec{AC}$ 之间。我们只需要再增加一个约束，即所有的点都在 $\vec{CB}$ 的左侧，就能表示出所有三角形内点。

![Screenshot from 2021-11-20 13-44-56.png](https://i.loli.net/2021/11/20/KtJDqdx6caR9Cmz.png)

对于 $\vec{CB}$ 上任意一点D，我们若要表示 $\vec{AD}$ ，我们可以简单地写成 $\vec{AC} + \alpha\vec{CB} = \vec{AC} + \alpha(\vec{AB} - \vec{AC})$ ，
此时该点就可以表示为 $\vec{AB}, \vec{AC}$ 的插值： $\beta\vec{AB} + \gamma\vec{AC}$ ，其中 $\beta + \gamma = 1$ 。如果需要点落在三角形内，
只需将条件修改为 $\beta + \gamma \le 1$ 。

此时我们再将三角形平移回原来的位置即可。三角形内的点均可以写为 $\beta\vec{AB} + \gamma\vec{AC}$ ，将向量拆开写作三个点的坐标，
我们可以得到 $\beta B - \beta A + \gamma C - \gamma A$ 。为了方便，我们令 $\alpha = 1 - \beta - \gamma$ ，
这样三角形内的点就可以表示为 $\alpha A + \beta B + \gamma C$ ，其中 $\alpha \ge 0, \beta \ge 0, \gamma \ge 0, (\alpha + \beta + \gamma) = 1$ 。

对于平面内任意的一个点 $(x, y)$ 我们都能算出对应的 $\alpha, \beta, \gamma$ ，虽然看似有三个未知数，而我们只能列出两个方程，但我们可以用 $(\alpha + \beta + \gamma) = 1$ 消掉其中一维：

$$
\begin{array}{c}
x = (1 - \beta - \gamma)A_x + \beta B_x + \gamma C_x \\
y = (1 - \beta - \gamma)A_y + \beta B_y + \gamma C_y
\end{array}
$$

解出上面的方程，我们可以得到：

$$
\begin{array}{c}
\alpha = 1 - \beta - \gamma \\
\beta = \frac{(x - A_x)(C_y - A_y) - (C_x - A_x)(y - A_y)}{(C_y - A_y)(B_x - A_x) - (C_x - A_x)(B_y - A_y)} \\
\gamma = \frac{(y - A_y)(B_x - A_x) - (x - A_x)(B_y - A_y )}{(C_y - A_y)(B_x - A_x) - (C_x - A_x)(B_y - A_y)}
\end{array}
$$
 
## 其他应用
### 光照纹理
我们在基本光照模型中提到了光照的计算方式，其中我们认为材质的每一个点的高光行为都是一样的。这其实并不合理。想象一个穿着铠甲的战士，
身上的铠甲由于是金属，其高光反射一定是要强于战士的手或是脸的。

为了实现不同位置的不同光照行为，我们可以给材质指定专门的光照贴图（图片来自于LearnOpenGL）：

![](https://learnopengl-cn.github.io/img/02/04/container2_specular.png)

如果在某个位置我们不需要高光，我们只需要将该点的光照贴图设置为全黑（也就是RGB均为0），这个点的高光系数就会变为全0，从而没有高光行为。

### 法线纹理
我们制作的模型的表面可能非常的光滑，如果我们希望显示出一种粗糙的感觉，我们需要不断增加模型的面数，从而模拟真正的不平滑的感觉。
但是模型面数上升的代价是极具的性能降低，这不是我们所期望的。

一个很好的折中的方案是使用纹理修改模型默认的法线，从而在计算光照的时候，生成一些本不应该存在的阴影（图片来自于LearnOpenGL，下同）：

![](https://learnopengl-cn.github.io/img/05/04/normal_mapping_compare.png)

这样的纹理被称为法线纹理（也叫法线贴图，凹凸贴图）。

我们在法线纹理中可以直接存储目标法线值，但这样会存在一些问题：法线值是绝对值，难以复用。比如我希望在另一个模型上复用这个法线纹理，但由于我表面发生了一些变化，法线就会出错。
我们可以通过存储法线的相对值来解决这个问题。

我们尝试重新建立一个局部坐标系，这个坐标系以当前位置原本的法线方向为z轴，以这个点的切线和副切线为x轴和y轴，这个空间被称为切线空间。由于大部分变换后的法线不会和原本的法线有太大的出入，所以切线控件下的法线贴图色调偏蓝。

一个点的切线可以延许多方向展开。为了方便计算，我们不妨认为切线空间的xy平面与法线贴图的uv是对齐的：

![](https://learnopengl-cn.github.io/img/05/04/normal_mapping_tbn_vectors.png)

我们将当前点当作原点，开始建立坐标系。每一个顶点都会连接三角形的两条边（记作 $E_1, E_2$ ）。由于他们和切线( $T$ )、副切线( $B$ )在同一平面内，
我们可以用这两个向量将其他向量表出：

$$
\begin{array}{c}
\vec E_1 = \Delta U_1 \vec T + \Delta V_1 \vec B
\vec E_2 = \Delta U_2 \vec T + \Delta V_2 \vec B
\end{array}
$$

我们可以将这个式子写为矩阵表示：

$$
\begin{bmatrix}
    E_{1x} & E_{1y} & E_{1z} \\
    E_{2x} & E_{2y} & E_{2z}
\end{bmatrix}
=
\begin{bmatrix}
    \Delta U_1 & \Delta V_1 \\
    \Delta U_2 & \Delta V_2
\end{bmatrix}
\begin{bmatrix}
    T_x & T_y & T_z \\
    B_x & B_y & B_z
\end{bmatrix}
$$

只需要在等式的两侧同时左乘UV矩阵的逆矩阵，我们就可以建立这样的切线空间。

## 参考资料
+ [材质贴图](https://zh.wikipedia.org/wiki/%E6%9D%90%E8%B4%A8%E8%B4%B4%E5%9B%BE)
+ [GAMES101-现代计算机图形学入门-闫令琪](https://www.bilibili.com/video/BV1X7411F744?p=8)
+ [LearnOpenGL-纹理](https://learnopengl-cn.github.io/02%20Lighting/04%20Lighting%20maps/)
+ [LearnOpenGL-光照贴图](https://learnopengl-cn.github.io/02%20Lighting/04%20Lighting%20maps/)
+ [LearnOpenGL-法线贴图](https://learnopengl-cn.github.io/05%20Advanced%20Lighting/04%20Normal%20Mapping/)
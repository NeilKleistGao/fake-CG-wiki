## Bresenham光栅化算法
我们先从最简单的画线开始。

绘制一条直线需要确定两个点的坐标。在两个点之间，我们可以认为存在一个向量。向量的方向不会改变，我们通过调整向量的大小来确定中间的点。
当大小为0时，这个点位于 $(x_0, y_0)$ ；当大小为原大小时，这个点位于 $(x_1, y_1)$ ：

```c++
void draw() {
    glColor3f(1.0f, 1.0f, 1.0f);
    glm::vec2 v0{100, 200}, v1{500, 600};
    glBegin(GL_POINTS);
    {
        for (float t = 0.0f; t <= 1.0f; t += 0.01f) {
            glVertex2f((v0.x + (v1.x - v0.x) * t - 400.0f) / 400.0f,
             (v0.y + (v1.y - v0.y) * t - 300.0f) / 300.0f);
        }
    }
    glEnd();
}
```

效果如下：

![Screenshot from 2021-09-21 16-31-20.png](https://i.loli.net/2021/09/21/BaOWVwYGRuJExym.png)

我们确实画出了一条线，但是问题在于：这条直线很明显不连续。这不是我们希望的。我们希望改进这个算法。

由于我们需要使用浮点数来计算中间的点，而屏幕上每个点是离散的，所以误差固然存在。我们希望最终算出一个浮点数，能找到一个距离它最近的整数，在这个位置画点，而不是盲目地向下取整。

我们每次保证x向前进1，再去考虑y的变化。如果增长的误差值达到了0.5，说明我们可以开始画下一个点了。这样我们就有了第二版算法，也就是Bresenham光栅化算法（也叫Bresenham直线算法，布雷森汉姆直线算法）：

```c++
void draw() {
    glColor3f(1.0f, 1.0f, 1.0f);
    glm::vec2 v0{0, 0}, v1{200, 100};
    glBegin(GL_POINTS);
    {
        int dx = v1.x - v0.x;
        int dy = v1.y - v0.y;
        float err = 0.0f;
        float dlt = (float)dy / dx;

        int y = v0.y;
        for (int i = v0.x; i <= v1.x; ++i) {
            glVertex2f(i / 400.0f , y / 300.0f);
            err += dlt;

            if (err >= 0.5f) {
                ++y;
                err -= 1.0f;
            }
        }
    }
    glEnd();
}
```

效果：

![Screenshot from 2021-09-21 16-32-46.png](https://i.loli.net/2021/09/21/WFugNxEinyO9GIq.png)

当然，这不是一个完美的算法。首先对于斜率不存在的直线，你必须进行特殊判断，否则将会造成除0的错误。此外，这个算法在斜率大于1时表现仍然可能不佳：

```c++
glm::vec2 v0{0, 0}, v1{10, 300};
```

这本该是一条直通天际的直线，但是我们绘制后会发现：

![Screenshot from 2021-09-21 16-38-08.png](https://i.loli.net/2021/09/21/49vDm5OUjC7fWP3.png)

由于`dlt`太大，我们的点还是会出现不连续甚至消失的情况。如果斜率大于1,我们需要遍历y而不是x：

```c++
glColor3f(1.0f, 1.0f, 1.0f);
glm::vec2 v0{0, 0}, v1{10, 300};
glBegin(GL_POINTS);
{
    int dx = v1.x - v0.x;
    int dy = v1.y - v0.y;
    bool flag = false;
    if (std::abs(dy) > std::abs(dx)) {
        std::swap(v0.x, v0.y);
        std::swap(v1.x, v1.y);
        std::swap(dx, dy);
        flag = true;
    }

    float err = 0.0f;
    float dlt = (float)dy / dx;

    int y = v0.y;
    for (int i = v0.x; i <= v1.x; ++i) {
        if (flag) {
            glVertex2f(y / 300.0f, i / 400.0f);
        }
        else {
            glVertex2f(i / 400.0f , y / 300.0f);
        }

        err += dlt;

        if (err >= 0.5f) {
            ++y;
            err -= 1.0f;
        }
    }
}
glEnd();
```

这样我们就能看到一条完整的直线了：

![Screenshot from 2021-09-21 16-43-24.png](https://i.loli.net/2021/09/21/Nkb9Piv6BLWCm7g.png)

此外，由于浮点运算的低效，你也可以将里面的浮点运算全部乘上`dx`，转化为整形运算，从而提高效率。

## 三角形光栅化
我们接下来开始进行画三角形的工作。

其实我们可以选择的形状有很多：三角形，四边形，甚至六边形等等，为什么一定要选择三角形来作为基本单位？这是因为：

+ 三角形是最简单的多边形
+ 所有的多边形都可以拆分为三角形
+ 保证所有点都在同一平面内
+ 内外分明，不会有洞
+ 易于插值

所有的这些性质让我们更倾向于使用三角形来表示模型。

### 扫描线算法

![Screenshot from 2021-09-23 20-07-53.png](https://i.loli.net/2021/09/23/QXVEA8FwWonNdJ2.png)

扫描线的思路非常简单：假定我有一条水平的直线（如图），我使用这条直线扫过三角形，判断线上某些点是否在三角形内，然后对在三角形内的部分执行下一步操作。

以 $\triangle ABC$ 为例：最开始的直线落在了B点上，此时需要渲染的点是非常明晰的（或者反过来落在线段AC上，我们也能知道要渲染的范围）。随着直线不断向下扫描，我们可以不断更新这个范围，
从而将整个三角形都渲染出来。稍复杂一点的情况类似于 $\triangle DEF$ ：当直线落到点E的时候，我们需要切换其中一侧的直线，才能继续渲染：

```c++
void draw() {
    glColor3f(1.0f, 1.0f, 1.0f);
    glm::vec2 v0{0, 0}, v1{100, 20}, v2{50, 100};
    glBegin(GL_POINTS);
    {
        // 按y顺序排好
        if (v0.y > v1.y) {
            std::swap(v0, v1);
        }
        if (v0.y > v2.y) {
            std::swap(v0, v2);
        }
        if (v1.y > v2.y) {
            std::swap(v1, v2);
        }

        // 先计算低的一半
        for (int i = v0.y; i <= v1.y; ++i) {
            // 两条边的t值
            float t1 = (i - v0.y) / (v1.y - v0.y + 1);
            float t2 = (i - v0.y) / (v2.y - v0.y + 1);

            // 当前扫描扫到的边界点
            auto vt1 = v0 + (v1 - v0) * t1;
            auto vt2 = v0 + (v2 - v0) * t2;
            if (vt1.x > vt2.x) {
                std::swap(vt1, vt2);
            }

            for (int j = vt1.x; j <= vt2.x; ++j) {
                glVertex2f(j / 400.0f, i / 300.0f);
            }
        }

        // 同理处理另一半
        for (int i = v1.y; i <= v2.y; ++i) {
            float t1 = (i - v1.y) / (v2.y - v1.y + 1);
            float t2 = (i - v0.y) / (v2.y - v0.y + 1);

            auto vt1 = v1 + (v2 - v1) * t1;
            auto vt2 = v0 + (v2 - v0) * t2;
            if (vt1.x > vt2.x) {
                std::swap(vt1, vt2);
            }

            for (int j = vt1.x; j <= vt2.x; ++j) {
                glVertex2f(j / 400.0f, i / 300.0f);
            }
        }
    }
    glEnd();
}
```

效果：

![Screenshot from 2021-09-23 20-32-09.png](https://i.loli.net/2021/09/23/IyhaCbTt6EGxQL2.png)

### 判断点是否在三角形内
上面的算法固然可行，但是实现复杂，并且不方便执行并行化：我首先需要计算出直线和三角形两边的交点，然后才可以进行渲染，甚至可能会涉及多次坐标值的判断和交换。
我们希望对所有点一视同仁：如果这个点在三角形中，那么渲染，否则抛弃。

回顾[线性代数](https://neilkleistgao.github.io/fake-CG-wiki/linear_algebra/)部分：我们可以利用两个点的叉乘来判断点是否在三角形内：

![Screenshot from 2021-09-23 20-40-19.png](https://i.loli.net/2021/09/23/O7dFwSzArq4uyE9.png)

如图，点D在 $\triangle ABC$ 内，而点E在 $\triangle ABC$ 外。我们先看点D：

+ $\vec {CB} \times \vec {CD} > 0$ （这里大于0指在右手定则下叉乘的结果指向屏幕外）
+ $\vec {BA} \times \vec {BD} > 0$
+ $\vec {AC} \times \vec {AD} > 0$

不难发现：所有的叉乘结果都是大于0的，这样我们就可以认为：这个点在三角形的内部。相反，我们看看点E：

+ $\vec {AC} \times \vec {AE} > 0$
+ $\vec {CB} \times \vec {CE} < 0$
+ $\vec {BA} \times \vec {BE} > 0$

此时存在一项叉乘的符号与其他两项不同，我们就可以认定点E是在三角形的外部。

```c++
void draw() {
    glColor3f(1.0f, 1.0f, 1.0f);
    glm::vec2 v0{0, 0}, v1{100, 20}, v2{50, 100};
    // 假定所有点按照顺时针给出，这样我们只需要判断是否有小于0的情况
    glBegin(GL_POINTS);
    {
        int min_x = std::min(std::min(v0.x, v1.x), v2.x);
        int max_x = std::max(std::max(v0.x, v1.x), v2.x);
        int min_y = std::min(std::min(v0.y, v1.y), v2.y);
        int max_y = std::max(std::max(v0.y, v1.y), v2.y);

        auto e1 = v1 - v0, e2 = v2 - v1, e3 = v0 - v2;
        for (int i = min_x; i <= max_x; ++i) {
            for (int j = min_y; j <= max_y; ++j) {
                auto p = glm::vec2{i, j};
                auto t1 = p - v0, t2 = p - v1, t3 = p - v2;
                if (e1.x * t1.y - e1.y * t1.x >= 0.0f &&
                        e2.x * t2.y - e2.y * t2.x >= 0.0f &&
                        e3.x * t3.y - e3.y * t3.x >= 0.0f) {
                    glVertex2f(i / 400.0f, j / 300.0f);
                }
            }
        }
    }
    glEnd();
}
```

### 重心公式与插值
上面的方法很不错，但是我们在确定这些像素后，还需要对他们进行插值。我们在指定如纹理坐标等数据的时候，无法详细到给每一个像素都指定一遍。
一般地，三角形的每个顶点会被赋予一个纹理坐标。我们需要将三角形中的点，表示为三个顶点的线性组合，这样我们才能计算每个像素的纹理坐标并进行纹理采样。

![Screenshot from 2021-09-25 09-20-34.png](https://i.loli.net/2021/09/25/RKNFTW4erhgwEzD.png)

我们将三角形的某个顶点平移到原点，可以看到AB和AC并不共线。我们以这两条边所在的直线为轴，建立新的坐标系。此时D的坐标就可以被表示为： $\alpha \vec {AB} + \beta \vec {AC}$ 。要保证点D在三角形内部，我们首先要保证 $\alpha \ge 0, \beta \ge 0$ 。此外，点D不能越过线段BC，而BC上的点均可以写为 $\alpha \vec {AB} + \beta \vec {AC}$ ，其中 $\alpha + \beta = 1$ 。我们将三角形平移回原来的位置，再将上面的式子展开： $\alpha \vec {AB} + \beta \vec {AC} = \alpha(B - A) + \beta(C - A) = \alpha B + \beta C + (1 - \alpha + \beta)A$ 。 其中 $\alpha \ge 0, \beta \ge 0, \alpha + \beta \le 1$ 。这就是三角形的重心公式。

我们可以将任意一个点带入方程（两个未知数，x和y各可以列一个方程），解出 $\alpha$ 和 $\beta$ ，如果这两个变量满足要求，则说明这个点在三角形的内部，并且这两个值可以直接用于插值；反之，这个点就在三角形外。


```c++
void draw() {
    glColor3f(1.0f, 1.0f, 1.0f);
    glm::vec2 v0{0, 0}, v1{100, 20}, v2{50, 100};
    // 假定所有点按照顺时针给出，这样我们只需要判断是否有小于0的情况
    glBegin(GL_POINTS);
    {
        int min_x = std::min(std::min(v0.x, v1.x), v2.x);
        int max_x = std::max(std::max(v0.x, v1.x), v2.x);
        int min_y = std::min(std::min(v0.y, v1.y), v2.y);
        int max_y = std::max(std::max(v0.y, v1.y), v2.y);

        for (int i = min_x; i <= max_x; ++i) {
            for (int j = min_y; j <= max_y; ++j) {
                float u = (-(static_cast<float>(i) - v1.x) * (v2.y - v1.y) + (static_cast<float>(j) - v1.y) * (v2.x - v1.x))
                          / (-(v0.x - v1.x) * (v2.y - v1.y) + (v0.y - v1.y) * (v2.x - v1.x)),
                        v = (-(static_cast<float>(i) - v2.x) * (v0.y - v2.y) + (static_cast<float>(j) - v2.y) * (v0.x - v2.x))
                            / (-(v1.x - v2.x) * (v0.y - v2.y) + (v1.y - v2.y) * (v0.x - v2.x)),
                        w = 1.0f - u - v;

                if (u >= 0.0f && v >= 0.0f && w >= 0.0f) {
                    glVertex2f(static_cast<float>(i) / 400.0f, static_cast<float>(j) / 300.0f);
                }
            }
        }
    }
    glEnd();
}
```

## 背面剔除
有的时候我们并不是每个三角形都需要进行这样的操作。显而易见的，三角形越多，我们渲染模型就需要更长的时间。我们会选择一些三角形并将他们忽略掉，这些三角形一般是背对我们的三角形。

如何定义三角形是否正面面对摄像机？我们可以根据叉乘计算出该三角形的法线向量，并用点乘判断法线向量与摄像机`lookat`向量之间的关系。如果两个向量点乘为负，说明三角形面向摄像机那一侧，那这个三角形就可以被保留。

## 参考资料
+ [tinyrenderer](https://github.com/ssloy/tinyrenderer/wiki)
+ [布雷森汉姆直线算法 - 维基百科](https://zh.wikipedia.org/wiki/%E5%B8%83%E9%9B%B7%E6%A3%AE%E6%BC%A2%E5%A7%86%E7%9B%B4%E7%B7%9A%E6%BC%94%E7%AE%97%E6%B3%95)
+ [GAMES101-现代计算机图形学入门-闫令琪](https://www.bilibili.com/video/BV1X7411F744?p=5)
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

## 扫描线算法

## 重心公式与插值

## 背面剔除

## 参考资料
+ [tinyrenderer](https://github.com/ssloy/tinyrenderer/wiki)
+ [布雷森汉姆直线算法 - 维基百科](https://zh.wikipedia.org/wiki/%E5%B8%83%E9%9B%B7%E6%A3%AE%E6%BC%A2%E5%A7%86%E7%9B%B4%E7%B7%9A%E6%BC%94%E7%AE%97%E6%B3%95)
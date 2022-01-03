## 摩尔纹
摩尔纹（也叫莫列波纹）是一种高频干扰条纹。摩尔纹没有明显的形状规律（图片来自于GAMES101，下同）：

![Screenshot from 2021-11-21 11-11-13.png](https://i.loli.net/2021/11/21/QCBxUuRZfbS2rzE.png)

摩尔纹的成因和锯齿很类似：由于在距离较远的位置，一个像素点可能覆盖了大量的uv坐标，但是事实上我们只对其中一个点进行了采样：

![Screenshot from 2021-11-21 11-16-55.png](https://i.loli.net/2021/11/21/tkboulaz1v6wQWm.png)

此时的采样频率跟不上信号频率，导致了高频信号恢复错误，所以出现了显示的失真。

## MIPMAP
为了解决采样不足的问题，我们不妨对图像进行模糊操作（即滤波操作），从而过滤掉其中的高频信号，其中每一幅图像的长宽都是上一幅的一半：

![Screenshot from 2021-11-21 11-19-33.png](https://i.loli.net/2021/11/21/xNfZIivt7KRYDhe.png)

有了滤波后的纹理，我们需要计算某个像素点在渲染的过程中，应该使用哪一张MIPMAP，这样我们才能做到对一个点的范围查询。

不妨设我们需要查询的点为 $(x, y)$ ，那么该点临近的两个点分别为 $(x + 1, y)$ 和 $(x, y + 1)$ 。
我们求出 $(x, y)$ 对应的uv坐标 $(u, v)$ 到另外两个点uv坐标的距离的最大值： $L = max(\sqrt{\Delta u_1^2 + \Delta v_1^2}, \sqrt{\Delta u_2^2 + \Delta v_2^2})$ ，
并利用这个最大值作为边长，建立一个正方形区域，用于近似该点实际覆盖的纹理范围：

![Screenshot from 2021-11-21 16-13-02.png](https://i.loli.net/2021/11/21/Stg8K1oOVPdhl6M.png)

由于在第D层的MIPMAP中，每一个单元格的大小为 $2^D$ 像素（当 $D = 0$ 时，每格的大小恰好为1像素，即原图），那么我希望取一个 $D$ ，使得 $2^D = L$ ，
此时我们就能得出：正方形边长为 $L$ 时，应该使用 $D = log_2^L$ Level的MIPMAP。

由于每次计算下一级的MIPMAP时，长宽各缩小到原来的一半，我们可以求出使用MIPMAP后所需要的空间大小： $1 + \frac{1}{4} + \frac{1}{16} + \dots$ 。
这是一个等比级数，我们可以写出一共有 $D$ 层MIPMAP时这个级数的结果： $\frac{1 \times (1 - (\frac{1}{4})^D)}{1 - \frac{1}{4}}$ 。
当 $D$ 趋于无穷时，该式子的结果是 $\frac{4}{3}$ ，也就是说，开启MIPMAP比起原先最多使用额外的 $\frac{1}{3}$ 的资源。
 

## 参考资料
+ [莫列波纹 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E8%8E%AB%E5%88%97%E6%B3%A2%E7%B4%8B)
+ [GAMES101-现代计算机图形学入门-闫令琪](https://www.bilibili.com/video/BV1X7411F744?p=9)
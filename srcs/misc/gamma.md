## 灰度系数
人眼对于亮度的感知并不是线性的，你对黑暗的感知能力远超过对光亮的感知：你可以在黑暗的房间中摸索，但你很难直视太阳。
你可以从这幅图中感受到这一点（图片来自于LearnOpenGL）：

![](https://learnopengl-cn.github.io/img/05/02/gamma_correction_brightness.png)

不难发现：人所能感知到的亮度是要低于实际的物理亮度的。
我们使用灰度系数（即 $\gamma$ ）来描述物理亮度和实际感受到的亮度之间的关系。我们所实际感受到的亮度通常是物理亮度的 $\gamma$ 次幂，
对于CRT（阴极射线管显示器）而言， $\gamma = 2.2$ ，这表示设备输出的亮度是输入电压的 $2.2$ 次幂：

![Screenshot from 2021-11-22 09-58-45.png](https://i.loli.net/2021/11/22/v3P5qxl81W9bfUD.png)

在CRT中，为了显示出真实的物理亮度，我们需要更高的电压才能实现。这种特性与人眼的感光是类似的。虽然这种机制有助于在黑暗中保护我们，
但却会影响渲染的效果，导致一些细节的丢失。为此我们需要u进行Gamma校正。

## Gamma校正

Gamma校正的基本思路非常简单，我们利用 $y = x^{\gamma}$ 关于 $y = x$ 的对称函数 $y = x^{\frac{1}{\gamma}}$ ：

![Screenshot from 2021-11-22 10-38-32.png](https://i.loli.net/2021/11/22/WTNxKCDozXyFEQJ.png)

我们可以在着色阶段手动对颜色进行调整：

```glsl
// ...
fragColor.rgb = pow(fragColor.rgb, vec3(1 / gamma))
// ...
```

或者我们可以使用sRGB色彩空间，在该空间中，颜色本身就是按照 $\gamma = 2.2$ 进行过校正的。在OpenGL中，你可以使用：

```c
glEnable(GL_FRAMEBUFFER_SRGB);
```

## 参考资料
+ [learnopengl-Gamma校正](https://learnopengl-cn.github.io/05%20Advanced%20Lighting/02%20Gamma%20Correction/)
+ [色彩校正中的 gamma 值是什么？](https://www.zhihu.com/question/27467127/answer/37602200)
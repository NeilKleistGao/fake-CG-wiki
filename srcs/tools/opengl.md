## 什么是OpenGL
OpenGL（英语：Open Graphics Library，译名：开放图形库或者“开放式图形库”）是用于渲染2D、3D矢量图形的跨语言、跨平台的应用程序编程接口（API）。这个接口由近350个不同的函数调用组成，用来从简单的图形比特绘制复杂的三维景象。

我们可以使用OpenGL完成很多绘制功能，而无需关心如何操控你的操作系统甚至是硬件。

## 为什么使用OpenGL
可能你也听说过像DirectX或Vulkan这样的图形库。使用这些图形库和他们对应的着色器语言也是可以的。
不过如果你是新手，我们还是建议你使用OpenGL，因为他能跨平台使用，且拥有丰富的教程和资源。

你可以在网络上找到大量关于OpenGL环境配置的文章。我们不再针对特定的平台进行展开。

我们将会使用[GLEW](http://glew.sourceforge.net/)实现，并使用[GLFW](https://www.glfw.org/)进行窗口管理。你也可以使用其他的实现或窗口管理模块，甚至可以直接使用WebGL，它在跨平台上比OpenGL更进一步，不过你需要使用`JavaScript`对它进行控制。

参考资料中补充了一些学习OpenGL的资料。

## 参考资料
+ [OpenGL - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/OpenGL)
+ [Learn OpenGL CN](https://learnopengl-cn.github.io/)
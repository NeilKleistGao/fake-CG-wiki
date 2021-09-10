是的！我们完全有可能不使用任何的框架，而是自己从头开始造轮子！
这对于学习一些基础的功能是非常有帮助的。过于完善的框架会使我们产生依赖。

对于部分基础的功能，我们会考虑使用C++自己编写。
由于使用CPU进行运算，性能上可能会受到较大的影响。因此，所有不使用框架的代码都是离线的（离线的反义是“实时”，这意味着我们每秒需要生成至少30张图片）。

尽管大部分的功能由我们完成，你仍然可以考虑使用如下的第三方库帮助你进行图片和模型的读写：

+ [stb image](https://github.com/nothings/stb)
+ [SOIL2](https://github.com/SpartanJ/SOIL2)
+ [OBJ Loader](https://github.com/Bly7/OBJ-Loader)

以下是一些经典的入门“自搭轮子”教程，你可以跟着这些教程一步一步实现一个自己的软渲染器：

+ [tiny renderer](https://github.com/ssloy/tinyrenderer)
+ [Ray Tracing in One Weekend](https://raytracing.github.io/books/RayTracingInOneWeekend.html) 

更多的教程和依赖库，我们将会在具体的文章里讲述。
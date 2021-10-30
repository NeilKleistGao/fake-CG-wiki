## 着色与光照
在拥有了几何数据以后，我们需要决定每个像素要如何渲染。我们需要为像素指定颜色，甚至是纹理（这部分在后面会提到）。
这部分的功能通常是由片段着色器来完成（如果是可编程的渲染管线）。如下是一个Unity中的简单的着色器代码：

```s
Shader "Unlit/Test"
{
    Properties
    {
        _Color ("Color", Color) = (1, 1, 1, 1)
    }
    SubShader
    {
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
            };

            struct v2f
            {
                float4 vertex : SV_POSITION;
            };

            fixed4 _Color;

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                return _Color;
            }
            ENDCG
        }
    }
}

```

这个着色器使用单独的颜色来渲染我们的三维物体。`frag`函数描述了这个材质的片段着色器，你可以看到他将我们输入的颜色直接返回了，所以我们渲染出的物体的颜色是固定的。
每个需要被渲染的几何上的片段都会被执行一次片段着色器，这样的点被称为着色点。我们可以看到这个着色器的效果：

![Screenshot from 2021-10-30 12-55-11.png](https://i.loli.net/2021/10/30/OMG4RLHurn3qV6N.png)

这个着色的效果似乎和我们所想象的有一些出入：我们只能看到一个圆，而不是一个球。渲染引擎并没有出错，因为一个球无论从哪里看过去，都是一个不变的圆。
出现这种情况的原因是因为缺少光照，所以这个球无法给我们带来立体感。虽然球的各处颜色相同，但由于光照的位置和方向不同，我们所看到的明暗应该是不一样的，这就是立体感缺失的原因。

## 光照类型
根据光照算法的不同，我们可以给出不同的划分方式。常见的划分方式是按照光照计算的频率进行的。

光照计算的频率是可以调整的，如果硬件水平跟不上，我们可以降低光照计算的频率；反之，我们可以在次世代的机器上进行更高频率的计算，以实现更好的效果。
按照这个划分方式，我们可以得到三种不同的类型：

+ 逐表面的光照：对于模型的不同表面，各自计算一次光照，并应用在表面的所有像素上。计算时使用的法线向量为平面的法线向量。这种计算方式在模型表面不多的时候开销低，但只适合于低多边形风格的渲染
+ 逐顶点的光照：对于每一个表面，计算其所有顶点的光照；对于表面内其他像素，我们使用这些顶点光照的插值来生成。
+ 逐像素（片段）的光照：对于每一个需要渲染的像素/片段，我们都执行一次光照计算。这样的渲染在模型面数较低的时候喜爱过最好，但开销也是最大的

![Screenshot from 2021-10-30 13-10-15.png](https://i.loli.net/2021/10/30/byctAfeY9iDILRh.png)

除此之外，还有其他的划分方式。例如，根据光线的弹射次数，可以分为直接光照（光线直接打在着色点上）和间接光照（光线通过弹射打在着色点上）；根据光线的生成方式，可以分为实时光照（运行时进行光照计算）和预计算光照（在运行前进行一系列的预处理，也叫光照烘焙）。

## 基本光照模型
### Lambert模型
Lambert模型非常简单，他只包含环境光和方向光。

环境光可以被简单地当作一个常数，或者是一个与光照强度正相关的函数。虽然光照并不能直接照射到场景中的所有部分，但这并不意味着照不到的地方就是完全黑暗的。
由于间接光照的存在，他们只是看起来比光照下的物体更暗一些。由于光栅化成像并不模拟光线的传递，间接光照的计算非常复杂，几乎不可能完成。为此，我们对间接光照进行了简化：使用一个常数代替。

Lambert模型的方向光遵循Lambert定律：表面接受到来自光照的能量，与表面法线和光线方向的夹角成反比。夹角越大，表面接收到的能量就越小。当夹角大于等于90度时，表面将无法接受到任何的能量。

![Screenshot from 2021-10-30 13-37-13.png](https://i.loli.net/2021/10/30/GhuTd5zVYobfXce.png)

这和地球产生四季的原理是相似的：夏天时，太阳直射点移动到北回归线附近（这里以北半球为例，如果有南半球的读者请自行转换），此时北半球某点（以北京为例）的法线与光线的夹角变小，此时人们会感到炎热；而当冬天来到的时候，太阳直射点移动到南回归线附近，此时的夹角会增大许多，天气也就变得寒冷。对于热带地区，由于这个夹角常年很小，所以终年炎热。

Lambert模型的方向光使用如下的公式计算： $Color \times Light \times max(0, \vec L \cdot \vec N)$ 。需要注意的是光线向量和法线向量需要进行归一化，此时点成的结果才是夹角的余弦值。

我们可以使用Unity实现这个效果：
```s
// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'

Shader "Unlit/Test"
{
    Properties
    {
        _AmbientColor ("Ambient Color", Color) = (1, 1, 1, 1)
        _DiffuseColor ("Diffuse Color", Color) = (1, 1, 1, 1)
    }
    SubShader
    {
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };

            struct v2f
            {
                float4 vertex : SV_POSITION;
                float3 normal : TEXCOORD0;
            };

            fixed4 _AmbientColor;
            fixed4 _DiffuseColor;

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.normal = mul(v.normal, (float3x3)unity_WorldToObject);
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                fixed3 normal = normalize(i.normal);
                fixed3 light = normalize(_WorldSpaceLightPos0.xyz);
                fixed3 color = _AmbientColor.rgb + _DiffuseColor.rgb * saturate(dot(normal, light));
                return fixed4(color, 1.0);
            }
            ENDCG
        }
    }
}

```

最终效果如下：

![Screenshot from 2021-10-30 13-53-59.png](https://i.loli.net/2021/10/30/VyviQqg9e8WzdLR.png)

### 半Lambert模型
我们已经可以看到几何感了，但是在光不能到达的部分，颜色都是一样的黑暗。这不是我们希望的。虽然他们颜色要比正常颜色暗，但也应该有一个递进的过程。
为此，Valve提出了一个改进做法：将慢反射系数的余弦值从 $[-1, 1]$ 映射到 $[0, 1]$ 上，此时我们就不需要再进行求最大值的计算了：

```s
// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'

Shader "Unlit/Test"
{
    Properties
    {
        _AmbientColor ("Ambient Color", Color) = (1, 1, 1, 1)
        _DiffuseColor ("Diffuse Color", Color) = (1, 1, 1, 1)
    }
    SubShader
    {
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };

            struct v2f
            {
                float4 vertex : SV_POSITION;
                float3 normal : TEXCOORD0;
            };

            fixed4 _AmbientColor;
            fixed4 _DiffuseColor;

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.normal = mul(v.normal, (float3x3)unity_WorldToObject);
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                fixed3 normal = normalize(i.normal);
                fixed3 light = normalize(_WorldSpaceLightPos0.xyz);
                float cof = dot(normal, light) * 0.5 + 0.5;
                fixed3 color = _AmbientColor.rgb + _DiffuseColor.rgb * cof;
                return fixed4(color, 1.0);
            }
            ENDCG
        }
    }
}
```

半Lambert模型的光照要比Lambert模型亮不少，但是过渡更加自然：

![Screenshot from 2021-10-30 14-02-34.png](https://i.loli.net/2021/10/30/WElTrC7egazvI8B.png)

### Phong模型
Phong模型在Lambert模型的基础上添加了高光项，使得效果更加真实。为了处理高光，我们需要引入第三个向量：摄像机（或者是人眼）的 `lookat` 向量。
当高光被反射时，只有当其可以反射到摄像机（或者人眼中）时才能起效果（类似于镜面反射）。因此我们需要根据法向量和入射向量求出出射向量，并判断出射向量与
观察向量之间的夹角：

![Screenshot from 2021-10-30 14-14-28.png](https://i.loli.net/2021/10/30/oCWIJgB3rK6bQjn.png)

如图，我们可以得到反射相关向量的式子： $2(\vec I \cdot \vec N)\vec N = \vec O - \vec I$ 。所以反射光线的向量的表达式为 $\vec O = \vec I + 2(\vec I \cdot \vec N)\vec N$ 。如果你使用的是Unity Shader，你可以使用内嵌的`reflect`函数得到反射光的方向 

我们改写之前的着色器代码：
```s
// Upgrade NOTE: replaced '_World2Object' with 'unity_WorldToObject'

Shader "Unlit/Test"
{
    Properties
    {
        _AmbientColor ("Ambient Color", Color) = (1, 1, 1, 1)
        _DiffuseColor ("Diffuse Color", Color) = (1, 1, 1, 1)
        _SpecularColor ("Specular Color", Color) = (1, 1, 1, 1)
    }
    SubShader
    {
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };

            struct v2f
            {
                float4 vertex : SV_POSITION;
                float3 normal : TEXCOORD0;
            };

            fixed4 _AmbientColor;
            fixed4 _DiffuseColor;
            fixed4 _SpecularColor;

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.normal = mul(v.normal, (float3x3)unity_WorldToObject);
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                fixed3 normal = normalize(i.normal);
                fixed3 light = normalize(_WorldSpaceLightPos0.xyz);
                float cof = dot(normal, light) * 0.5 + 0.5;

                fixed3 ref = normalize(reflect(light, normal));
                fixed3 view = normalize(_WorldSpaceCameraPos.xyz - i.vertex.xzy);

                fixed3 color = _AmbientColor.rgb + _DiffuseColor.rgb * cof + _SpecularColor.rgb * max(0, dot(view, ref));
                return fixed4(color, 1.0);
            }
            ENDCG
        }
    }
}

```

为了方便演示，我调整了摄像机的方位：

![Screenshot from 2021-10-30 14-29-05.png](https://i.loli.net/2021/10/30/WFSrgwoEjbH83Ui.png)

我们会发现这个高光实在是太大了！因为余弦函数的变化过于缓慢，导致角度增加时高光消失的速度也过于缓慢。但现实生活中高光只有在非常小的角度内才能看到。
为此，我们不再使用 $cos\theta$ 作为高光的系数，而是改用 $cos^k\theta$ 。由于这个 $k$ 的存在，余弦函数将会很快降到0：

![Screenshot from 2021-10-30 14-33-13.png](https://i.loli.net/2021/10/30/fcxLRq2CMt5sbrd.png)

我们还可以通过控制 $k$ 的大小来调整材质的光滑程度， $k$ 越大，说明材质越粗糙，能看到高光的角度就越小：

```s
fixed4 frag (v2f i) : SV_Target
{
    fixed3 normal = normalize(i.normal);
    fixed3 light = normalize(_WorldSpaceLightPos0.xyz);
    float cof = dot(normal, light) * 0.5 + 0.5;

    fixed3 ref = normalize(reflect(light, normal));
    fixed3 view = normalize(_WorldSpaceCameraPos.xyz - i.vertex.xzy);

    fixed3 color = _AmbientColor.rgb + _DiffuseColor.rgb * cof + _SpecularColor.rgb * pow(max(0, dot(view, ref)), _Gloss);
    return fixed4(color, 1.0);
}
```

最终效果：

![Screenshot from 2021-10-30 14-36-13.png](https://i.loli.net/2021/10/30/otk1y4FznC7efJT.png)

### Blinn-Phong光照模型
Phong模型已经交出了一份令人满意的答卷，但我们依然感到不满足。并不是因为Phong模型的效果不够好，而是反射运算的运算量造成了不小的开销： $\vec O = \vec I + 2(\vec I \cdot \vec N)\vec N$ 。为了得到余弦值，我们需要进行两次点乘，两次数乘和一次向量加法。

Blinn-Phong光照模型对这个算法进行了优化：我们不再计算最终的反射响亮，而是通过将光线向量和视线向量相加得到半程向量，然后判断半程向量和法线向量之间的夹角：

![Screenshot from 2021-10-30 14-43-57.png](https://i.loli.net/2021/10/30/fqFys5EuQribMz9.png)

此时我们的开销仅包含一次加法和一次点乘，比原来的模型更加快速：

```s
fixed4 frag (v2f i) : SV_Target
{
    fixed3 normal = normalize(i.normal);
    fixed3 light = normalize(_WorldSpaceLightPos0.xyz);
    float cof = dot(normal, light) * 0.5 + 0.5;

    fixed3 h = normalize(normalize(i.vertex.xzy - _WorldSpaceCameraPos.xyz) - light.xzy);

    fixed3 color = _AmbientColor.rgb + _DiffuseColor.rgb * cof + _SpecularColor.rgb * pow(max(0, dot(h, normal)), _Gloss);
    return fixed4(color, 1.0);
}
```

## 参考资料
+ 《Unity Shader入门精要》 冯乐乐
+ [GAMES101-现代计算机图形学入门-闫令琪](https://www.bilibili.com/video/BV1X7411F744?p=8)
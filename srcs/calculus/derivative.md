## 函数极限
我们可以顺理成章地将数列的极限推广到函数的极限：数列就是特殊的函数， $\lim_{n \to \infty} a_n$ 其实可以被看作 $\lim_{n \to \infty} f(n)$ ，
其中 $f(n) = a_n$ 。

函数极限的定义和数列极限的定义类似： $\lim_{x \to x_0} f(x) = a$ ，当且仅当对于任意的 $\epsilon > 0$ ，都存在 $\delta > 0$ ，使得 $|x - x_0| < \delta, |f(x) - a| < \epsilon$ 。

你会发现两者的极限形式都是类似的：给定任意的“接近的需求”，我们都可以满足，这就被称之为极限。
不同之处在于数列的极限中 $n$ 只能取正整数，且只能朝着一个方向增长；函数极限中 $x$ 的定义域被扩充到了实数域，且不再仅限于“无限逼近无穷”，甚至这个值并不在 $f(x)$ 的定义域上。

以 $f(x) = 2x, (x \ne 2)$ 为例：

![Screenshot from 2021-10-10 09-30-56.png](https://i.loli.net/2021/10/10/4J7tSdPhU1Yu9mG.png)

我们希望求 $\lim_{x \to 2}f(x)$ 。直观地看，这个极限的值应该就是 $4$ ；另一方面， $x = 2$ 时函数并没有定义，这多少让人有些担心。好在我们的定义中写的是 $|f(x) - a| < \epsilon$ ，而不是 $|f(x) - f(x_0)| < \epsilon$ ，这使得我们依然可以完成这份证明。

我们只考虑 $x < 2$ 的情况（ $x > 2$ 时同理）， $|f(x) - 4| = 4 - 2x < \epsilon$ ，所以 $2x > 4 - \epsilon$ ， $x > 2 - \frac{\epsilon}{2}$ 。
于是，对于任意的 $\epsilon > 0$ ，我们都可以找到 $\delta = \frac{\epsilon}{2}$ ，使得 $|x - 2| < \delta$ 时， $|f(x) - 4| = 4 - 2x < \epsilon$ 成立。
证毕。
 
## 函数求导
在有了函数极限的概念后，我们便可以解决下面的这个问题：函数在 $x = x_0$ 时，发生了微小的改动，那么 $y$ 的值将会如何变化？
这样的问题在物理中非常常见：质点运动的速度 $v$ 是时间 $t$ 的函数（假设这是直线上的运动，这样我们就可以忽略速度是个向量这件事，从而简化问题），时间发生了微妙的变化，速度的变化要如何描述？

假设这是一个加速运动，一个“足够微小”的变化可以用极限表示： $\lim_{\Delta t \to 0}$ 。
我们可以使用二者之间的比值来表示二者之间的关系： $\lim_{\Delta t \to 0} \frac{f(t + \Delta t) - f(t)}{\Delta t}$ 。这个式子就是函数导数的定义，记作 $f'(x_0)$ 或者 $\frac{df}{dx}(x_0)$ 。

导数表示的是函数在一个点附近的变化程度。如果函数本身表示的是速度的话，那么其导数表示的就是某个时刻的加速度： $a(t) = \lim_{\Delta t \to 0} \frac{v(t + \Delta t) - v(t)}{\Delta t}$ 。在几何上，导函数的图像是原函数的切线的斜率。

## 常见的求导公式
### 四则运算
导函数之间也可以进行四则运算。

加减法是比较简单的情况：

$$
\frac{d}{dx}(f(x) \pm g(x)) = \lim_{\Delta x \to 0} \frac{(f(x + \Delta x) \pm g(x + \Delta x)) - (f(x) \pm g(x))}{\Delta x} =\\ \lim_{\Delta x \to 0} \frac{(f(x + \Delta x) - f(x)) \pm (g(x + \Delta x) - g(x))}{\Delta x} = f'(x) \pm g'(x)
$$

对于乘法，我们需要一点小技巧： $\frac{d}{dx}(f(x)g(x)) = \lim_{\Delta x \to 0} \frac{f(x + \Delta x)g(x + \Delta x) - f(x)g(x)}{\Delta x}$ 。
此时分子上我们无从下手，我们需要在其中进行添项：
$\frac{d}{dx}(f(x)g(x)) = \lim_{\Delta x \to 0} \frac{f(x + \Delta x)g(x + \Delta x) - f(x + \Delta x)g(x) + f(x + \Delta x)g(x) - f(x)g(x)}{\Delta x}$ 。
现在我们就可以将其视为两部分： $f(x + \Delta x)g'(x + \Delta x)$ 和 $f'(x)g(x)$ 。由于 $\lim_{\Delta x \to 0}$ ，我们最后可以得到 $\frac{d}{dx}(f(x)g(x)) = f'g + g'f$ 。

最为棘手的是除法的情况： $\frac{d}{dx}\frac{f(x)}{g(x)}, g(x) \ne 0$ 。 $\lim_{\Delta x \to 0} \frac{\frac{f(x + \Delta x)}{g(x + \Delta x)} - \frac{f(x)}{g(x)}}{\Delta x}$ 。对这个式子进行通分得到 $\lim_{\Delta x \to 0} \frac{1}{g(x + \Delta x)g(x)} \frac{f(x + \Delta x)g(x) - g(x + \Delta x)f(x)}{\Delta x}$ 。
对后面的部分，我们使用和乘法类似的操作： $\lim_{\Delta x \to 0} \frac{f(x + \Delta x)g(x) - f(x)g(x) + f(x)g(x) - g(x + \Delta x)f(x)}{\Delta x}$ ，最终我们可以得到： $\frac{d}{dx}\frac{f(x)}{g(x)} = \frac{f'g - g'f}{g^2}$ 。

### 链式法则
除了普通的四则运算，我们还经常会遇到两个函数的复合： $f(g(x))$ 。如果这个函数的导数存在，它应该是什么样的？

因为 $g'(x) = \lim_{\Delta x \to 0} \frac{g(x + \Delta x) - g(x)}{\Delta x}$ ，我们不妨把极限定义中的 $\epsilon$ 暂时借过来： $g'(x) = \frac{g(x + \Delta x) - g(x)}{\Delta x} - \epsilon$ （即我们现在不再求极限，而是保留 $\epsilon$ 作为一个“误差”）。
同理我们可以得到： $f'(g(x)) = \frac{f(g(x) + \Delta g(x)) - f(g(x))}{\Delta g(x)} - \epsilon_2 = \frac{f(g(x + \Delta x)) - f(g(x))}{\Delta g(x)}$ 。

所以 $\frac{f(g(x + \Delta x)) - f(g(x))}{\Delta x} = \frac{f'(g(x))\Delta g(x)}{\Delta x} = f'(g(x))g'(x)$ 。这就是函数求导的链式法则。
 
### 常数函数
常数函数表示该函数的值恒为某个数，比如： $f(x) = 42$ 。从加速度的例子，你很容易可以理解：这样的函数的导数是恒为 $0$ 的，因为他们从来不会发生变化！
我们也可以利用刚才的公式验证这一点： $f'(x) = \lim_{\Delta x \to 0} \frac{f(x + \Delta x) - f(x)}{\Delta x} = \frac{0}{\Delta x} = 0$ 。

### 指数函数
指数函数是形如 $f(x) = a^x$ 的函数。我们使用定义来求指数函数的导数： $\lim_{\Delta x \to 0} \frac{a^{x + \Delta x} - a^x}{\Delta x} = \lim_{\Delta x \to 0} \frac{a^x(a^{\Delta x} - 1)}{\Delta x}$ 。这个式子我们并不好处理：x在指数的位置，很难与分母发生反映。我们可以通过取自然对数的方式将指数拿下来，运算完成后再使用 $e^x$ 将式子还原回去。

对极限内的式子取对数，得到： $\lim_{\Delta x \to 0} xlna + ln(a^{\Delta x} - 1) - ln\Delta x$ 。
还原后我们就可以得到： $f'(x) = a^x \lim_{\Delta x \to 0} \frac{a^{\Delta x} - 1}{\Delta x}$ 。
我们不妨另 $t = a^{\Delta x} - 1$ ，此时极限内变为 $\lim_{\Delta t \to 0} \frac{t}{log_a^{t + 1}} = \lim_{\Delta t \to 0} \frac{t}{log_a^{t + 1}}$ 。
我们把 $t$ 从分子上拿下来，得到 $\lim_{\Delta t \to 0} \frac{1}{log_a^{(t + 1)^{\frac{1}{t}}}}$ 。

我们现在来着手解决 $\lim_{\Delta t \to 0} (t + 1)^{\frac{1}{t}}$ 。这是一个非常经典的极限形式。 $0$ 的无穷次方的形式一定会让你感到很迷惑：这个极限真的存在吗？我们再一次借助自然对数： $\lim_{\Delta t \to 0} \frac{ln(t + 1)}{t}$ 。我们可以将这个式子看作 $\lim_{\Delta t \to 0} \frac{ln(t + 1) - ln(1)}{t}$ ，这样我们就将极限再一次转换成了求导。我们需要知道 $ln(x)$ 在 $x = 1$ 处切线的斜率，而该点的切线恰好为 $y = x - 1$ （如果你熟悉的话， $y = x$ 恰好也是 $ln(x + 1)$ 的切线）：

![Screenshot from 2021-10-10 12-35-07.png](https://i.loli.net/2021/10/10/5c2CwAoINahElud.png)
 
将式子回带，我们得到 $\lim_{\Delta t \to 0} (t + 1)^{\frac{1}{t}} = e$ 。所以我们的导函数变为了 $f'(x) = a^x \frac{log_a^a}{log_a^e} = a^xlna$ 。我们最后使用了换底公式，得到导函数的形式。特殊的， $e^x$ 的导函数依然是它本身。
 
### 对数函数
指数函数是形如 $f(x) = log_a^x$ 的函数。对数函数的求导和指数函数类似： $\lim_{\Delta x \to 0} \frac{log_a^{x + \Delta x} - log_a^x}{\Delta x}$ 。
我们将分母单独拿出来，得到 $\lim_{\Delta x \to 0} \frac{1}{\Delta x} log_a\frac{x + \Delta x}{x} = \lim_{\Delta x \to 0} \frac{1}{x} \frac{x}{\Delta x} log_a(1 + \frac{\Delta x}{x})$ 。使用和指数函数中相同的伎俩，我们最终可以轻松得到结果： $f'(x) = \frac{1}{xlna}$ 。类似的，当 $f(x) = lnx$ 时，我们有特殊形式 $f'(x) = \frac{1}{x}$ 。


### 幂函数
幂函数是形如 $f(x) = x^m$ 的函数。我们使用定义进行计算： $\lim_{\Delta x \to 0} \frac{(x + \Delta x)^m - x^m}{\Delta x}$ 。

这个式子非常难处理！除非你使用广义的二项式定理（即 $(a + b)^n$ 的推广形式，此时的 $n$ 对全体实数均成立）将分子展开。
我们再次使用惯用伎俩重写函数： $f(x) = e^{mlnx}$ 。根据上面已知的求导公式，以及链式法则，我们得到： $f'(x) = e^{mlnx}\frac{m}{x} = mx^{m - 1}$ 。

这个式子虽然简单，但需要大量依赖上面的结论，故将它放在后面讨论。

### 三角函数
我们将只讨论 $sinx$ 和 $cosx$ 的导数，其他三角函数的结论都可以通过这两个函数的四则运算得到。

根据公式，我们有： $sin'(x) = \lim_{\Delta x \to 0} \frac{sin(x + \Delta x) - sinx}{\Delta x}$ 和 $cos'(x) = \lim_{\Delta x \to 0} \frac{cos(x + \Delta x) - cosx}{\Delta x}$ 。
利用三角形和差公式，我们可以将式子展开： $sin'(x) = \lim_{\Delta x \to 0} \frac{sinxcos\Delta x + cosxsin\Delta x - sinx}{\Delta x}$ ， $cos'(x) = \lim_{\Delta x \to 0} \frac{cosxcos\Delta x - sinxsin\Delta x - cosx}{\Delta x}$ 。因为 $cos 0 = 1$ ，我们将 $cos\Delta x$ 项消去， 得到： $sin'(x) = \lim_{\Delta x \to 0} \frac{cosxsin\Delta x}{\Delta x}$ ， $cos'(x) = \lim_{\Delta x \to 0} \frac{-sinxsin\Delta x}{\Delta x}$ 。

此时我们需要用到另一个经典的极限： $\lim_{t \to 0} \frac{sint}{t}$ 。

![Screenshot from 2021-10-10 19-18-21.png](https://i.loli.net/2021/10/10/fMaEheANq3LvwJg.png)

如图，我们可以非常直观地得到一个有关面积的不等式： 扇形AFG < 三角形ABC < 扇形ABC，
所以有 $\frac{1}{2}tcos^2t < \frac{1}{2}sint < \frac{1}{2}t$ 。不等式乘上 $\frac{2}{t}$ ,得到 $cos^2t < \frac{sint}{t} < 1$ 。
由于 $\lim_{t \to 0}$ ， $cos^2t = 1$ 。由夹逼定理（这是一个非常简单而且显然的定理，我们不打算花更多的篇幅去描述他）， $\lim_{t \to 0} \frac{sint}{t} = 1$ 。

带入之前三角函数的式子，我们可以得到： $sin'x = cosx, cos'x = -sinx$ 。
 
## 微分
微分是一个和导数关系很近的概念，但仅局限于一元函数。在一元函数中，可微和可导是等价的。
在一元函数中，当 $f(x)$ 在 $x_0$ 处发生变动，变为 $x_0 + \Delta x$ 时， 函数的增量 $\Delta y = f(x_0 + \Delta x) - f(x_0)$ 。
如果函数增量可以被表示为 $\Delta y = A\Delta x + o(\Delta x)$ （其中 $o(\Delta x)$ 是 $\Delta x$ 的高阶无穷小，这意味着当极限情况下，他比 $\Delta x$ 更快接近于0），
我们称 $A\Delta x$ 为 $f(x)$ 的微分。事实上，对于医院函数来说， $A = f'(x)$ 。

微分和导数类似，拥有着相同的四则运算法则和链式法则。

## 参考资料
+ [导数- 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E5%AF%BC%E6%95%B0)
+ [链式法则- 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E9%93%BE%E5%BC%8F%E6%B3%95%E5%88%99)
+ [微分- 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E5%BE%AE%E5%88%86)
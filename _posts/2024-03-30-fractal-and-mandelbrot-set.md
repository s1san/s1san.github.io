# 分形与曼德博集

分形（fractal），通常被定义为“一个粗糙或零碎的几何形状，可以分成数个部分，且每一部分都（至少近似地）是整体缩小后的形状”，即具有自相似的性质。 分形在数学中是一种抽象的物体，用于描述自然界中存在的事物。 分形也被称为扩展对称或展开对称。如果在每次放大后，形状的重复是完全相同的，这被称为自相似。

分形的性质

* 自相似性：分形的一部分在足够的放大下可以看到整体的特征
* 尺度不变性：分形在不同的尺度下都具有相似的结构。
* 分形维度：分形通常具有非整数的分数维度。
* 复杂性：分形呈现出复杂的结构和形态，尽管它们通常由简单的规则生成。
* 混沌性：一些分形系统表现出混沌行为，即对初始条件极其敏感，微小的变化可能导致完全不同的结果。

### 曼德博集

曼德博集(Mandelbrot set)是一种在**复平面**上组成分形的点的集合，以数学家本华·曼德博的名字命名。

**定义**
曼德博集合可以用复二次多项式来定义：
$$\begin{aligned}
&f_z(z)=z^2+c
\end{aligned}$$

其中$c$是一个复数参数。
从$z=0$开始对$f_c(z)$进行迭代：

$$\begin{aligned}
&z_{n+1}=z_n^2+c,n=0,1,2,\ldots  \\
&z_{0}=0 \\
&z_{1}=z_{0}^{2}+c=c \\
&z_{2}=z_{1}^{2}+c=c^{2}+c
\end{aligned}$$

不同的参数$c$可能使序列的绝对值逐渐发散到无限大，也可能收敛在有限的区域内。

曼德博集合$M$就是**使序列不延伸至无限大**的所有复数的集合。

### 曼德博集分形实现

本文使用[SDL](https://github.com/libsdl-org/SDL)实现。

在主函数中，主要有两部分。

```c++
    for (double i = 0.0; i < 1.0; i += 0.001) // 步长越小，图像越精细
    {
        for (double j = 0.0; j < 1.0; j += 0.001)
        {
            // 在每个位置上，通过 std::lerp 函数将屏幕上的像素位置映射到复平面上的实部。
            // 范围在 [-2.0, 2.0] 中能更好的观察集合。
            double x = std::lerp(-2.0, 2.0, i);
            double y = std::lerp(-2.0, 2.0, j);

            int res = in_set(std::complex< double >(x, y));
            if (res == false) // 在集合内，将颜色设置为黑色
            {
                SDL_SetRenderDrawColor(renderer, 0, 0, 0, 255);
                SDL_RenderDrawPointF(renderer, i * width / 2, j * height / 2);
            }
            else
            {
                SDL_SetRenderDrawColor(renderer, 10 * res % 255, 10 * res % 255, 10 * res % 255, 255);
                SDL_RenderDrawPointF(renderer, i * width / 2, j * height / 2);
            }
        }
    }
```

在`in_set`函数中，按照曼德博集定义进行迭代，判断传入的参数是否在集合内。

```c++
int in_set(std::complex< double > c)
{    
	std::complex< double > z(0, 0);
    for (int i = 0; i < 100; i++)
    {
        z = std::pow(z, 2) + c;
        if (std::norm(z) > 10) // 如果大于10，则认为该点不属于Mandelbrot集合，返回当前迭代次数 i。
        {
            return i;
        }
    }
    return 0;
}
```

本文章代码在[这里](https://github.com/s1san/sbr/tree/main/Fractal)

**references**

* YouTube, The Builder, [Mandelbrot Set In C++ with SDL2 Tutorial](Mandelbrot Set In C++ with SDL2 Tutorial)
* YouTube, Pezzza's Work, [Simple Fractal rendering](https://www.youtube.com/watch?v=uc2yok_pLV4&list=WL&index=2&t=27s)
* Wikipedia, [Mandelbrot set](https://en.wikipedia.org/wiki/Mandelbrot_set)

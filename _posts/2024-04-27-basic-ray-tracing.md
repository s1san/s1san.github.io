# 基础光线追踪

不同于[光栅化](https://zh.wikipedia.org/wiki/%E6%A0%85%E6%A0%BC%E5%8C%96)，光线追踪基于物理光学原理，模拟光线在场景中的传播，生成高质量图像。传统的光栅化难以实现真实世界中的阴影、全局效果，而在光线追踪中，从相机或观察者位置发射光线，并跟踪这些光线在场景中的路径，直到它们与物体相交或到达场景的边界为止。通过考虑光线与场景中物体之间的相交情况，光线追踪可以模拟出真实世界中的光照、反射、折射等光学效果。

本文将使用C++且不依赖第三方库，实现一个基本的

光线追踪。你也可以将本文看做[tinyraytracer](https://github.com/ssloy/tinyraytracer)的中文翻译版，以下是最终效果：

![refraction](https://cdn.jsdelivr.net/gh/s1san/asusual@img/img/refraction.png)

光线追踪通常包括以下步骤：

1. 发射光线：从相机或观察者位置发射光线，通常是沿着每个像素的视线方向。
2. 光线求交：跟踪光线在场景中的路径，确定光线是否与场景中的物体相交。
3. 计算光线与物体的交点：如果光线与物体相交，则计算光线与物体的交点，这个交点可能是光线与物体表面的交点。
4. 光照计算：对于交点处的光线，根据场景中的光源以及物体的表面属性（如材质、颜色等），计算光线对交点的影响，包括漫反射、镜面反射、折射等效果。
5. 递归：对于反射、折射等情况，可以递归地发射新的光线，并在场景中继续跟踪，以模拟光线的多次反射和折射。

我们将一步步完成所有步骤，在实现光线追踪之前，我们需要做一些准备工作，搭建一个场景。

* 向量操作
* 图像合成
* 创建物体
* 相机配置

### 向量操作

因为不依赖与第三方库，我们需要自己写一个简单的向量库以方便后续操作。这里直接使用Github上的[tinyraytracer](https://github.com/ssloy/tinyraytracer/wiki)项目的中`geometry.h`。简单讲解一下代码，主要内容是C++中模板的使用：

```c++
template <size_t DIM, typename T> struct vec {
    vec() { for (size_t i = DIM; i--; data_[i] = T()); }
    T& operator[](const size_t i) { assert(i < DIM); return data_[i]; }
    const T& operator[](const size_t i) const { assert(i < DIM); return data_[i]; }
private:
    T data_[DIM];
};
```

定义一个模板结构体 `vec`，它有两个模板参数，`DIM` 表示向量的维度，`T` 表示向量元素的数据类型。

```c++
typedef vec<2, float> Vec2f;
typedef vec<3, float> Vec3f;
typedef vec<3, int  > Vec3i;
typedef vec<4, float> Vec4f;
```

定义了维度为2、3、3和4的向量的类型别名。

```c++
template <typename T> struct vec<3, T> {
    vec() : x(T()), y(T()), z(T()) {}
    vec(T X, T Y, T Z) : x(X), y(Y), z(Z) {}
    T& operator[](const size_t i) { assert(i < 3); return i <= 0 ? x : (1 == i ? y : z); }
    const T& operator[](const size_t i) const { assert(i < 3); return i <= 0 ? x : (1 == i ? y : z); }
    float norm() { return std::sqrt(x * x + y * y + z * z); }
    vec<3, T>& normalize(T l = 1) { *this = (*this) * (l / norm()); return *this; }
    T x, y, z;
};
```

对于三维向量 `vec<3, T>`进行了偏特化，添加了额外的成员`z`，并提供了计算向量长度和标准化的函数，以及实现叉乘运算的函数 `cross`。

```c++
template <typename T> vec<3, T> cross(vec<3, T> v1, vec<3, T> v2) {
    return vec<3, T>(v1.y * v2.z - v1.z * v2.y, v1.z * v2.x - v1.x * v2.z, v1.x * v2.y - v1.y * v2.x);
}
```

向量叉乘函数，用于计算两个三维向量的叉积，返回一个新的向量。

剩下的就是重载。

运算符重载

- `operator*` 重载乘法运算符，使得可以对两个向量进行点乘操作。
- `operator+` 和 `operator-` 重载了加法和减法运算符，可以对两个向量进行逐元素的加法和减法操作。
- `operator-` 重载取负号运算符，可以对向量进行取反操作。
- `operator<<` 重载输出流运算符，使得可以方便地输出向量的元素值。

输出流重载

- `operator<<` 重载输出流运算符，使得可以方便地输出向量的元素值。

### 图像合成

我们要将程序最后输出的结果保存为一张图像。

```c++
#include "geometry.h"
#include <chrono>
#include <fstream>
#include <iostream>
#include <vector>

void render() {
    const int width = 1024;
    const int height = 768;
    std::vector<Vec3f> framebuffer(width * height);

    for (size_t j = 0; j < height; j++) {
        for (size_t i = 0; i < width; i++) {
            // 通过计算像素的 UV 坐标（归一化坐标）来生成渲染结果。UV 坐标的范围是 [0, 1]，表示屏幕上每个像素的位置。
            framebuffer[i + j * width] = Vec3f(j / float(height), i / float(width), 0);
        }
    }

    std::ofstream ofs;
    ofs.open("out.ppm",  std::ofstream::binary);
    ofs << "P6\n" << width << " " << height << "\n255\n";
    for (size_t i = 0; i < height * width; ++i) {
        for (size_t j = 0; j < 3; j++) {
            ofs << (char)(255 * std::max(0.f, std::min(1.f, framebuffer[i][j])));
        }
    }
    ofs.close();
}

int main() {
    auto start = std::chrono::high_resolution_clock::now();
    
    render();
    
    auto end = std::chrono::high_resolution_clock::now();
    std::chrono::duration< double > diff = end - start;
    std::cout << "take: " << diff.count() << " s\n";
    return 0;
}
```

 PPM格式相对简单，易于理解和实现。 P6格式是PPM的**二进制**子格式之一。与P3格式（文本PPM）相比，P6格式使用二进制数据来存储图像信息，这在一定程度上可以减少文件大小。

P6格式中的每个像素用RGB三原色来表示，每个颜色通道占据一个字节，因此每个像素需要3个字节来表示颜色信息。

通过代码不难看出，P6格式：

* 文件头：包含格式标识符、图像宽度、图像高度和最大像素值等信息
* 按照RGB顺序排列的像素数据

在主函数中，还添加了记录程序运行时间的代码。

你可以用Adobe Photoshop，[GIMP](https://www.gimp.org/)等软件打开ppm文件。若代码运行成功，你将看到如下图像：

![out](https://cdn.jsdelivr.net/gh/s1san/asusual@img/img/out.jpg)

[这里](https://github.com/s1san/sbr/tree/main/BasicRayTracing/tutorial/step1)找到生成图像的代码。

### 创建物体

接下来，创建一个`Scene.h`的头文件，我们尝试在场景中添加一个球体，只需要一个三维向量和一个描述半径的标量即可表示一个球体。

此外，在球体结构体中，还需要一个与光线求交的函数它可以检查给定的射线（从`orig`出发沿`dir`方向的射线）是否与球体相交。射线与球体的相交计算不难，可以看成线与圆的相交，建议在草稿纸上推导出`ray_intersect`函数中的变量。

```c++
struct Sphere
{
    Vec3f    center;
    float    radius;

    Sphere( const Vec3f& c, const float r ) : center( c ), radius( r ) {}

    bool ray_intersect( const Vec3f& orig, const Vec3f& dir, float& t0 ) const
    {
        Vec3f L   = center - orig;	// 球心与光线起点的向量
        float tca = L * dir;	// 投影在光线方向上的距离，可以理解为将球心到光线起点的向量投影在光线方向上，得到投影长度，表示光线与球心的最近距离
        float d2  = L * L - tca * tca;	// 光线与球心的距离平方
        if ( d2 > radius * radius )	// 不相交
            return false;
        float thc = sqrtf( radius * radius - d2 );	// 光线与球面的交点到光线起点的距离，即球半径与投影长度的差值
        t0        = tca - thc;	// to，t1为光线与球体的两个交点距离
        float t1  = tca + thc;
        if ( t0 < 0 )	// 两距离取较小值作为实际的交点距离
            t0 = t1;
        if ( t0 < 0 )	// 若最终得到的交点距离仍然小于0，则说明光线的起点在球体内部
            return false;
        return true;
    }
};
```

现在，对于每个像素，我们将形成一条来自原点并穿过像素的射线，然后检查该射线是否与球体相交：

![camera](https://cdn.jsdelivr.net/gh/s1san/asusual@img/img/camera.svg)

在`Scene.cpp`中实现`cast_ray`函数，如果与球体没有交集，则填充背景色，否则填充球体颜色

```c++
Vec3f cast_ray( const Vec3f& orig, const Vec3f& dir, const Sphere& sphere )
{
    float sphere_dist = std::numeric_limits< float >::max();
    if ( !sphere.ray_intersect( orig, dir, sphere_dist ) )
    {
        return Vec3f( 0.2, 0.2, 0.2 );	// 背景色
    }

    return Vec3f( 0.8, 0.8, 0.8 );
}
```

### 相机配置

相机的参数有

* 图片宽度
* 图片高度
* 视场角（Field of View）
* 相机位置
* 视图方向，沿 z 轴，负无穷大方向

```c++
void render( const Sphere& sphere )
{
    const int            width  = 1024;
    const int            height = 768;
    const int            fov    = M_PI / 2.;
    std::vector< Vec3f > framebuffer( width * height );

#pragma omp parallel for
    for ( size_t j = 0; j < height; j++ )
    {
        for ( size_t i = 0; i < width; i++ )
        {
            float x = ( 2 * ( i + 0.5 ) / ( float )width - 1 ) * tan( fov / 2. ) * width / ( float )height;
            float y = -( 2 * ( j + 0.5 ) / ( float )height - 1 ) * tan( fov / 2. );
            Vec3f dir = Vec3f( x, y, -1 ).normalize();
            framebuffer[ i + j * width ] = cast_ray( Vec3f( 0, 0, 0 ), dir, sphere );
        }
    }

	[...]
}

float x = ( 2 * ( i + 0.5 ) / ( float )width - 1 ) * tan( fov / 2. ) * width / ( float )height;
float y = -( 2 * ( j + 0.5 ) / ( float )height - 1 ) * tan( fov / 2. );
Vec3f dir = Vec3f( x, y, -1 ).normalize();
framebuffer[ i + j * width ] = cast_ray( Vec3f( 0, 0, 0 ), dir, sphere );
```

修改`render`函数，我们要将场景中的物体渲染出来。使用**透视投影**的方式、根据**画布的宽高比**将像素映射到视平面上。

根据每个像素在视平面上的位置，构造射线的方向`dir`。这里射线的起点固定为 `(0, 0, 0)`，表示相机位置，而射线的方向则是从相机位置指向当前像素的位置。场景投影在位于平面 z = -1 的屏幕上。视场表示指定屏幕上可见的空间部分。

![fov](https://cdn.jsdelivr.net/gh/s1san/asusual@img/img/fov.png)

如图，假设我们想要通过屏幕第 12 个像素的中心投射一个向量，即计算蓝色向量。如何做到？

从屏幕左侧到蓝色尖端的距离为12+0.5。屏幕的16个像素对应于$2 * tan(fov/2)$世界单位。因此，向量的尖端位于距离屏幕左边缘$(12+0.5)/16 * 2*tan(fov/2)$ 个世界单位处。最后在计算中添加屏幕的纵横比，就能准确的找到光线方向的公式。

修改完`render`函数，在场景中添加一个球体后，你将会得到这样一张图像：

![sphere](https://cdn.jsdelivr.net/gh/s1san/asusual@img/img/sphere.png)

[这里](https://github.com/s1san/sbr/tree/main/BasicRayTracing/tutorial/step2)找到生成图像的代码。

### 添加光照

但是上面的图像哪是个球，不是一圆吗。因为这个场景还没有光，只需要添加光照，整个球体就会立体起来。然而，完全模拟自然世界中的光照是非常难的。图形学中有一句话：看起来是对的，那就是对的。在实时渲染中，为了更流畅的体验，广大图形学研究人员各显神通，用尽各种奇技淫巧目的就是想要骗过你的眼睛，即使是非物理的公式，却能以假乱真。

那么，如何让球体看起来像个球呢？想象一下，太阳光线射向地球，太阳从地平线升起越高，地表就越亮。相反，距离地平线越低，光线就越暗。当太阳从地平线落下后，光子根本就无法到达我们这里。回到球体，我们从相机发出光线，它停在一个球体上。我们如何知道交点照明的强度？计算**该点的法线向量与光线方向之间的角度**就可以了。角度越小，表面照明越好。

添加光源：

```c++
struct Light {
    Light(const Vec3f& p, const float &i): position(p),intensity(i) {}
    Vec3f position;
    float intensity;
};
```

添加材质：

```c++
struct Material {
    Material(const Vec3f &color):diffuse_color(color) {}
    Material():diffuse_color() {}

    Vec3f diffuse_color;
};
```

同时在`Sphere`结构体中添加材质变量：

```c++
struct Sphere
{
	[...]

    Material material;

    Sphere(const Vec3f& c, const float& r, const Material& m) : center(c), radius(r), material(m) {}

	[...]
};
```

在`scene_intersect`函数中处理光线与场景中物体是否相交。同时，把`render`和`cast_ray`函数中的球体变量修改为数组，以便在场景中添加多个球体。

```c++
bool scene_intersect(const Vec3f& orig, const Vec3f& dir, const std::vector<Sphere>& spheres, Vec3f& hit, Vec3f& N, Material& material)
{
    // 初始化为一个很大的值，记录光线与场景中各个物体的最近交点距离
    float spheres_dist = std::numeric_limits<float>::max();
    
    for (size_t i = 0; i < spheres.size(); i++) {
        float dist_i;
        if (spheres[i].ray_intersect(orig, dir, dist_i) && dist_i < spheres_dist) {
            spheres_dist = dist_i;
            hit = orig + dir * dist_i;
            N = (hit - spheres[i].center).normalize();
            material = spheres[i].material;
        }
    }
    return spheres_dist < 1000;
}
```

在cast_ray函数中，我们将返回计算后光源的颜色，而不是恒定的颜色。

```C++
Vec3f cast_ray(const Vec3f& orig, const Vec3f& dir, const std::vector<Sphere>& spheres, const std::vector<Light>& lights)
{
	[...]

    float diffuse_light_intensity = 0;
    for (size_t i = 0; i < lights.size(); i++)
    {
        Vec3f light_dir = (lights[i].position - point).normalize();
        diffuse_light_intensity += lights[i].intensity * std::max(0.f, light_dir * N);
    }

    return material.diffuse_color * diffuse_light_intensity;;
}
```

通过迭代场景中的每个光源，计算光线与该光源的夹角和光源的强度，进而计算出漫反射光照的强度。最后将漫反射光照强度乘以材质的漫反射颜色，作为最终的颜色结果返回。

![light](https://cdn.jsdelivr.net/gh/s1san/asusual@img/img/light.png)

[这里](https://github.com/s1san/sbr/tree/main/BasicRayTracing/tutorial/step3)找到生成图像的代码。接下来的步骤，都是为了让结果更接近物理世界。

### 镜面光照

为了让物体更有光泽，我们引入描述光照的经典模型：Phong模型。

Phong模型是一个局部光照模型，它包含三部分：

* 环境反射：物体表面受到周围环境光的间接照射所产生的光照效果。
* 漫反射：光线照射到粗糙表面后，由于表面微观结构的散射，使得光线在各个方向上均匀地反射。
* 镜面反射：光线照射到光滑表面后，由于表面的镜面反射特性，使得光线在特定方向上产生较强的反射现象。

![phong](https://cdn.jsdelivr.net/gh/s1san/asusual@img/img/phong.png)
$$
\begin{align*}
& I = I_d + I_s + I_{\text{ambient}} \\
& I_d = k_d \cdot I \cdot (\mathbf{L} \cdot \mathbf{N}) \\
& I_s = k_s \cdot I \cdot (\mathbf{R} \cdot \mathbf{V})^n
\end{align*}
$$
在漫反射公式中，

- $I_d$是漫反射光照强度
- $k_d$是漫反射系数
- $I$是光源强度
- $L$是入射光线方向向量
- $N$是表面法线向量

在镜面反射公式中，

* $I_s$是镜面反射光照强度，
* $k_s$是镜面反射系数，
* $I$是光源强度，
* $R$ 是反射光线方向向量，
* $V$ 是视线方向向量，
* $n$是镜面指数。

材质中增加漫反射系数，镜面反射指数，其中，漫反射系数是一个二维向量，表示材质的漫反射颜色和反射率

```C++
struct Material
{
    Material(const Vec2f& a, const Vec3f& color, const float& spec) : albedo(a), diffuse_color(color), specular_exponent(spec) {}
    Material() : albedo(1, 0), diffuse_color(), specular_exponent() {}

    Vec2f albedo;
    Vec3f diffuse_color;
    float specular_exponent;
};
```

添加`reflect`函数，计算光线的反射方向：

```c++
Vec3f reflect(const Vec3f& I, const Vec3f& N)
{
    return I - N * 2.f * (I * N);
}
```

修改`cast_ray`函数：

```c++
Vec3f cast_ray(const Vec3f& orig, const Vec3f& dir, const std::vector< Sphere >& spheres, const std::vector< Light >& lights)
{
	[...]

    float diffuse_light_intensity = 0, specular_light_intensity = 0;
    for (size_t i = 0; i < lights.size(); i++)
    {
        Vec3f light_dir = (lights[i].position - point).normalize();

        diffuse_light_intensity += lights[i].intensity * std::max(0.f, light_dir * N);
        specular_light_intensity += powf(std::max(0.f, -reflect(-light_dir, N) * dir), material.specular_exponent) * lights[i].intensity;
    }

    return material.diffuse_color * diffuse_light_intensity * material.albedo[0] + Vec3f(1., 1., 1.) * specular_light_intensity * material.albedo[1];
}
```

点积来计算光线方向向量 `light_dir` 和表面法线向量 `N` 的夹角余弦值，然后将其与光源的强度相乘，得到漫反射的强度。用反射函数计算出入射光线在表面上的反射方向向量，然后将其与视线方向向量求点积并取幂，再乘以光源的强度，得到镜面反射的强度。

将漫反射光照强度和镜面反射光照强度分别乘以材质的漫反射系数和镜面反射系数，然后加权求和，作为最终的颜色结果返回。

这样，图像就显得更有光泽了：

![specular](https://cdn.jsdelivr.net/gh/s1san/asusual@img/img/specular.png)

[这里](https://github.com/s1san/sbr/tree/main/BasicRayTracing/tutorial/step4)找到生成图像的代码。

### 添加阴影

如果大晴天走在街上看到一个人没有影子是非常吓人的，我们的光追也要有阴影。只需六行便可做到。

```c++
for (size_t i = 0; i < lights.size(); i++)
{
    Vec3f light_dir = ( lights[ i ].position - point ).normalize();

    float light_distance = ( lights[ i ].position - point ).norm();

    Vec3f    shadow_orig = light_dir * N < 0 ? point - N * 1e-3 : point + N * 1e-3;
    Vec3f    shadow_pt, shadow_N;
    Material tmpmaterial;
    if ( scene_intersect( shadow_orig, light_dir, spheres, shadow_pt, shadow_N, tmpmaterial ) && ( shadow_pt - shadow_orig ).norm() < light_distance )
        continue;

    [...]
}
```

计算从光源位置到交点的距离 `light_distance`。在计算漫反射和镜面反射光照强度之前，先检测光线是否被阻挡，被阻挡则存在阴影。为了检测阴影，我们从交点沿着光线方向发射一条射线，看它是否与场景中的其他物体相交。为了避免射线与自身相交，我们在光线方向上添加一个微小的偏移量，并发射一条新的射线 `shadow_orig`。如果这条射线与场景中的其他物体相交，并且交点到光源的距离小于从交点到光源的距离 `light_distance`，则说明光线被阻挡，该点处将产生阴影。

简单的说，做两条射线：相机到交点的射线和光线到交点的射线。如果这两个射线的路径上都存在其他物体（即相机和光源都无法直接看到交点），那么交点处就会被阻挡，认为产生阴影。

效果如下：

![shadow](https://cdn.jsdelivr.net/gh/s1san/asusual@img/img/shadow.png)

[这里](https://github.com/s1san/sbr/tree/main/BasicRayTracing/tutorial/step5)找到生成图像的代码。

### 反射与折射

当与球体相交时，我们只需计算反射光线（重用镜面反射相同的函数）并在反射光线的方向递归调用`cast_ray`函数。在现实中，光线可以反射多次。显然，在我们编写的程序中，反射必须要有限制，这里反射递归深度设为4。

```c++
    Vec3f reflect_dir = reflect(dir, N).normalize();
    Vec3f reflect_orig = reflect_dir*N < 0 ? point - N*1e-3 : point + N*1e-3;
    Vec3f reflect_color = cast_ray(reflect_orig, reflect_dir, spheres, lights, depth + 1);
```

反射做完，折射就很容易了。斯涅尔定律描述了光线从一种介质进入另一种介质时的折射规律。

数学形式如下：

$n_1\sin(\theta_1)=n_2\sin(\theta_2)$

- $n_1$ 和 $n_2$ 分别是光线所处介质的折射率
- $\theta_1$是光线在第一个介质中的入射角（光线与法线的夹角）
- $\theta_2$是光线在第二个介质中的折射角

折射函数如下：

- `I` 是入射光线的方向向量。
- `N` 是表面法线的方向向量。
- `refractive_index` 是第二个介质的折射率。

```c++
Vec3f refract( const Vec3f& I, const Vec3f& N, const float& refractive_index )
{
    float cosi = -std::max( -1.f, std::min( 1.f, I * N ) );
    float etai = 1, etat = refractive_index;
    Vec3f n = N;
    if ( cosi < 0 )
    {
        cosi = -cosi;
        std::swap( etai, etat );
        n = -N;
    }
    float eta = etai / etat;
    float k   = 1 - eta * eta * ( 1 - cosi * cosi );
    return k < 0 ? Vec3f( 0, 0, 0 ) : I * eta + n * ( eta * cosi - sqrtf( k ) );
}
```

先计算入射光线方向向量 `I` 和表面法线方向向量 `N` 之间的夹角余弦值 `cosi`。确定入射介质和出射介质的折射率比： `etai` 和 `etat`。如果入射角 `cosi` 小于 0（即光线从介质内部射向外部），则将入射角的余弦值取正，并交换入射介质和出射介质的折射率，并且将法线方向向量 `n` 取负。

根据斯涅尔定律，计算折射光线的方向向量，并根据折射角的余弦值和折射率比来确定折射光线的方向。如果折射率比和余弦值的平方乘积超过了1，表示发生全反射，此时光线无法折射进入第二个介质，函数返回黑色（即光线被完全反射）。

同样在`cast_ray`中添加折射的参数，此处我还修改了背景色，得到如下图像：

![refraction](https://cdn.jsdelivr.net/gh/s1san/asusual@img/img/refraction.png)

[这里](https://github.com/s1san/sbr/tree/main/BasicRayTracing/tutorial/step6)找到生成图像的代码。

## References

GitHub, ssloy, [Understandable RayTracing in 256 lines of bare C++](https://github.com/ssloy/tinyraytracer)

Github, RayTracing, [Ray Tracing in One Weekend Book Series](https://github.com/RayTracing/raytracing.github.io)

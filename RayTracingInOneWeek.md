# Ray Tracing In One Week

个人的渣翻（随缘翻，很多没翻完。翻译只是为了锻炼自己的英文水平。）以及笔记，[原网址](https://raytracing.github.io/books/RayTracingInOneWeekend.html)。原版英文其实十分简单，建议看原版。如有错误的地方，可以发邮件给俺，一起探讨探讨ヾ(≧▽≦*)o。俺的邮箱：liu_texwood5082@foxmail.com

[TOC]

## 1. 概述

这些年来，我已经教过许多图形学课程。我经常利用光线追踪来去教导学生，因为这样学生既可以专注于撰写全部代码，又可以不利用任何API获得很cool的图片。我决定将我的课程笔记改造成一个编程指南，来让你尽快获得一个很酷的程序。它将不会是一个功能很全的光线追踪器，但它包含间接光照的功能——这个功能使光线追踪成为电影技术的主力军。跟着这些步骤，你制作的光线追踪器的框架将会是一个很便于扩展的光线追踪器，如果你很兴奋并且想继续探讨光线追踪。

当一个人提及“光线追踪”，它可以代表很多东西。我将要描述的，严格意义上说，是一个普遍意义上的路径追踪器。虽然这些代码十分简单（让电脑运行起来吧！），我相信你仍然会为你所得到的图片感到兴奋。

我将根据我所编写的顺序指导你写出一个光线追踪器，并且提供一些debug的提示。最后你将获得一个可以产生很棒图片的光线追踪器。你仅仅只需要一周的时间就可以完成这些。即使你花了更长的时间，也不要因此而感到担心。我将使用C++作为驱动的语言，而你不需要。然而，我还是建议你使用C++，因为它很快、很便携的（portable，俺也不知道这里什么意思最恰当了），并且绝大多数电影、电子游戏的渲染器都是由C++撰写的。



## 2. 输出一张图片

### 2.1 PPM图片格式

无论你从什么时候开启一个渲染器，你始终需要找到一种可以看见图片的方法。其中最直接的方法就是把它写入文件。关键是，图片的格式有很多，其中许多都十分复杂。我总是从纯文本的ppm文件开始。以下是Wiki上关于ppm一个很好的描述：

![img](https://raytracing.github.io/images/fig-1.01-ppm.jpg)

* 注：ppm文件一般由4部分组成：第一行表示文件类型（P2、P3、P6等）、第二行表示图像的宽和高、第三行表示最大像素值、剩下则是图像数据块，按行存储，坐标原点在左上角。[ppm文件查看器](http://www.cs.rhodes.edu/welshc/COMP141_F16/ppmReader.html)。

### 2.2 创造一个图片文件



### 2.3 添加一个进度指示器



## 3. vec3类

几乎所有的图形程序都有用来存储几何向量和颜色的一些类。在许多系统中，这些向量是4维的（3维向量加上一维齐次坐标，或者是RGB的颜色加上一维alpha透明通道）。对我们来说，三维坐标足够了。我们将使用相同的`vec3`类来表达颜色、位置、方向、偏移量等等。一些人并不喜欢这样，因为它不能防止你做一些类似将一个颜色加到位置这样愚蠢的操作。他们的观点很好，但是我们将采取“少代码”的路线当没有明显错误时。尽管如此，我们还是为`vec3`声明了两个别名：`point3`和`color`。由于这两种类型仅仅只是别名，因此例如：如果你将`color`传给了需要`point3`的函数，你将不会受到警告。我们使用它们只是为了阐明意图和用途。

*俺想如果用两个继承了`vec3`的子类`point3`和`color`是否也能起到同样的效果，并且能够在赋错值的时候还收到提示*

### 3.1 变量和方法



### 3.2 有关vec3的内Utility Functions



### 3.3 有关颜色的Utility Functions

使用我们新创建的`vec3`类，我们可以创建一个十分实用的函数用于将单个像素的颜色输出至标准输出流。

```C++
#ifndef COLOR_H
#define COLOR_H

#include "vec3.h"

#include <iostream>

void write_color(std::ostream &out, color pixel_color) {
    // Write the translated [0,255] value of each color component.
    out << static_cast<int>(255.999 * pixel_color.x()) << ' '
        << static_cast<int>(255.999 * pixel_color.y()) << ' '
        << static_cast<int>(255.999 * pixel_color.z()) << '\n';
}

#endif
```



* 注：之所以用255.999作为系数，可以考虑两种由于浮点精度造成的情况：0.999×255.0和1.0×256.0。由于我们颜色的范围为[0,255]，我们要从[0,1.0]的小数映射到[0,255]的整数。如果是0.999×255.0，我们将得到254，如果是1.0×256.0我们将得到256，显然都不是我们想得到的255，所以使用255.999防止错误。

现在我们可以更改我们的main函数了：

```C++
#include "color.h"
#include "vec3.h"

#include <iostream>

int main() {

    // Image

    const int image_width = 256;
    const int image_height = 256;

    // Render

    std::cout << "P3\n" << image_width << ' ' << image_height << "\n255\n";

    for (int j = image_height-1; j >= 0; --j) {
        std::cerr << "\rScanlines remaining: " << j << ' ' << std::flush;
        for (int i = 0; i < image_width; ++i) {
            color pixel_color(double(i)/(image_width-1), double(j)/(image_height-1), 0.25);
            write_color(std::cout, pixel_color);
        }
    }

    std::cerr << "\nDone.\n";
}
```



## 4. 射线、一个简单的相机、以及背景

### 4.1 射线类



### 4.2 将射线射向场景中



## 5. 添加一个球



### 5.1 射线与球的相交



### 5.2 创建我们第一个由光线追踪生成的图片



## 6. 表面法线与多物体

### 6.1 使用表面法线着色



```c++
double hit_sphere(const point3& center, double radius, const ray& r) {
    vec3 oc = r.origin() - center;
    auto a = dot(r.direction(), r.direction());
    auto b = 2.0 * dot(oc, r.direction());
    auto c = dot(oc, oc) - radius*radius;
    auto discriminant = b*b - 4*a*c;
    if (discriminant < 0) {
        return -1.0;
    } else {
        return (-b - sqrt(discriminant) ) / (2.0*a);//求最近的点，即最小t
    }
}

color ray_color(const ray& r) {
    auto t = hit_sphere(point3(0,0,-1), 0.5, r);//计算出系数t
    if (t > 0.0) {//打中球
        vec3 N = unit_vector(r.at(t) - vec3(0,0,-1));//r.at(t)求出打中的点，vec3(0,0,-1)为球心，得到法线
        return 0.5*color(N.x()+1, N.y()+1, N.z()+1);//[-1,1]映射到[0,1]
    }
    //没打中球
    vec3 unit_direction = unit_vector(r.direction());
    t = 0.5*(unit_direction.y() + 1.0);
    return (1.0-t)*color(1.0, 1.0, 1.0) + t*color(0.5, 0.7, 1.0);
}
```



### 6.2 简化射线与球相交的代码

我们来回顾一下射线与球相交的函数：

```C++
double hit_sphere(const point3& center, double radius, const ray& r) {
    vec3 oc = r.origin() - center;
    auto a = dot(r.direction(), r.direction());
    auto b = 2.0 * dot(oc, r.direction());
    auto c = dot(oc, oc) - radius*radius;
    auto discriminant = b*b - 4*a*c;

    if (discriminant < 0) {
        return -1.0;
    } else {
        return (-b - sqrt(discriminant) ) / (2.0*a);
    }
}
```

首先，回想一下，一个向量点乘它本身得到它长度的平方。

其次，我们注意到，得到b的等式中有一个系数2。考虑一下如果$b=2h$二次方程会发生什么：
$$
\begin{aligned}
& \qquad \frac{-b\pm{\sqrt{b^2-4ac}}}{2a} \\
&= \frac{-2h\pm{\sqrt{(2h)^2-4ac}}}{2a} \\
&= \frac{-2h\pm{2\sqrt{h^2-ac}}}{2a} \\
&= \frac{-h\pm{\sqrt{h^2-ac}}}{a} \\
\end{aligned}
$$
使用以上我们通过观察得出的结论，我们可以将代码简化为以下形式：

```C++
double hit_sphere(const point3& center, double radius, const ray& r) {
    vec3 oc = r.origin() - center;
    auto a = r.direction().length_squared();
    auto half_b = dot(oc, r.direction());
    auto c = oc.length_squared() - radius*radius;
    auto discriminant = half_b*half_b - a*c;

    if (discriminant < 0) {
        return -1.0;
    } else {
        return (-half_b - sqrt(discriminant) ) / a;
    }
}
```



### 6.3 对可命中物体的抽象



### 6.4 正面与背面的对比



### 6.5 关于可命中物体的列表



### 6.6 一些新的C++特性



### 6.7 常用的一些常量与Utility Functions



## 7. 反走样


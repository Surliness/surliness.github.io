[《GAMES101》作业框架问题详解 - 知乎](https://zhuanlan.zhihu.com/p/509902950)

[【Games101】作业解析_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1o5411e7cf?spm_id_from=333.788.videopod.sections)，代码也主要参考了这个up的，[Games101_Homework-master](https://github.com/Azalea8/Games101_Homework)

[GAMES101 作业问题整理 - 知乎](https://zhuanlan.zhihu.com/p/375391720)

主要是实验过程中遇到的一些问题，以及作业框架中涉及的一些算法。

## PA0

CMakelists中的`include_directories(EIGEN3_INCLUDE_DIR)`要修改为`include_directories(${EIGEN3_INCLUDE_DIR})`，否则 `cmake` 会将其视为普通字符串而非变量。如果eigen不在cmake的默认搜索路径中，就会在`make`编译时报找不到头文件的错误，但是是可以cmake的

如果使用vscode中的cmake插件，可以在`setting.json`中指定cmake搜索的路径。不过不建议这样做，因为这样会设置全局的路径，除非每一个工程都各自配置一个`.vscode`文件夹，不过我比较懒。

### `cmake ..`的时候出现找不到`xxx.cmake`的错误

是cmake定位不到eigen库的原因，需要指定`CMAKE_MODULE_PATH`到eigen的cmake文件夹。可以在GUI中add entry，也可以用命令行`-DCMAKE_MODULE_PATH=path_to_eigen/cmake`，或者直接在CMakelist中指定。

### 命令行`cmake..`没有生成makefile

命令行改为`cmake -G "MinGW Makefiles" ..`指定生成的文件类型，否则CMake 不会默认生成 `Makefile`，而是根据指定的生成器生成对应构建系统的文件（如 Makefile、Visual Studio 项目、Ninja 脚本等）。如果未明确指定生成器，CMake 会根据系统环境自动选择，可能导致生成的不是Makefile。

vscode可以在`setting.json`中指定`"cmake.generator": "MinGW Makefiles"`

## Assignment1

### opencv库配置相关

预先编译好的opencv库可能会由于使用的编译器版本不同，opencv版本低等原因而不能使用，所以可能需要重新根据使用的编译器版本重新编译生成链接库。

如果想使用vscode中的run 直接运行exe文件，window下需要在launch.json中指定外部库的bin文件夹路径。

```json
"environment": [
    // {
    //     // 将库目录添加到 Path 环境变量中
    //     "name": "Path",
    //     "value": "${env:path};${cwd}..\\..\\..\\..\\Cpp_program\\Library\\opencv\\install\\x64\\mingw\\bin;",
    // },
    {
        // 直接指定需要依赖的 Path 环境变量，需要自己试验才知道具体用了哪一个
        "name": "Path",
        "value": "C:\\Users\\CodeBlocks\\MinGW\\bin;${cwd}..\\..\\..\\..\\Cpp_program\\Library\\opencv\\bin;"
    },
],
```

如果想要双击运行exe文件，需要配置系统环境变量，才能让该exe找到opencv的链接库。在命令行中，可以采用添加临时环境变量的方法添加需要的路径后使用`./***.exe`运行，如在powershell中，可以通过`$env:PATH += ";C:\Users\ProgramPractice\Cpp_program\Library\opencv\bin"`添加所需路径，也可以直接将dll库复制到该exe所在目录下。

#### 颜色通道问题

框架代码在设置颜色时，包括存储在颜色缓存中的颜色次序是按照012对应显示RGB，但是opencv的存储是按照通道编号012对应显示BGR，因此需要进行转化。使用`cv::imread()`时也是按照通道编号012对应显示BGR的次序存储。

```c++
void Triangle::setColor(int ind, float r, float g, float b)
{}

cv::cvtColor(image, image, cv::COLOR_RGB2BGR);
//cv::COLOR_RGB2BGR 或cv::COLOR_BGR2RGB的效果都是交换RB两个通道，选一个用即可。
```

### cmake module路径问题

明明路径中包含了 OpenCVConfig.cmake 文件，但是 CMake 仍然找不到？

[听说你安装测试 OpenCV 总是不成功？你可能遇到这个find_package坑了！ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/106473031)

> find_package() 有两种模式: Module 模式和 Config 模式，分别对应上面的 Findxxx.cmake 和xxxConfig.cmake 两个文件。CMake 默认优先 Module 模式，而 Config 模式是备选项。
>
> CMake 会优先搜索 CMAKE_MODULE_PATH 指定的路径，如果在 CMakeLists.txt 中没有设置 CMAKE_MODULE_PATH 为存储 Findxxx.cmake 的路径，也就是说没有下面的指令： set(CMAKE_MODULE_PATH "Findxxx.cmake文件所在的路径")，那么 CMake 不会搜索 CMAKE_MODULE_PATH 指定的路径，此时 CMake 会搜索第二优先级的路径，也就是 /share/cmake-x.y/Mdodules
>
> CMake 会优先搜索 xxx_DIR 指定的路径。如果在 CMakeLists.txt 中没有设置这个 CMake变量。也就是说没有下面的指令: set(xxx_DIR "xxxConfig.cmake文件所在的路径")，那么 CMake 就不会搜索 xxx_DIR 指定的路径，此时 CMake 就会自动到第二优先级的路径下搜索，也就是 /usr/local/lib/cmake/xxx/ 中的 xxxConfig.cmake 文件。

### 三角形是倒过来的

[关于GAMES101作业1渲染出的三角形颠倒问题的个人解答 - 知乎](https://zhuanlan.zhihu.com/p/618878989)

> 实际上是因为传入的far和near是正值，导致“取景空间”（那个frustum）在摄像头朝向的背面。三角形实际上已经不在取景空间里了，不应该显示。但是那个作业代码没有对在canonical空间中z超出[-1,1]的部分做裁剪，于是即便超出了canonical cube的范围也会继续显示出去，显示出的三角形就跟小孔成像一样反过来了。 Mpersp->ortho不需要改，要么改far和near，要么改摄像头朝向。

我写的时候，保持相机朝向-z方向，**尝试在所有的射影长度计算时始终将near/far当作坐标看待**。这么做能够保留射影的方向信息，但在计算正交投影的伸缩因子或者其他参数的尺寸时需要额外判断是否是以大减小。所写代码中采取的方法是改变传入值的正负号，并且加入交换判断保证以大减小。

```c++
Eigen::Matrix4f get_projection_matrix(float eye_fov, float aspect_ratio, float zNear, float zFar)
{
    // ...
    
    if (b > t)
        std::swap(b, t);
    if (l > r)
        std::swap(l, r);
    
    //...
}
int main()
{
    // ...
    
    // 将视锥转换到z轴的负半轴，因为物体在负半轴，否则穿过焦点会形成倒像
	// r.set_projection(get_projection_matrix(45, 1, 0.1, 50));
	r.set_projection(get_projection_matrix(45, 1, -0.1, -50));
    
    //...
}
```

`draw`函数中的`vert.z() = vert.z() * f1 + f2;`做了深度的线性变换，但是似乎直接`vert.z() = vert.z();`也并没有影响。

### 逆时针转过底边的时候会数组越界

[作业1 绕X轴旋转会数组越界 – 计算机图形学与混合现实在线平台](https://games-cn.org/forums/topic/zuoye1-raoxzhouxuanzhuanhuishuzuyuejie/)

>  `rasterizer.cpp`的`set_pixel()`中，如果`x`和`y`同时为 0，`ind`此时等于`height * width`，但是数组范围为`[0, height * width – 1]`，因此出现了越界解决方法是不允许`x`和`y`同时为 0，也就是把小于号改成小于等于号。

### Bresenham画线算法

[一文读懂Bresenham画线算法 - 知乎](https://zhuanlan.zhihu.com/p/659289699)

> 1. Bresenham算法是一个增量算法，依据$x_k$推导出下个像素$x_{k + 1}$的坐标。
> 2. Bresenham算法只使用整数运算，在图形学刚刚起步的年代，计算效率非常重要，整数运算要比浮点、乘除运算快。

[Bresenham画直线算法（所有斜率） - 明明1109 - 博客园](https://www.cnblogs.com/fortunely/p/17660786.html)

### 视角fov

调大视角，所得的结果是三角形变小，相当于远离了屏幕，这似乎和视角变大相当于视点靠近屏幕的"近大远小"相违背。这里需要理解成像视锥的视角和物体与视点构成的视角之间的区别。

物体与视点构成的视角：指的是空间中固定尺寸的物体，与视点构成的角度。当物体远离视点时，该角度会变小，物体靠近视点时，该角度会变大。这是"近大远小"。

成像视锥的视角：渲染中的FOV指的是Camera视锥体接收画面的角度，即视锥体的顶角。顶角越大，锥体覆盖到的内容也就越多。覆盖范围变化了，但对于同一物体，不同FOV的Camera观察到的是完全一致的，只是**大FOV的Camera因为要在固定大小的屏幕内渲染更大的范围**，所以那些在小FOV下观察到的内容经过透视变换被压缩显示在了屏幕的中央区域。

[初探FOV - 知乎](https://zhuanlan.zhihu.com/p/312194696)

在屏幕尺寸固定的条件下，其实视角调大确实距离屏幕更近了，但是由于要渲染的内容增多(视锥覆盖的内容变多)，物体压缩到屏幕中的尺寸就变小了。看起来就像变远了。

## Assignment2

### 初始化depthbuffer 和注意z values 的符号

> 请注意我们是如何初始化depth buffer 和注意z values 的符号。为了方便 同学们写代码，我们将z进行了反转，保证都是正数，并且越大表示离视点越远。为了方便同学们写代码，我们将z进行了反转，保证都是正数，并且越大表示离视点越远。

**注意**：自己写的时候，由于保持相机朝向-z方向，所以需要针对**传入的数值和深度的初始值**进行修改。

### 透视深度插值修正

由于进行深度插值修正时还需要使用到齐次坐标的权系数，因此再归一化齐次坐标时，权系数应该予以保留。不过不改，视觉上也能得到正确的结果。

```c++
//Homogeneous division
for (auto& vec : v)
{
    // vec /= vec.w(); 改为
    vec.x() /= vec.w();
    vec.y() /= vec.w();
    vec.z() /= vec.w();
    // 这个问题在作业三中修正了
}

```

对于插值修正，作业的框架有问题。`w_reciprocal`已经是三维空间的正确深度信息，`z_interpolated`最后值为原本是为了表达透视校正插值后的二维平面下正确坐标深度信息。

```c++
auto [alpha, beta, gamma] = computeBarycentric2D(pxc, pyc, t.v);
float w_reciprocal = 1.0 / (alpha / v[0].w() + beta / v[1].w() + gamma / v[2].w());
float z_interpolated = alpha * v[0].z() / v[0].w() + beta * v[1].z() / v[1].w() + gamma * v[2].z() / v[2].w();
z_interpolated *= w_reciprocal;
```

> **但是**`v[0].w()`，`v[1].w()`，`v[2].w()` 这三个值因为做了 `auto v = t.toVector4()` 这一步骤之后，值都为 1，而**实际上应该是三角形顶点的深度值**。

而且`.z()`是齐次坐标归一化的值，并不代表原空间的属性，这里相当于用的是投影屏幕的属性。所以总的问题还是出在对顶点向量的处理上。[计算机图形学中，如何推导透视校正插值？ - 知乎](https://www.zhihu.com/question/332096916/answer/2408545417)

我重新传入了齐次坐标的权重，但是要严格按照插值意义需要改动的地方有点多，就没有再修改`.z()`的内容，所以写的代码相当于还是将投影屏幕的深度作为原空间的属性。更好的方法是将`Triangle`类中的顶点向量修改为4维，然后重新把对应的类函数重写一遍。作业3就这么做了。

## Assignment3

作业框架修改了插值公式的名称以及`Triangle`类中顶点向量的维数。

### 框架代码公用部分说明

#### eye_pos

每个片段着色器函数中`eye_pos`都指定为`{0,0,10}`，我怎么感觉这个相机位置应该作为参数传入着色器更好一点。

#### 依赖名称解析

[c++11-17 模板核心知识（十四）—— 解析模板之依赖型模板名称 - 知乎](https://zhuanlan.zhihu.com/p/339183858)

如果不适用`auto`而是直接指定具体类型，那么则不需要显示地使用`template`关键字对成员函数进行修饰。

```c++
// 当 v 的类型是模板参数（例如在泛型 lambda 中 auto& v，或模板函数中 T v），编译器无法确定 v.head<3> 中的 < 是“小于号”还是“模板参数开始”。
// 因此必须用关键字 template 显式告诉编译器：“head 是一个模板函数”
std::transform(mm.begin(), mm.end(), viewspace_pos.begin(), [](auto& v) {
    return v.template head<3>();
});
```

#### `shader.hpp`中的`view_pos`

指的是视觉(相机)空间中的需要进行着色的着色点坐标，不是世界空间中的相机坐标。

### 插值部分

#### interpolated_shadingcoords是啥

1. [作业3 interpolated_shadingcoords – 计算机图形学与混合现实在线平台](https://games-cn.org/forums/topic/zuoye3-interpolated_shadingcoords/)
2. [graphics - How exactly does OpenGL do perspectively correct linear interpolation? - Stack Overflow](https://stackoverflow.com/questions/24441631/how-exactly-does-opengl-do-perspectively-correct-linear-interpolation#:~:text=Perspective correct interpolation,-So let’s say&text=First we calculate its barycentric,on the 2D triangle projection.）)

其实就是相机空间中需要渲染的坐标

#### 是否需要插值矫正

由于框架中使用的重心坐标由屏幕空间的顶点计算，因此对于相机空间的插值，严格来说都是需要针对所得重心坐标进行插值矫正。但框架中的插值函数只是做了个线性组合，因此我根据插值矫正公式重新写了个插值函数。虽然结果上和网上大部分不做插值矫正的结果区别不大😂，但至少逻辑上比较能自洽一点。

```c++
static Eigen::Vector3f interpolate_from_screen_space(float alpha, float beta, float gamma, const Eigen::Vector3f &vert1, const Eigen::Vector3f &vert2, const Eigen::Vector3f &vert3, const Eigen::Vector3f &z_vert)
{
    float weight = (alpha / z_vert[0] + beta / z_vert[1] + gamma / z_vert[2]);
    return (alpha * vert1 / z_vert[0] + beta * vert2 / z_vert[1] + gamma * vert3 / z_vert[2]) / weight;
}
```

> 还是有人也考虑到这一点的👏[Games101作业三回顾 - 哔哩哔哩](https://www.bilibili.com/opus/1099626389334654982)

### 任务2的实现

法向量的着色器函数的颜色是根据法向量的值计算出来的，没有使用到三角形顶点设定的颜色值。相当于可视化法向量。

### 任务3的实现

#### phong模型

环境光应该放在光线的循环外计算。它模拟的是“场景的基底亮度”，不应随光源增减而剧烈变化。（不过也有认为环境光应该和光源数有关）。我写的时候是将环境光作为全局使用的，因为传统的phong模型是将环境光设为常量。

> [11.1 Phong 模型 | Shadertoy中文教程](https://shadertoy.peakcoder.com/phong/phong)

最终颜色就是高光、环境光、散射光的直接相加，不需要额外归一化到$[0,1]$。颜色值可以超过 1.0，表示“过曝”（物理上合理的高亮度）。

Phong 模型计算的是“表面亮度”，这个亮度与观察者位置无关（除非视线方向影响镜面反射角度）。 在计算公式中的距离时，是光源到物体表面的距离，与物体表面到观察者的距离无关。

### 任务4的实现

#### 纹理坐标取值不在$[0,1]$

给的.obj文件中纹理坐标 vt 存在超出区间$[0,1]$的情况，而代码框架在获取纹理映射的`getColor()`中没有对`u,v`的边界进行判断，会导致再获取图片颜色时超出图片范围导致报错，需要自行加上。

在处理 PNG 图像时，可能会遇到 `libpng warning: iCCP: known incorrect sRGB profile` 的警告。这通常是由于图像文件中嵌入的 sRGB 颜色配置文件不符合 libpng 的标准。可以不理它，问题应该不大。

#### 切线计算公式

框架中计算切线的方法是将法向量旋转九十度获得，用球坐标推即可。使用这种方法得到的TBN矩阵不需要再进行正交化与单位化。不过注释所给的计算方法y分量漏了个负号。

### 任务5的实现

该任务也是实现向量贴图后的可视化结果。给的图也不是高度图，更像是法线贴图，需要考虑怎么根据采样点的RGB信息计算"高度"。根据课程助教的答疑，使用范数作为高度。

> 正规的凹凸纹理应该是只有一维参量的灰度图，而本课程为了框架使用的简便性而使用了一张 RGB 图作为凹凸纹理的贴图，因此需要指定一种规则将彩色投影到灰度，而我只是「恰好」选择了 norm 而已。为了确保你们的结果与我一致，我才要求你们都使用 norm 作为计算方法。

这里实在卡了很多时间，要不是找到这个说明，还以为法线贴图和凹凸贴图有什么必然联系没学到来着。

### 任务6的实现

对框架所给的计算点位移的方式比较困惑，为什么沿着顶点之前的法向量方向位移？

```cpp
// Vector ln = (-dU, -dV, 1)
// Position p = p + kn * n * h(u,v)
// Normal n = normalize(TBN * ln)
```

解释：位移贴图储存的内容也是高度，和凹凸贴图的储存内容类似。位移贴图的过程要确定位移方向和距离，距离一般指的是贴图曲面与原曲面的高度差，该值可以从高度图中获取，而高度的变化一般用该顶点的原法线方向描述。所以整个流程就是：法向移点，更新法向量。

### 提高项

rock.obj中的顶点坐标没有归一化，存在不属于$[-1,1]$的值，会导致在Viewport transformation处理后坐标越界，超出光栅化的范围。实现的时候需要自行限制一下范围。比如粗暴一点直接除以2。bunny.obj是坐标太小了，也应该先合理的缩放到$[-1,1]$。我给光栅化的类加了一个归一化因子。bunny的效果也没有很好。

crate.obj没有三角形，也没有提供读取.3ds的函数，所以我使用了cube.obj+crate材质，效果一般。

## Assignment4

提高项中的反走样，意思是对于需要显示的点列，其周围的像素也应该根据像素中心到该点的距离进行着色。不是像双线性插值那样拿周围点的颜色计算该点的颜色。

## Assignment5

代码框架中射线追踪使用的模型是Whitted-style light transport algorithm，其实严格来说并不算完整的路径追踪，所以框架中的函数命名是`castRay`

存储ppm文件的时候是先写入缓存中索引小的值，并且从上往下，从左至右开始存，因此相当于渲染的图像中y的索引向下增长，在推导像素索引对应的方向向量的时候注意索引与坐标的关系。

### 偏置处理

在renderer.cpp里，使用偏置处理表面因为浮点精度产生的自交问题。法线的计算方法是规定好的，意味着框架中的三角形法线都朝向表面的外侧(看代码实现)。而当光线在表面内侧发生反射时，反射向量就会与表面法线夹钝角。

光线与表面的交互点应该与入射光线位于同一侧，但是由于浮点精度的存在，并不能很好的保证，所以需要使用偏置处理保证这一关系。在统一规定表面法线朝外的条件下（这个前提改变后面的判断也要跟着修正），当$\vec{r}\cdot\vec{n}<0$时，交互点应沿着法线反方向向内移动；当$\vec{r}\cdot\vec{n}>0$时，交互点应沿着法线方向向外移动。

```c++
Vector3f reflectionDirection = normalize(reflect(dir, N));
Vector3f reflectionRayOrig = (dotProduct(reflectionDirection, N) < 0) ? hitPoint - N * scene.epsilon : hitPoint + N * scene.epsilon;
```

不做这一步偏置修正，可能在判断反射光线时，会与改交互点所在的表面相交。

### if语句中初始化

renderer.cpp中`if (auto payload = trace(orig, dir, scene.get_objects()); payload)`可参考[if statement - cppreference.com](https://en.cppreference.com/w/cpp/language/if.html)中的if statements with initializer部分。

## Assignment6

### 框架代码说明

代码框架中射线追踪使用的模型是Whitted-style light transport algorithm，其实严格来说并不算完整的路径追踪，所以框架中的函数命名是`castRay`

#### 判断射线是否会与三角形相交

在triangle.cpp中判断射线是否会与三角形相交的算法中，使用射线和三角形的法向量的夹角判断射线是否从三角形的正面交互。正常来说交互点只会在物体外侧，但是由于浮点精度的问题有时外部的点可能会变成内侧点。

```c++
// 由于定义的三角形顶点编号0-2统一按逆时针排列，而且统一了法线计算方法，所以三角形法线方向统一
// 交互判断算法中 det = -d * normal, normal = e1.cross(e2),则 det < 0 代表射线方向背离三角形所在平面，等价于下面的判断，
// 即射线与三角形非法线方向的面交互，对于单面渲染，这种情况无需渲染
if (dotProduct(ray.direction, normal) > 0)
    return inter;

```

#### 包围盒构建

BVH.hpp中选择**包围盒最长轴**的包围盒质心中位数进行划分，这样能最大程度减少子节点包围盒的重叠

> 一个扁平矩形场景（x轴很长，y/z很短），若沿短轴（y）划分：左右子树物体数相等，但包围盒几乎完全重叠；若沿长轴（x）划分：包围盒分离明显，重叠少。

### 包围盒的距离理解

需要处理的是o到包围盒边界pMin，pMax的距离中，哪一个对应到达，哪一个对应穿出。

对于负值距离，在几何上代表的是从o出发沿射线反方向到达目标的长度。这个到达和包围盒中的到达不是一个意思，**包围盒中的到达指定的是**，在$\vec{o}+t\vec{d}$这样一条直线中，沿着该$\vec{d}$方向（同向为正无穷，反向为负无穷，不是世界坐标系），**从负无穷开始，在以o为原点的坐标系中达到包围盒边界所在的坐标**。

> 理解一下上面所说的内容，我写的时候就是没分清，**导致不知道当出现pMin，pMax包含o的情况时，到达边界和穿出边界怎么定义**，因为当时觉得负的有向距离也是指向穿出包围盒边界的方向，没意识到包围盒的穿出是针对直线上从负无穷沿着d行进的方向而言。

其实下面两种理解只是角度不同，使用o为原点的坐标系就直接比大小即可，此时可以将t_pMin、t_pMax的计算看成是相对于o的，使用世界坐标系就得考虑o与包围盒的相对位置，将t_pMin、t_pMax看成是以pMin、pMax为主，调整后才能再赋给o，因为当射线方向与世界坐标方向不一致时，可能pMax对应的到达边界（比如pMin < pMax < o的情况）：

1. 从以o为原点的坐标角度，负号代表已经到达过了，绝对值代表在多远的地方到达，越负代表到达越早；正号代表还未到达，越正代表到达越晚；符号坐标的大小关系就可以表示是到达包围盒还是穿出包围盒；
2. 从世界坐标系中朝向和位置的关系：直接得出到达和穿出的距离分别应该对应pMin还是pMax。

```c++
// 到达包围盒边界的距离
Vector3f t_pMin = (pMin - ray.origin) * invDir, t_pMax = (pMax - ray.origin) * invDir;
// 下面两种理解均可，以一维为例
// 理解一： 以o为原点的坐标系中，t_enter、t_exit代表的是沿着d方向，进入和离开包围盒的位置到o的坐标，因此t_enter一定是两者中较小的那一个。
Vector3f t_enter = Vector3f::Min(t_pMin, t_pMax), t_exit = Vector3f::Max(t_pMin, t_pMax);
// // 理解二：世界坐标系中，当d为正时，沿着该方向的直线(起点为负无穷)一定先通过pMin，
// // 当d为负时，沿着该方向的直线(起点为正无穷)一定先通过pMax，
// // 因此需要根据d的正负给t_enter、t_exit赋值
// Vector3f t_enter = t_pMin, t_exit = t_pMax;
// if(!dirIsNeg[0])
//     std::swap(t_enter.x, t_exit.x);
// if(!dirIsNeg[1])
//     std::swap(t_enter.y, t_exit.y);
// if(!dirIsNeg[2])
//     std::swap(t_enter.z, t_exit.z);
```

### 怎么渲染那么久？我的电脑太破了？

```shell
Time taken: 0 hours     
          : 23 minutes  
          : 1399 seconds
```

[作业6 提高题结果比较(提升似乎有限？) – 计算机图形学与混合现实在线平台](https://games-cn.org/forums/topic/zuoye6-tigaotijieguobijiaotishengsihuyouxian/)

[作业6 渲染时间很长 – 计算机图形学与混合现实在线平台](https://games-cn.org/forums/topic/zuoye6-xuanranshijianhenchang/)

我吐了，判断是否BVH.cpp中判断相交的时候逻辑错了，漏了个非，之前相当于把所有的三角形都检测了一遍，改了之后加速效果明显。还好debug时第一个包围盒就不符合条件，跳出来的时候却没有直接进入分支，不然不知道要什么时候才能发现😂。

```c++
// if (node == nullptr || node->bounds.IntersectP(ray, ray.direction, dirIsNeg))
if (node == nullptr || !node->bounds.IntersectP(ray, ray.direction_inv, dirIsNeg))
{
    // ...
}

// 传参写错了，应该是要传入ray.direction_inv的，难怪渲染那么快，而且渲染结果是一片蓝😂
// 不懂为什么之前还能正确渲染出来，是后面写svh时发现没有一个包围盒相交才找到这个bug。
BVH Generation complete: 
Time Taken: 0 hrs, 0 mins, 1 secs

 - Generating BVH...

BVH Generation complete:
Time Taken: 0 hrs, 0 mins, 0 secs

Render complete: ======================================================] 100 %
Time taken: 0 hours
          : 0 minutes
          : 2 seconds
```

### 提高项

<img src="http://15462.courses.cs.cmu.edu/fall2015content/lectures/10_acceleration/images/slide_026.png" alt="img" style="zoom:50%;" />

#### 实现逻辑

网上有不少是按照在枚举切分边界的循环中遍历物体在边界的左侧还是右侧，这种方法对于$k$个bucket和$n$个物体而言，所需的时间复杂度是$O(kn)$。如果按照作业参考链接中的方法，遍历物体判断其属于哪个bucket(索引是计算边界坐标的逆过程)，时间复杂度为$O(k+n)$。

讨论区的相似提问[作业6 提高题结果比较(提升似乎有限？) – 计算机图形学与混合现实在线平台](https://games-cn.org/forums/topic/zuoye6-tigaotijieguobijiaotishengsihuyouxian/#post-5177)

> 1.分 bucket 是工程上的做法，不然真的按 object 切分的话每一次找切分位置运算量太大，所以也别按物体切了
>
> 2.你可以理解总 bucket 就是 centroidBounds 一个包围所有 object 的 bound，附件的 3 行其实就是算每个 object 在这个大 bound 中的相对位置也就是 offset，然后乘以 bucket 的数量就是该 object 所属 bucket 的index，那就 bucket[i].union(该 object) 就算把该 object 划分到了 bucket[i] 中

与作业参考思路一致写法：

1. [SAH加速结构 - 知乎](https://zhuanlan.zhihu.com/p/501370100)

   > 和我的写法逻辑大致是一样的。参考中想到使用bounds类中提供的offset函数，通过质心在大包围盒的偏置量计算出bucket索引，这个方法好👍。

2. [从零开始学图形学：通俗讲解简单加速结构——k-d tree, SAH, BVH - 知乎](https://zhuanlan.zhihu.com/p/349594815)

因为提供的代码框架每次划分的时候考虑的是质心分布最长的一条轴，所以不用每次都考虑三个轴。

#### 特殊情况的处理

<img src="http://15462.courses.cs.cmu.edu/fall2015content/lectures/10_acceleration/images/slide_027.png" alt="img" style="zoom:50%;" />

对于以下两种情况其实debug的时候也遇到了，只有一个bucket里面有物体，而且由于使用的包围盒作为划分边界，精度问题导致每次划分物体质心都划为一个bucket，导致下一轮递归时边界不缩小。这个问题后面改成使用质心作为边界后解决了，无论多靠近，总会在有限次内到达递归出口，即质心包围盒内的物体数量为1或2。

图中左边的这种二维平面上的质心重叠，并不能通过上诉方法解决，不停细分的话，bucket的步长会趋于零，计算可能会溢出。我在代码中使用当只有一个bucket非零的时候直接返回这个bucket内部物体的包围盒(因为最长轴的划分都不能将物体区分成至少两块，那说明没有必要分割下去了)，虽然没有被触发过。如果计算的bucket索引`(obj->getBounds().Centroid()[dim] - p_min) / bucket_step`能够趋于同一个整数那么就有可能被触发。

> 但是对于这种二维平面的情况，应该需要更合适的细分方法。
>
> 使用提供的模型时没有出现划分时有一边为零的情况，但是如果由于比较浮点数的精度问题，或者极值在边界取得等情况，出现存在上诉现象是可能出现的，因此在框架中补了一个针对传入的物体向量为空的递归出口，直接返回一个空节点。

右边的情况是为了说明，这种已经具有相同包围盒的三角形，即使成功划分，两个子节点的包围盒也会完全相同。光线很可能同时与两个子节点相交，失去了BVH的空间划分优势，划不划分区别不大。

#### 精度问题

一些精度导致的问题，根据图中的划分方法与算法写的，但是其实有不少地方有问题的。两点钟半小时写出来的代码，debug搞到六点，回去睡觉下午4点过了又弄了两个小时才搞通。😤，睡了个觉起来对着网上的参考一查就发现错误，看来有时还是需要适当暂停一下再继续。

转化为索引的时候记得限制索引范围为`[0, buckets - 1]`，否则会出现越界的情况。

```c++
// 可复刻的错误（不同电脑可能就不一定了）
float p_min = bounds.pMin[dim], p_max = bounds.pMax[dim];	// 这个应该用质心构成边界
float bucket_size = (p_max - p_min) / buckets;
float p_spilt = std::min(p_min + dd * (min_cost_idx + 1), p_max);
// 判据1
if (obj->getBounds().Centroid()[dim] < p_spilt)
{}
// 无限递归，报错数据:
// svh_node_num[3]=2 && svh_node_num[6]=1
// svh_bounds_area[3]=0.617485285 && svh_bounds_area[6] =0.262827456
// 原因: 切分边界为bucket=3, 边界=0.771506071，但所有本应划分到左边的质心=0.771506071的物体都被划分到了右边

// 判据2
if (obj->getBounds().Centroid()[dim] <= p_spilt)
{}
// 无限递归，报错数据:
// svh_node_num[0] == 1 && svh_node_num[3] == 2 && svh_node_num[4] == 1
// svh_bounds_area[0] = 0.0749589279 && svh_bounds_area[3] =  0.487371176 && svh_bounds_area[4] = 0.293007851
// 原因: 切分边界为bucket=3, 边界=-1.46954274，但所有本应划分到右边的质心=-1.46954274的物体都被划分到了左边
```

> 按理说划分后包围盒应该会变小，但是为什么会以相同的物体递归？
>
> 因为划分bucket的边界取的是包围盒边界，而判据用的是质心。精度问题使得所有质心归为了同一边，"划分bucket的边界取的是包围盒边界"导致下一轮的划分边界的范围没有改变，质心落在的bucket没有改变，然后继续因为精度问题使得所有质心归为了同一边，如此重复产生了无限递归。
>
> 可以将划分bucket边界取为质心，这样上面使用浮点数判断的方法就可以正常运行了，虽然可能依然存在有问题，只是提供的模型没有触发而已。

**图中所用算法是以物体的最小最大边界作为划分起终点，这有点误导了，因为判据都是依赖质心，用质心可能更合适一点。**

判断质心属于哪个`bucket`的时候，最好将质心转化为索引，否则划分物体时，计算`bucket`边界后与质心进行比较的时候，需要考虑浮点数的精度问题。

> 将质心转化为索引，虽然有点多此一举，并竟直接根据索引算出坐标边界后，直接遍历物体质心比大小即可，而且递归的特殊处理能帮忙擦屁股，减少出错的概率。不过感觉使用直接比较的方法判断浮点数的边界还是尽量谨慎。

用了SAH，比BVH7.6秒快了个0.3秒左右😂。

## Assignment7

[GAMES101作业7及课程总结（重点实现多线程加速，微表面模型材质） - 知乎](https://zhuanlan.zhihu.com/p/606074595)

### 作业说明

#### 向量方向

> 为了与之前框架保持一致，$\omega_o$定义与课程介绍相反

这个是指课程中表面的立体角$\omega_i,\omega_o$都是由着色点指向外的，这也是一般遵循的规则，即离开表面方向。而原来的框架中射线是由屏幕发出，因此击中物体时$\omega_o$是由外部指向着色点。因此在针对间接光照递归调用`shade(p,wo)`时，直接使用`shade(q, wi)`即可，不用使用$-\omega_i$，因为此时对于q而言，$\omega_i$已经是由外部指向q的了。

框架的代码是把视点放在$z=-800$的位置，视线朝向$z=+1$方向，obj文件中的问题都位于$z\ge0$的位置，这也对应了击中物体时$\omega_o$是由外部指向着色点。

#### 光源

课程在介绍路径追踪的伪代码时，略去考虑着色点为光源的情况，伪代码是计算积分项的。但是算法为了避免着色点使用蒙特卡洛积分法递归计算发生射线的指数增长，因此把蒙特卡洛积分法中的采样换成了对像素进行多次采样(方向是不变的)。因此如果着色点是光源，直接光照应该初始化为光源的辐射值。

其实光源的强度可以进入蒙特卡洛积分的计算中，而是直接添加在renderer.cpp的像素采样前（求值的结果是一样），让像素采样专门负责计算其他直接光源的直接光照和间接光照。

#### 代码框架

```c++
float Material::pdf(const Vector3f &wi, const Vector3f &wo, const Vector3f &N);
Vector3f Material::eval(const Vector3f &wi, const Vector3f &wo, const Vector3f &N);
// 这里的wi指的是从相机发出的射线的入射光线，而不是来自光源的入射光线
// 与之对应的，wo指的是指向(远离表面方向)来自光源方向的射线，而不是指向来自相机方向的射线。
```

传参的时候需要注意，否则会出现[黑色噪点问题](#高的方块阴影靠墙侧黑色噪点很多)。代码框架的wo，wi似乎是从相机开始定义的（存疑，实现微表面的时候在好好确认一下）。

来自相机的射线在针对同一个像素采样时击中的是同一个着色点，即"入射光线"方向是固定(从相机发出光线的角度看是wi，从光源的角度看是wo)。此时需要对来自不同方向的光线进行采样。因此采样所获得的方向是指向来自光源的射线的，在该框架中应该对应函数形参wo，否则传入的`eval(wi, -ray.direction, N);`

一般定义双向反射分布函数时，wi针对的是从光源到着色点的方向，wo是针对着色点去往相机的方向。这里不要弄混了。

> [Lambertian材质漫反射BRDF方程的解释与推导 - 知乎](https://zhuanlan.zhihu.com/p/1897962300603864554)
>
> 看了一下推导，我感觉是不是作业参考的传参写错了呢？🤔

### 后面的墙壁出现黑条纹

[Games101 作业7 绕坑引路 (Windows) – 计算机图形学与混合现实在线平台](https://games-cn.org/forums/topic/games101-zuoye7-raokengyinlu-windows/)

判断是否计算直接光照的阈值需要加上一个偏置，这个应该是浮点数带来问题。

```c++
// 取小于等于0.001的值会导致矮的方块的右上角没有直接光照
if (inter_middle.distance + 0.001 >=  p_l_dis)
```

我还遇到了矮的方块的右上角没有直接光照，只有右边绿色墙壁反射的间接光照的诡异问题。打印distance后发现distance还会出现负值。这个是因为在三角形求交的函数里没有排除distance<0的情况导致的，从作业6拖到作业7的bug😂🤪。

### 白噪点很多，墙壁与天花板交界是有对面墙的颜色

递归写错了😤，有点被伪代码的`shade(q, wi)`用的是下一个着色点搞混了，忘记`castRay`的传入的射线和射线起点了。

墙壁与天花板交界是有对面墙的颜色，是使用材质的着色点也写错了😤，应该使用当前着色点`p_inter`的材质而不是`inter_middle`。

```c++
Vector3f wi = normalize(p_inter.m->sample(-ray.direction, p_inter.normal)), l_indir;
inter_middle = intersect(Ray(p_inter.coords, wi));
if (inter_middle.happened && !inter_middle.obj->hasEmit())
    l_indir = castRay(Ray(inter_middle.coords, wi), depth + 1) * inter_middle.m->eval(wi, -ray.direction, p_inter.normal) *
              dotProduct(wi, p_inter.normal) / (p_inter.m->pdf(wi, -ray.direction, p_inter.normal) * RussianRoulette);
// 递归的射线起点应该使用p_inter.coords，后面的材质也应该用p_inter.m
```

### 高的方块阴影靠墙侧黑色噪点很多

测试了一下，主要是直接光照的影响比较大，传参的时候要按照作业说明中的传才能得到作业参考中的结果。

> 虽然可能作业的参考不一定对。

```c++
float Material::pdf(const Vector3f &wi, const Vector3f &wo, const Vector3f &N);
Vector3f Material::eval(const Vector3f &wi, const Vector3f &wo, const Vector3f &N);
p_inter.m->eval(-ray.direction, ws, p_inter.normal);
// 使用p_inter.m->eval(ws, -ray.direction, p_inter.normal)就会出现很多黑色噪点
// 调整间接光照中的传入参数影响没有那么大。
```

### 微表面白色噪点多

[Games101,作业7（微表面模型）_微表面模型c++实现-CSDN博客](https://blog.csdn.net/weixin_44518102/article/details/122698851)

尝试修改一下sphere的求交判断阈值，有可能是精度的问题。roughness过小，如=0会导致球体直接变黑，暂时不知道为什么。

[大作业重要性采样及多重重要性采样实现中遇到的问题 – 计算机图形学与混合现实在线平台](https://games-cn.org/forums/topic/dazuoyezhongyaoxingcaiyangjiduozhongzhongyaoxingcaiyangshixianzhongyudaodewenti/)

重要性采样后续在慢慢写吧😫

## Assignment8

window只用把freetype的源码下载make编译获得freetype.a即可。

### make工程的时候出现 undefined reference to `__imp_glewInit'

编译器按动态库（DLL）声明函数如`glewInit`，就会生成 `__imp_glewInit` 符号，如果实际链接的是静态编译的库`.a`文件，那么库中函数无 `__imp_` 前缀，在链接时会因为符号不匹配导致链接失败。

```sh
../CGL/src/libCGL.a(viewer.cpp.obj):viewer.cpp:(.text+0x520): undefined reference to `__imp_glewInit'
```

针对src文件夹下的文件出现链接不匹配，因此向src文件夹下的cmakelist.txt中加入以下语句

```cmake
#-------------------------------------------------------------------------------
# Building static library (always)
#-------------------------------------------------------------------------------
add_library(CGL STATIC ${CGL_SOURCE} ${CGL_HEADER})
# -------------------------------------------------
# 添加内容 针对GLEW库指定使用静态链接
target_compile_definitions(CGL PRIVATE GLEW_STATIC)
# --------------------------------------------------
target_link_libraries(
  CGL
  ${GLEW_LIBRARIES}
  ${GLFW_LIBRARIES}
  ${OPENGL_LIBRARIES}
  ${FREETYPE_LIBRARIES}
)
```

### 欧拉法结果发散

[关于作业8的一些问题解答 – 计算机图形学与混合现实在线平台](https://games-cn.org/forums/topic/guanyuzuoye8deyixiewentijieda/)

不加阻尼的话，属于正常现象。

## 框架环境搭建

### vs的问题

[Visual Studio Community 2017/2019 下载方法 - 知乎](https://zhuanlan.zhihu.com/p/740245143)

[VS 工程下的.vs文件过大问题解决_.vs文件夹-CSDN博客](https://blog.csdn.net/qq_21611223/article/details/148256705)

### 运行环境

[GAMES102课程框架环境配置_games102环境配置-CSDN博客](https://blog.csdn.net/weixin_51017692/article/details/131860505)

最好使用虚拟机按照课程要求的工具以及版本安装。否则高于vs2019的版本会报一些奇怪的问题，比如标准库文件缺失，语法不对等等。直接使用vs2019的话，基本cmake构建完成就可以直接运行了。

> 原本想着使用mingw和vscode弄弄应该也可以，但是后面版本和编译器不一致的问题踩了一堆坑，搞了我差不多一天，最后还是老老实实用虚拟机装vs2019，装好就能把Ubpa引擎正常运行起来。以后配环境还是得匹配好才行，框架、标准、库这些版本一改就很容易报莫名其妙的问题。

asset.zip最好实现下载好，然后按照上面的配置教程修改cmakelist指向该文件，避免git的下载网络问题。

方法一会下载所需要的源码文件，构建好后整个解决方案项目下会带有所有源码文件夹。方法二构建好后只会带有作业的工程，记得把upba的bin添加到环境变量，否则在运行时会出现找不到dll的错误。

如果出现d3d10warp.dll缺失(这个dll可能在sysWOW64或者system32中)，可参考[VS“无法查找或打开PDB文件”是怎么回事？如何解决 - C语言中文网](https://c.biancheng.net/view/474.html)，[Visual Studio 调试器中的符号/PDB 文件 | Microsoft Learn](https://learn.microsoft.com/zh-cn/visualstudio/debugger/specify-symbol-dot-pdb-and-source-files-in-the-visual-studio-debugger?view=vs-2022)

> 整个问题很玄学，有时重启之后有没有了。

按照方法二构建的工程项目中，ctrl+点击查看某些变量或命名空间时出现"未找到符号定义"的提示，原因是有些头文件没有包含添加到vs的查找路径中，通过项目属性中的"附加包含目录"中添加对应的头文件路径即可。按方法一构建的话应该已经自动添加好了。

更换环境变量后最好重启一次vs，不然可能会检索不到所设置的链接库，比如hw8踩的坑就很难绷。

### eigen库相关

#### 在vs使用

对于eigen库的使用，vs中最好按项目hw1，hw3等分别添加头文件，尽量不要直接添加到整个工程的解决方案中。或者直接往每个hw下的cmakelist中添加引用头文件的语句。

项目属性 -> 配置属性 -> C/C++ -> 常规 -> 附加包含目录。尽管在vc++中的包含目录中也能引入头文件，不过还是建议在C/C++中设置，在c/c++中设置类似于编译器使用`-I`命令，可能更符合命令行的链接习惯。参考

1. [Visual Studio中C++的包含目录、附加包含目录和库目录和附加库目录的区别_c++附加库目录-CSDN博客](https://blog.csdn.net/qq_27825451/article/details/103035258)
2. [Visual Station 2022的头文件包含目录设置的区别_vs2022项目目录-CSDN博客](https://blog.csdn.net/yellow_hill/article/details/126399072)
3. [“VC++ 目录”属性页 | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/build/reference/vcpp-directories-property-page?view=msvc-170)

#### 语法坑

##### 混淆

1. [Eigen: Aliasing](https://libeigen.gitlab.io/eigen/docs-nightly/group__TopicAliasing.html)
2. [Eigen: Lazy Evaluation and Aliasing](https://libeigen.gitlab.io/eigen/docs-nightly/TopicLazyEvaluation.html)

当执行`m = m.transpose()`时，Eigen的懒惰求值机制会导致问题：Eigen不会自动创建临时副本来存储转置结果，而是尝试在原地进行操作。这意味着在计算转置的过程中，Eigen会同时读取和写入同一个矩阵对象，导致数据在完全读取之前就被覆盖。

如果对于eigen默认识别的aliasing，如数乘，矩阵乘法，这种两侧都出现的变量时，Eigen 会将乘积计算到一个临时矩阵中，计算完成后才将其赋值给该变量。如果调用`m.noalias() = m * mat_b`，`noalias()`会明示没有混叠，此时eigen将会直接对原矩阵进行操作，会得到错误的结果。具体说明可以参考链接。

#### 函数传参

1. [Eigen: Eigen::Ref< PlainObjectType, Options, StrideType > Class Template Reference](https://libeigen.gitlab.io/eigen/docs-nightly/classEigen_1_1Ref.html)
2. [eigen3 - Eigen::Ref 使用指南：生命周期、存储顺序问题与 AnyOptions 解决方案](https://runebook.dev/zh/docs/eigen3/classeigen_1_1ref)
3. [请问矩阵运算库Eigen中的Ref怎么使用？ - 知乎](https://www.zhihu.com/question/365470994)
4. [Eigen使用矩阵作为函数参数 - 学习时间轴 - 博客园](https://www.cnblogs.com/yabin/p/6506484.html)

使用eigen的ref对象可以较快的向函数传递矩阵，减少大量的元素拷贝。就算不使用ref类型的引用，ref对象的拷贝量也很少。而且要使用ref的引用还得在传参前先定义一个ref对象才能传入，不想定义就需要使用const，这又会导致函数内部不能修改传入的对象。所以根据官网的推荐的两种用法写就行。

```c++
// eigen的官网示例
// in-out argument:
void foo1(Ref<VectorXf> x);
 
// read-only const argument:
void foo2(const Ref<const VectorXf>& x);
```

尽管使用`MatrixXf&`也能传递引用，但是灵活性(比如想传入矩阵的某部分)不如ref，因此官方推荐使用ref进行矩阵的参数传递。

```c++
// 如果使用了ref的引用，记得确认需不需要要加上const
Eigen::VectorXf Solver(Eigen::Ref<const Eigen::MatrixXf>& A); // 接受的是Ref类型的左值引用，不能直接传入MatrixXf的对象，调用前必须保证A是Ref类型，如下：
Eigen::Ref<const Eigen::MatrixXf> rA = A;
Solver(rA);

// 如果直接传入MatrixXf的对象，Ref会使用该对象创建一个临时的Ref类型的右值，所以必须要加const。
Eigen::VectorXf Solver(const Eigen::Ref<const Eigen::MatrixXf>& A); // A可以是MatrixXf类型
// 一般不会在传参时额外定义一个ref对象再传入函数,因此const ref更常用一些,后面的&可用可不用
```

> [c++ - Correct usage of the Eigen::Ref<> class - Stack Overflow](https://stackoverflow.com/questions/21132538/correct-usage-of-the-eigenref-class)
>
> 我有想过能不能写成`const Ref<const VectorXf&>& x`😂，不过ref是类模板，其模板参数`PlainObjectType`必须是具体类型，而不能接受引用类型。
>
> 我想尝试`const Ref<const VectorXf>`中哪个`const`回导致对象不可更改，结果`Ref<const VectorXf>`和`const Ref<VectorXf>`都会使得对象不能更改。当我想像测试常量指针和指针常量一样尝试对ref的const进行对比时出现了下面的情况：
>
> ```c++
> Eigen::MatrixXf m(2, 2);
> m << 1, 2, 2, 3;
> const Eigen::Ref<Eigen::MatrixXf> a = m;
> Eigen::MatrixXf b = Eigen::MatrixXf::Random(2, 2);
> Eigen::Ref<Eigen::MatrixXf> c = b;
> a = c;	// vscode编辑器会直接警告没有匹配的运算符，应该是eigen为了确保const被不能赋值
> // no operator "=" matches these operands
> 
> // ------------------
> 
> Eigen::MatrixXf m(2, 2);
> m << 1, 2, 2, 3;
> Eigen::Ref<const Eigen::MatrixXf> a = m;
> // 但是这个就如果好像不允许const类型之间赋值
> Eigen::MatrixXf b = Eigen::MatrixXf::Random(2, 2);
> Eigen::Ref<const Eigen::MatrixXf> c = b;
> 
> a = c; // 这里vscode编辑器能够根据库匹配到能够使用=，
> // inline Eigen::Ref<const Eigen::MatrixXf> &Eigen::Ref<const Eigen::MatrixXf>::operator=(const Eigen::Ref<const Eigen::MatrixXf> &)
> // 但是编译调用时会报错，这是因为eigen在设计时不希望只读视图类型Ref<const T>被重新绑定，因此显示删除了只读视图的赋值函数导致报错。
> error: use of deleted function 'Eigen::MapBase<Eigen::Ref<const Eigen::Matrix<float, -1, -1> >, 0>& Eigen::MapBase<Eigen::Ref<const Eigen::Matrix<float, -1, -1> >, 0>::operator=(const Eigen::MapBase<Eigen::Ref<const Eigen::Matrix<float, -1, -1> >, 0>&)'
> ```
>
> 这种为了禁止赋值而显示删除函数的做法由c++的关键字delete实现，为了禁止一些不希望被用户使用的显示函数调用，使用delete来显式地禁止编译器自动生成某些函数。简单示例可参考，[C++11中=delete的巧妙用法_= delete-CSDN博客](https://blog.csdn.net/whahu1989/article/details/90648536)

## 框架说明

可以参考课程视频P8中对框架的介绍。

- 框架的入口函数是`WinMain()`，在windows GUI程序中和`main`等价。

- 该框架的UI使用ImGui，每帧调用函数update时都会重新绘制。


框架运行时可能会报比较多的警告，但是只要没有错误就问题不大😏。

教程中针对修改CanvasData.h的表述。手动把project文件夹中hw的src文件夹中删除即可，vs中点击本地调试后会自动生成，不用额外重新通过cmake或cmake-gui进行构建。框架作者使用了 USRefl 来实现静态反射功能，能够提供自动生成描述文件 .inl 的工具。

> 我们使用了 [USRefl](https://github.com/Ubpa/USRefl) 来实现静态反射功能，提供了一个自动生成描述文件 [CanvasData_AutoRefl.inl](https://github.com/Ubpa/GAMES102/blob/main/homeworks/project/src/hw1/Components/details/CanvasData_AutoRefl.inl) 的工具。
>
> 当你修改 CanvasData.h 后，你需要将 CanvasData_AutoRefl.inl 删除，以便使 CMake 自动生成新的描述文件。

## hw 1 

拉格朗日插值是用暴力循环写的，有些坐标是整数的计算优化可以参考[[多项式学习笔记\] 拉格朗日插值 - DengStar - 博客园](https://www.cnblogs.com/dengstar/p/18786311/note-poly-lagrange)，工程上可能也用不到，针对算法竞赛的，了解了解即可。

高斯函数插值表达式中，如果带有常数项或者待定的非高斯函数多项式，会导致方程个数少于未知数，这时需要自行添加约束条件，可以直接指定偏移，也可以根据需要指定，比如所有系数和为0等。方差可以取大一些效果会比较好。

[Eigen: Solving linear least squares systems](https://libeigen.gitlab.io/eigen/docs-nightly/group__LeastSquares.html)，[数值计算库Eigen：求解线性最小二乘方程组 - 知乎](https://zhuanlan.zhihu.com/p/536011746)。由于屏幕上点击x的值一般都比较大，使用的多项式次数过高，在计算系数矩阵的时候很可能会直接超出浮点数的运算范围。使用库的求解器会在左侧出现平滑的现象，还不知道怎么解决。

中途虚拟机内存不够迁移项目的时候会遇到静态反射的exe文件执行不了的问题，还没有解决。拿使用预编译的方法构建的项目如果删除了inl后也会执行错误，不知道是不是权限的问题。虽然不修改inl文件也能正常运行。

> 好家伙，在C盘下构建的就可以生成，麻了😤。20251220
>
> 哦吼，并不是权限的问题，是CanvasData.h 的注释不能带有中文，我靠我服了，真难蚌啊😂。20251221

## hw2

1. [机器学习算法推导&手写实现06——RBF网络](https://www.zhihu.com/tardis/zm/art/159199147?source_id=1005)
2. [【机器学习】径向基（RBF）神经网络的tensorflow实现_tensorflow rbf神经网络-CSDN博客](https://blog.csdn.net/zqx951102/article/details/89503542)
3. [从零开始几何处理：RBF函数 - 知乎](https://zhuanlan.zhihu.com/p/413596878)

BP神经网络可以有多个隐含层，但是RBF一般只使用一个隐含层。rbf训练的隐藏层中高斯激活函数的参数，而不是像BP神经网络中训练层之间传递的权重矩阵。[RBF（径向基）神经网络 - 禅在心中 - 博客园](https://www.cnblogs.com/pinking/p/9349695.html)

AI给的代码中，会使用最小二乘法求解权重，但是均值一般就是随机选取，方差则根据所选均值之间的距离或其他度量方法生成。当然也可以使用梯度下降法对所有参数进行更新，不过AI说这种方法更适合大规模数据。

## hw3

如果把参数变量被限制在$[0,1]$内，使用高斯函数对参数向量进行逼近，方差需要小一点，方差过大会导致分量的拟合过于平坦，最后的效果会很不好。

最好根据输入点列的取值重新分配用于绘图的参数变量，或者做归一化，否则如果直接把参数变量的范围固定，随着点列的增多，间距会越来越小，拟合分量的时候容易精度缺失。

感觉写出来的bug挺多的。尤其两个点的时候用高斯基函数有时会得不到直线，可能应该是需要调参（调方差）或者需要加入一些幂基函数的约束。如果参数化没有问题的话，问题可能主要是函数拟合，函数拟合的性能会影响绘制的结果，方差应该作为一个自适应参数调整，比如用神经网络。

games101的assignment4中的贝塞尔曲线，当时使用的就是均匀参数化。

## hw4

对于函数插值（不是曲线），三次样条插值的函数插值点要求有序(参考matlab的生成c++代码说明，输入 `x` 必须严格递增)，也就是输入的向量中不能出现后面的点的x坐标比前面小的情况，这和之前hw1中的拉格朗日等全局插值的方法不同。其实最好就是函数内部做一次排序。

G1连续的要求对应位置加上即可修改切线时候的限制即可，我没有去做。目前只实现了G0连续，也就是对应PPT中角部顶点的情况，拖动和编辑某顶点的信息只会影响该顶点所连的两段曲线。

理想的效果应该是像ppt一样可以单独编辑每个顶点是否要满足G0还是G1连续。PPT中的直线点和角部顶点的说明

- 角部顶点：该顶点一侧的曲线变形不会影响另一侧。
- 直线顶点：现象比较奇怪，如果相邻顶点发生移动导致相连曲线形变，那么该点的导数与形变侧导数一致；如果相邻顶点切线修改导致相连曲线形变，那么该点的导数与原来保持一致。

我的拖动切线变化不明显的原因感觉应该是坐标相对于参数变量变换太大了(x，y的取值划一下就是几百，而参数被限定在0~1，算出来导数就会很大，而且曲线会很长，导数影响的部位看不见)，可能需要合理调整参数变量的取值范围才行。

## hw5

比较简单，对照ppt写即可。

## hw6

### 框架使用

hemesh为半边数据结构，用于计算；mesh用于渲染。mesh to HEMesh是将当前状态的Mesh(正在显示的mesh)转为HEMesh，节点的排列顺序和Mesh的排列顺序一致(一般是012按照逆时针排列)。操作的时候注意状态的变化，不然可能会导致模型混乱。

编辑器中的denoiseData组件是给赋予的模型添加噪声，如果场景中有该模型，首先需要将mesh转为hemesh，在点击添加噪声，最后需要将hemesh转为mesh后才可在视图中可见。

更换模型时，recover mesh储存的网格可能是上一个不同模型的，这会在hemesh to mesh时把现在的模型覆盖，导致模型混乱，这个时候就重新运行吧。

提供的框架中，如果要直接在inspector界面中修改组件的位置或其他可输入的地方，需要连点三次才能修改。

### 半边数据结构

根据`HEMeshX.h`中`Txxx`的实现，每个顶点、边、多边形的访问都以一条半边开始，即顶点、边、多边形结构体的成员变量都是绑定在某一条半边上，通过这条半边和访问函数实现对顶点、边、多边形的邻居等属性的访问。而半边数据`THalfEdge.h`中，只储存了边的起点。

### 极小曲面的局部法

如果想通过节点的邻边按序获取三角形的顶点，最好通过`outhalfedge()`获得的外向半边结构邻边，`AdjEdges()`获得的邻边需要再次访问`HalfEdge()`才能获得半边，而且这个半边的起点还不一定是原节点，处理节点顺序的时候很不方便。

bunny_head的网格太小了，很容易导致除零错误，而且迭代更新的lambda值也要设置得很小才行。反正我写出来的效果很不好。

参考作业中也有提到：应使lambda尽量小且多步迭代，这样才能保证算法的稳定性。否则出现一个噪点后可能就会导致后续更新发生剧烈异动。

## hw7

全局方法中需要使用eigen的稀疏矩阵求解器。[Eigen: Solving Sparse Linear Systems](https://libeigen.gitlab.io/eigen/docs-nightly/group__TopicSparseSystems.html)，内置的直接求解器中，`SimplicialLLT`和`SimplicialLDLT`要求稀疏矩阵正定，但是构造的拉普拉斯矩阵其实不一定能满足，误差可能会比较大，结果会爆炸，所以建议使用LU、QR分解或者迭代求解器。

可以通过框架提供的函数`Boundaries()`获取网格中洞的入口列表，再通过每个入口的`NextLoop()`获取该洞的边界环。

参数化方法暂时只用来均匀参数化，把边界等距映射到圆上。

## hw8

原理不难，参考[几何建模与处理-实现平面点集CVT的Lloyd算法老铁，点个在看呗 欢迎关注公众号:sumsmile /专注图形渲染、A - 掘金](https://juejin.cn/post/7160337746014240776)，照搬了[GAMES102/homeworks/HW8/cvt.py at 77e7d2f9379a06df6f72e640d581371ec90d8195 · g1n0st/GAMES102](https://github.com/g1n0st/GAMES102/blob/77e7d2f9379a06df6f72e640d581371ec90d8195/homeworks/HW8/cvt.py)的代码，用的是python库直接生成voronoi图。如果自己写划分为算法的话可能就比较复杂，涉及计算几何的内容[「计算几何」Fortune 算法生成维诺图（Voronoi Diagram） | Caleb's Space](https://caleb.ink/posts/92/index.html)。或者可以用cgal中的现成函数。

### cgal配置

[CGAL安装(C++)并配置vs-CSDN博客](https://blog.csdn.net/qq_54042725/article/details/136763062)

[VS编译报错 LNK2019 _imp_gmpq_init_无法解析的外部符号 imp gmpq-CSDN博客](https://blog.csdn.net/qq_38014822/article/details/110524106)

更换cgal版本5.0.2后能够运行，但是lloyd算法会出现"堆已损坏"的错误。难蚌，重启vs之后又没事了，这框架😂。

重启vs之后用新版本也可以运行了，感觉是作业框架的问题。另外换环境变量后最好也重启一下，不然可能会检索不到所设置的链接库。

boost和cgal都可以使用最新版，vs2019使用boost-msvc-14.2的版本。6.1.1的cgal在github上下载文件名带`library`的压缩包即可(不包含例程)以及`GMP and MPFR libraries, for Windows 64bits`，前者用于cgal的头文件但没包含gmp计算库，后者是运行cgal需要的运算库，包含动态链接库与头文件。

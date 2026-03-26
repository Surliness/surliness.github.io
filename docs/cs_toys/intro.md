纯靠三分钟热度🤪，记一记流水账

# 3DGS小试

202602

wsl + colmap + gsplat + supersplat

[效果](video/3dgs.mp4 ':include :type=iframe width=90% height=500px')

顶部的效果不是很好，可能是身高不够拍不到上面的图片导致

---

# 扩散曲线渲染器

202505-06

学校选修课数字图像处理课程设计。本来想找个三维重建的项目，最后打算仿照室友毕设用Qt做的画布程序改了一个，加了点图形几何和渲染的东西。虽然算法基本都是用了现成，不过也算是温习把games101 102学的内容温习了一遍。

改掉了原框架使用的Qt库和visual studio(破烂笔记本实在承受不起这厚重的ide)，额外补了渲染管线的配置，功能全部更换为开源库实现。借用了一部分中科大图形学课程的开源作业框架。感谢图像重建曲线的Potrace算法开源实现[zhuethanca/DiffusionCurves](https://github.com/zhuethanca/DiffusionCurves)

工具：

- 构建工具：cmake
- 图形和计算库：opengl、glfw、eigen
- 其他：imgui、sigslots、opencv

功能：

- 交互式GUI，实时参数调整和可视化
- 三次贝塞尔曲线和三次B样条曲线的构造绘制以及渲染功能
- 求解扩散方程，使用多级帧缓冲和着色器实现颜色点的扩散效果
- 集成图像矢量化处理流程，由高斯模糊、边缘检测的图像预处理+位图转换矢量曲线的Potrace算法实现

### 效果

[效果](video/dcr.mp4 ':include :type=iframe width=90% height=500px')

AI什么时候能根据运行出来的图修bug就好了😫


### 后续

待解决问题：在自己的AMD笔记本上用核显跑会出现atio6axx.dll出错而导致的异常退出，nvidia的显卡则不会报错。

待改进：向量化可以使用多线程优化一下速度，以免影响交互

--------

# 魔改斗地主
202403-07

寒假和老同学打牌意犹未尽，网上没找到和老家类似玩法的现成品，想自己试着写一个用于几个好朋友联机玩玩😏。不过后面做出一版后有点烂尾了，焦虑毕设、科研没心思完善。
### 参考资料
玩法借鉴：[广西♠A♠3玩法（欢迎推广） - 知乎](https://zhuanlan.zhihu.com/p/613609802)

基本框架和素材来源：[【Unity】Unity2017多人网络斗地主开发实战_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1db411F7tW/?spm_id_from=333.337.search-card.all.click)。

头一次做这种游戏开发，只能先跟着写了一遍，主要视频的版本有些老旧了，C#和Unity的有些函数甚至已经因为安全问题或者版本更新被遗弃了。处理起来确实踩了不少坑。

### 初步效果
[效果](video/a3game.mp4 ':include :type=iframe width=90% height=500px')


有些数值溢出的边界问题，不过出牌、贡牌、胜负判定逻辑啥的基本可用。这个教程想要的功能也没写全，不过学个棋牌小游戏大致的开发流程也够了。

### 后续

待改进：玩家排队候场机制






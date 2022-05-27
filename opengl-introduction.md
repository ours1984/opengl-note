---
title: 'OpenGL的前世今生'
date: '2021/7/23 22:46:25'
tags: [CG,OpenGL]
abbrlink: opengl-introduction
gitrep: opengl-note
---

本文介绍OpenGL的由来,它的设计思想,以及OpenGL的上下文和窗口的概念.最后介绍其他的图形api和显示引擎,分析比较了他们的异同.

博客链接<https://blog.ours1984.top/posts/opengl-introduction>
<!--more-->

## 简介

OpenGL（英语：Open Graphics Library，译名：开放图形库或者“开放式图形库”）是用于渲染2D、3D矢量图形的跨语言、跨平台的应用程序编程接口（API）。这个接口由近350个不同的函数调用组成，用来从简单的图形比特绘制复杂的三维景象。而另一种程序接口系统是仅用于Microsoft Windows上的Direct3D。OpenGL常用于CAD、虚拟实境、科学可视化程序和电子游戏开发。

OpenGL的高效实现（利用了图形加速硬件）存在于Windows，部分UNIX平台和Mac OS。这些实现一般由显示设备厂商提供，而且非常依赖于该厂商提供的硬件。开放源代码库Mesa是一个纯基于软件的图形API，它的代码兼容于OpenGL。但是，由于许可证的原因，它只声称是一个“非常相似”的API。

OpenGL规范由1992年成立的OpenGL架构评审委员会（ARB）维护。ARB由一些对创建一个统一的、普遍可用的API特别感兴趣的公司组成。根据OpenGL官方网站，2002年6月的ARB投票成员包括3Dlabs、Apple Computer、ATI Technologies、Dell Computer、Evans & Sutherland、Hewlett-Packard、IBM、Intel、Matrox、NVIDIA、SGI和Sun Microsystems，Microsoft曾是创立成员之一，但已于2003年3月退出。
<!--more-->

## 设计

OpenGL规范描述了绘制2D和3D图形的抽象API。尽管这些API可以完全通过软件实现，但它是为大部分或者全部使用硬件加速而设计的。

OpenGL的API定义了若干可被客户端程序调用的函数，以及一些具名整型常量（例如，常量GL_TEXTURE_2D对应的十进制整数为3553）。虽然这些函数的定义表面上类似于C编程语言，但它们是语言独立的。因此，OpenGL有许多语言绑定，值得一提的包括：JavaScript绑定的WebGL（基于OpenGL ES 2.0在Web浏览器中的进行3D渲染的API）；C绑定的WGL、GLX和CGL；iOS提供的C绑定；Android提供的Java和C绑定。

OpenGL不仅语言无关，而且平台无关。规范只字未提获得和管理OpenGL上下文相关的内容，而是将这些作为细节交给底层的窗口系统。出于同样的原因，OpenGL纯粹专注于渲染，而不提供输入、音频以及窗口相关的API。

OpenGL是一个不断进化的API。新版OpenGL规范会定期由Khronos Group发布，新版本通过扩展API来支持各种新功能。每个版本的细节由Khronos Group的成员一致决定，包括显卡厂商、操作系统设计人员以及类似Mozilla和谷歌的一般性技术公司。

除了核心API要求的功能之外，GPU供应商可以通过扩展的形式提供额外功能。扩展可能会引入新功能和新常量，并且可能放松或取消现有的OpenGL函数的限制。然后一个扩展就分成两部分发布：包含扩展函数原型的头文件和作为厂商的设备驱动。供应商使用扩展公开自定义的API而无需获得其他供应商或Khronos Group的支持，这大大增加了OpenGL的灵活性。OpenGL Registry负责所有扩展的收集和定义。

每个扩展都与一个简短的标识符关系，该标识符基于开发公司的名称。例如，英伟达（nVidia）的标识符是NV。如果多个供应商同意使用相同的API来实现相同的功能，那么就用EXT标志符。这种情况更进一步，Khronos Group的架构评审委员（Architecture Review Board，ARB）正式批准该扩展，那么这就被称为一个“标准扩展”，标识符使用ARB。第一个ARB扩展是GL_ARB_multitexture。

OpenGL每个新版本中引入的功能，特别是ARB和EXT类型的扩展，通常由数个被广泛实现的扩展功能组合而成。

## 书籍

所有版本的OpenGL规范文档都被公开的寄存在Khronos那里。如果你想深入到OpenGL的细节（只关心函数功能的描述而不是函数的实现），这是个很好的选择。如果你想知道每个函数具体的运作方式，这个规范也是一个很棒的参考。

有兴趣的读者可以去阅读[规范文档](<https://www.khronos.org/registry/OpenGL/specs/gl/>),这里面有四个方面的文档

glspec说明了OpenGL api的规范

glu对辅助函数提供了说明,包括mipmapping、矩阵操作、多边形镶嵌、二次曲面、NURBS 和错误处理

glx说明了X 窗口系统的 OpenGL 扩展

glslangspec说明了glsl语言规范

### 红宝书

Dave Shreiner, Graham Sellers, John M. Kessenich and Bill M. Licea-Kane. 2013.**OpenGL Programming Guide**: The Official Guide to Learning OpenGL, Version 4.3（8th Edition）. Addison-Wesley Professional.ISBN 978-0321773036.

相当于查询手册了

### 橙宝书

Randi J. Rost, Bill M. Licea-Kane, Dan Ginsburg, John M. Kessenich, Barthold Lichtenbelt, Hugh Malan and Mike Weiblen. 2009.**OpenGL Shading Language** (3rd Edition). Addison-Wesley Professional.ISBN 978-0321637635

### 蓝宝书

OpenGL SuperBible -- 官方推荐书籍，比较系统全面，适合入门学习

### wiki

主要查询手段

<https://www.khronos.org/opengl/wiki>

> This Wiki is a collection of information about OpenGL, as well as frequently asked questions about OpenGL and its API. Tutorials are also welcome and may be hosted on this Wiki.
Contributions on this wiki are open to the public, you only need to create a user account. We ask that you please respect the content on this wiki and post only information that is relevant to OpenGL.

## 核心功能包

OpenGL api通过核心程序库来实现其功能

opengl32.lib, OpenGL核心库,装完驱动就有的

glu32.lib , opengl 实用库,装完驱动也有,43个函数，以glu开头

glaux.lib , oepngl 辅助库,装完驱动也有.31个函数，以aux 开头

opengl 实用库通过调用核心库gl的函数，为开发者提供相对简单的用法，实现一些较为复杂的操作。包括纹理映射、坐标变换、多边形分化、绘制一些如椭球、圆柱、茶壶等简单多边形实体部分函数象核心函数一样在任何OpenGL平台都可以应用。

### 高级功能集成

OpenGL被设计为只有输出的，所以它只提供渲染功能。核心API没有窗口系统、音频、打印、键盘/鼠标或其他输入设备的概念。虽然这一开始看起来像是一种限制，但它允许进行渲染的代码完全独立于他运行的操作系统，允许跨平台开发。

然而，有些集成于原生窗口系统的东西需要允许和宿主系统交互。这通过下列操作系统附加API实现：

GLX- X11: linux系统集成

WGL: MicrosoftWindows 系统集成

### 渲染功能扩展包

GLU 最后一次更新规格要求是在 1998 年，对已弃用的 OpenGL 特性有依赖,然后就没更新过了.`<GL/gl.h>`这个头文件，里面只有OpenGL 1.1版本中所规定的内容，而没有OpenGL 1.2及其以后版本。但 OpenGL现在都发展到4.6以上了，要使用这些OpenGL的高级特性，就必须下载最新的扩展，另外，不同的显卡公司，也会发布一些只有自家显卡才支持的扩展函数，你要想用这数涵数，不得不去寻找最新的glext.h

往往使用 **[GLEW](http://glew.sourceforge.net/index.html)** 库,有了GLEW扩展库，你就再也不用为找不到函数的接口而烦恼，因为GLEW能自动识 别你的平台所支持的全部OpenGL高级扩展涵数。也就是说，只要包含一个glew.h头文件，你就能使用gl,glu,glext,wgl,glx的全部函数。GLEW支持目前流行的各种操作系统（including Windows, Linux, Mac OS X, FreeBSD, Irix, and Solaris）

它可以在程序的运行期判断当前硬件是否支持相关的扩展，防止程序崩溃甚至造成硬件损坏。

## OpenGL上下文

### Context是什么

OpenGL自身是一个巨大的状态机(State Machine)：一系列的变量描述OpenGL此刻应当如何运行。OpenGL的状态通常被称为OpenGL上下文(Context)。我们通常使用如下途径去更改OpenGL状态：设置选项，操作缓冲。最后，我们使用当前OpenGL上下文来渲染。

假设当我们想告诉OpenGL去画线段而不是三角形的时候，我们通过改变一些上下文变量来改变OpenGL状态，从而告诉OpenGL如何去绘图。一旦我们改变了OpenGL的状态为绘制线段，下一个绘制命令就会画出线段而不是三角形。

当使用OpenGL的时候，我们会遇到一些状态设置函数(State-changing Function)，这类函数将会改变上下文。以及状态使用函数(State-using Function)，这类函数会根据当前OpenGL的状态执行一些操作。只要你记住OpenGL本质上是个大状态机，就能更容易理解它的大部分特性。

### Context和窗口创建

在我们画出出色的效果之前，首先要做的就是创建一个OpenGL上下文(Context)和一个用于显示的窗口。然而，这些操作在每个系统上都是不一样的，OpenGL有目的地从这些操作抽象(Abstract)出去。我们不得不自己处理创建窗口，定义OpenGL上下文以及处理用户输入

在OpenGL中一个对象是指一些选项的集合，它代表OpenGL状态的一个子集。比如，我们可以用一个对象来代表绘图窗口的设置，之后我们就可以设置它的大小、支持的颜色位数等等。可以把对象看做一个C风格的结构体(Struct)：

```c
// OpenGL的状态
struct OpenGL_Context {
    ...
    object* object_Window_Target;
    ...     
};

// 创建对象
unsigned int objectId = 0;
glGenObject(1, &objectId);
// 绑定对象至上下文
glBindObject(GL_WINDOW_TARGET, objectId);
// 设置当前绑定到 GL_WINDOW_TARGET 的对象的一些选项
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_WIDTH, 800);
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_HEIGHT, 600);
// 将上下文对象设回默认
glBindObject(GL_WINDOW_TARGET, 0);
```

这一小段代码展现了你以后使用OpenGL时常见的工作流。我们首先创建一个对象，然后用一个id保存它的引用（实际数据被储存在后台）。然后我们将对象绑定至上下文的目标位置（例子中窗口对象目标的位置被定义成GL_WINDOW_TARGET）。接下来我们设置窗口的选项。最后我们将目标位置的对象id设回0，解绑这个对象。设置的选项将被保存在objectId所引用的对象中，一旦我们重新绑定这个对象到GL_WINDOW_TARGET位置，这些选项就会重新生效。

使用对象的一个好处是在程序中，我们不止可以定义一个对象，并设置它们的选项，每个对象都可以是不同的设置。在我们执行一个使用OpenGL状态的操作的时候，只需要绑定含有需要的设置的对象即可。比如说我们有一些作为3D模型数据（一栋房子或一个人物）的容器对象，在我们想绘制其中任何一个模型的时候，只需绑定一个包含对应模型数据的对象就可以了（当然，我们需要先创建并设置对象的选项）。拥有数个这样的对象允许我们指定多个模型，在想画其中任何一个的时候，直接将对应的对象绑定上去，便不需要再重复设置选项了。

## 窗口扩展包

OpenGL Context 创建过程相当复杂，在不同的操作系统上也需要不同的做法。因此很多游戏开发和用户界面库都提供了自动创建 OpenGL 上下文的功能，其中包括SDL、Allegro、SFML、FLTK、Qt等。也有一些库是专门用来创建 OpenGL 窗口的，其中最早的便是GLUT，后被freeglut取代，比较新的也有GLFW可以使用。
这些是对操作系统的封装,使得我们把操作系统透明化,只需要知道有context和窗口就行

### 最基本的窗口包

以下包可以用来创建并管理 OpenGL 窗口，也可以管理输入，但几乎没有除此以外的其它功能

最适合用来学习OpenGL了,因为它对OpenGL不管但又必须要有的东西提供了最简单的实现

**[GLFW](https://www.glfw.org/download.html)** ——跨平台窗口和键盘、鼠标、手柄处理；偏向游戏

**freeglut** ——跨平台窗口和键盘、鼠标处理；API 是 GLUT API 的超集，同时也比 GLUT 更新、更稳定

GLUT ——早期的窗口处理库，已不再维护

### 多媒体包

支持创建 OpenGL 窗口的还有一些“多媒体库”，同时还支持输入、声音等类似游戏的程序所需要的功能：

Allegro 5 ——跨平台多媒体库，提供针对游戏开发的 C API

SDL ——跨平台多媒体库，提供 C API

SFML ——跨平台多媒体库，提供 C++ API；同时也提供 C#、Java、Haskell、Go 等语言的绑定

### 窗口进化包

**FLTK** ——小型的跨平台 C++ 窗口组件库

**Qt** ——跨平台 C++ 窗口组件库，提供了许多 OpenGL 辅助对象，抽象掉了桌面版OpenGL 与 OpenGL ES 之间的区别

**wxWidgets** —— 是一个开源的跨平台的C++构架库（framework），它可以提供GUI（图形用户界面）和其它工具。2.x版本支持所有版本的Windows、带GTK+或Motif的Unix和MacOS。一个支持OS/2的版本正在开发中。

## 其他图形api介绍

### Metal

苹果操作gpu的api

- Metal 是一个和 OpenGL ES 类似的面向底层的图形编程接口，可以直接操作GPU；
- 支持iOS和OS X，提供图形渲染和通用计算能力。
- Metal 的最大好处就是与 OpenGL ES 相比显著降低了消耗。
- 在 OpenGL 中无论创建缓冲区还是纹理，OpenGL 都会复制一份以防止 GPU 在使用它们的时候被意外访问。出于安全的原因, 复制类似纹理和缓冲区这样的大的资源, 是非常耗时的操作。
- Metal 并不复制资源。开发者需要负责在 CPU 和 GPU 之间同步访问。

### Direct3D

Direct3D是DirectX套装的一部分，只能用于windows相关的平台，用C++实现，并以COM的方式提供API

![20220525012432](https://pic.ours1984.top/img/20220525012432.png!shuiyin)

### Vulkan

- Vulkan是一个跨平台的2D和3D绘图应用程序接口（API），由Khronos组织在2015年游戏开发者大会（GDC）上发表。旨在替代OpenGL，提高图形性能。

- 更简单的显示驱动（ Vulkan提供了能直接控制和访问底层GPU的显示驱动抽象层，显示驱动只是对硬件薄薄的封装，这样能够显著提升操作GPU硬件的效率和性能。之前OpenGL的驱动层对开发者隐藏的很多细节，现在都暴露出来。Vulkan甚至不包含运行期的错误检查层。驱动层干的事情少了，隐藏的bug也就少了）

- 支持多线程（Vulkan不再使用OpenGL的状态机设计，内部也不保存全局状态变量。显示资源完全由应用层负责管理，包括内存管理、线程管理、多线程绘制命令产生、渲染队列提交等。应用程序可以充分利用CPU的多核多线程的计算资源，减少CPU等待，降低延迟。为GPU端并行渲染提供了可能）Vulkan 的API更贴近硬件调用，所以Vulkan 绘制效率要好于OpenGL。

- 预编译shader(驱动层不提供前端shader编译器，只支持标准可移植中间表示二进制代码（SPIR-V）。即提高了执行Shaders的效率又增加了将来着色语言的灵活性。所以目前的GLSL/HLSL可以直接通过工具转换为SPIR-V，在Vulkan中使用。这样就可以使用离线的shader编译。)

### Api功能对比

![20220525012541](https://pic.ours1984.top/img/20220525012541.png!shuiyin)

| OpenGL版本       | D3D版本                | OS版本  |
|----------------|----------------------|-------|
| OpenGL 2.1     | Direct3D 9.0（SM2.0）  | WinXP |
| OpenGL 3.0~3.1 | Direct3D 9.0c（SM3.0） | WinXP |
| OpenGL 3.2~3.3 | Direct3D 10.0（SM4.0） | Vista |
| OpenGL 4.0~4.6 | Direct3D 11.0（SM5.0） | Win7  |

| 3D API    | 最后只支持固定管线的版本 | 第一个支持可编程管线版本 | 第一个只支持可编程管线版本 |
|-----------|--------------|--------------|---------------|
| OpenGL    | 1.5          | 2            | 3.1           |
| OpenGL ES | 1.1          | 2            | 2             |
| WebGL     | 无            | 1            | 1            |
| Direct3D  | 7            | 8            | 10            |

## 市面上的显示引擎

- osg是一种场景图的方法，每个opengl相关的函数都是一个节点， 适合用物理仿真。集成多相机，多视图，粒子系统，各种回调函数，只支持opengl, 扩展起来很方便
- ogre用于游戏，支持Direct3D和OpenGL,支持Windows，Linux和Mac OSX。插件式粒子系统，场景管理
- VTK 采用面向对象思想，基于 OpenGL 开发出目标函数库。它将将一些常用的算法封装为类的形式，用户在开发过程中可以直接调用其函数库进行开发，而不必纠结函数内部具体的实现过程。
- Cocos2d-x是一个基于MIT协议的开源框架，用于构建游戏、应用程序和其他图形界面交互应用。基于OpenGL ES
- Qt3d, Unity3d，Unreal Engine

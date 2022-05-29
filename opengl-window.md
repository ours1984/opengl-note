---
title: '从零开始画窗口'
date: '2021/7/25 22:46:25'
tags: [CG,OpenGL]
abbrlink: opengl-window
gitrep: opengl-note
---
经过上一节的学习介绍,我们已经知道OpenGL是个啥了,在正式开始画三角形之前,还有一只拦路虎,也就是图像渲染的载体--窗口.

虽然我们的各种窗口库可以分简单地弄出窗口来,但我们必须搞明白这个东西,不能云里雾里,否则今后的学习你肯定会出现各种问题

<!--more-->

## GLFW安装

GLFW可以从它官方网站的[下载页](http://www.glfw.org/download.html)上获取。GLFW已提供为Visual Studio（2012到2019都有）预编译好的二进制版本和相应的头文件，但是为了完整性我们将从编译源代码开始。所以我们需要下载源代码包。

也可以直接从笔者github上拉取源码<https://github.com/xiaoqide/glfw.git>.笔者[本项目代码仓库](https://github.com/xiaoqide/opengl-note-code.git)直接配置了glfw的子仓库,可以一并拉下来

从源代码编译库可以保证生成的库完全适合你的操作系统和CPU的，而预编译的二进制文件则并非总是提供（有时候，即便提供了预编译的二进制文件，也可能不适用于您的系统）。开放源代码所产生问题在于：并不是每个人都用相同的IDE或者构建系统来搞开发，因而提供的项目/解决方案文件可能和一些人的IDE不兼容。所以人们必须使用给定的.c/.cpp和.h/.hpp文件来自己建立项目/解决方案，这是一项很枯燥的工作。但因此也诞生了一个叫做CMake的工具。

vscode可以安装cmake扩展,我们的ide使用vscode,编译器使用mingw环境的gcc.尽可能和linux保持一致,一切都在vscode里完成.不要用vs studio.使用linux系统的你们就偷着乐吧!不用忍受瘟到死了

然后如此这般,我们就把glfw编译好了.我们把编译好的头文件和库文件拿出备用.放到项目根目录下的include和lib下面

## GLAD安装

也不叫安装.还记得我们之前说的GLEW吗,glad提供了一个网页在线生成OpenGL api地址查找函数服务,我们可以动态配置自己的OpenGL版本以及扩展需求

因为OpenGL只是一个标准/规范，具体的实现是由驱动开发商针对特定显卡 实现的。由于OpenGL驱动版本众多，它大多数函数的位置都无法在编译时确定下来，需要在运行时查询。所以任务就落在了开发者身上，开发者需要在运行时获取函数地址并将其保存在一个函数指针中供以后使用。取得地址的方法因平台而异，在Windows上会是类似这样：

```c
// 定义函数原型
typedef void (*GL_GENBUFFERS) (GLsizei, GLuint*);
// 找到正确的函数并赋值给函数指针
GL_GENBUFFERS glGenBuffers  = (GL_GENBUFFERS)wglGetProcAddress("glGenBuffers");
// 现在函数可以被正常调用了
GLuint buffer;
glGenBuffers(1, &buffer);
```

你可以看到代码非常复杂，而且很繁琐，我们需要对每个可能使用的函数都要重复这个过程。幸运的是，有些库能简化此过程，其中GLAD是目前最新，也是最流行的库。

打开GLAD的[在线服务](http://glad.dav1d.de/)，将语言(Language)设置为C/C++，在API选项中，选择3.3以上的OpenGL(gl)版本（我们将使用3.3版本，但更新的版本也能用）。之后将模式(Profile)设置为Core，并且保证选中了生成加载器(Generate a loader)选项。现在可以先（暂时）忽略扩展(Extensions)中的内容。都选择完之后，点击生成(Generate)按钮来生成库文件。

GLAD现在应该提供给你了一个zip压缩文件，包含两个头文件目录，和一个glad.c文件。

将两个头文件目录（glad和KHR）复制到放到项目根目录下的include，glad.c放到lib下.我们不做库文件了,直接源码编译

## CMakeList.txt编写

我们要写一个cmakelist,把我们编译好的glfw和glad以及OpenGL依赖配置好.同时引入googletest做单元测试

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(opengl-note VERSION 0.1.0)

set(CMAKE_CXX_STANDARD 11)

set(nodeOutDir ${PROJECT_SOURCE_DIR}/bin)
set(gladIncDir ${PROJECT_SOURCE_DIR}/glad)
set(gtestIncDir ${PROJECT_SOURCE_DIR}/googletest/googletest/include)
set(glfwIncDir ${PROJECT_SOURCE_DIR}/glfw/include)

include_directories(${gladIncDir})
include_directories(${glfwIncDir})
include_directories(${gtestIncDir})
link_directories(${nodeOutDir})

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${nodeOutDir})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${nodeOutDir})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${nodeOutDir})
set(LIBRARY_OUTPUT_PATH ${nodeOutDir})

add_subdirectory(googletest)
add_subdirectory(glfw)
add_subdirectory(glad)

set (dependLib
    glfw
    glad
    gtest
    -lopengl32
)
add_subdirectory(src)

```

我们采用add_subdirectory的方式生成子项目glad,glfw,gtest,同时把他们作为项目依赖,附加给src里面的单元测试

在子目录里分别写好CMakeList.txt,就可以编译了

关于cmake的使用,尤其是子项目的编写,可以参考博文[CMake从入门到入门](https://blog.ours1984.top/posts/cmake)

## 你好窗口

首先给出源码文件,参考[hellow-window](https://github.com/xiaoqide/opengl-note-code/blob/main/src/test_window.cpp).这里知识代码说明,单独运行不起来的.你需要下载整个项目才能跑起来

首先，我们在main函数中调用glfwInit函数来初始化GLFW，然后我们可以使用glfwWindowHint函数来配置GLFW。

glfwWindowHint函数的第一个参数代表选项的名称，我们可以从很多以GLFW_开头的枚举值中选择；第二个参数接受一个整型，用来设置这个选项的值。如果你现在编译你的cpp文件会得到大量的 undefined reference (未定义的引用)错误，也就是说你并未顺利地链接GLFW库。

接下来我们创建一个窗口对象，这个窗口对象存放了所有和窗口相关的数据，而且会被GLFW的其他函数频繁地用到。glfwCreateWindow函数需要窗口的宽和高作为它的前两个参数。第三个参数表示这个窗口的名称（标题），这里我们使用"LearnOpenGL"，当然你也可以使用你喜欢的名称。最后两个参数我们暂时忽略。这个函数将会返回一个GLFWwindow对象，我们会在其它的GLFW操作中使用到。创建完窗口我们就可以通知GLFW将我们窗口的上下文设置为当前线程的主上下文了。

然后我们把窗口设置为当前context,再调用gladLoadGLLoader初始化OpenGL api地址.注意,需要有当前context才能初始化成功

## 你好视口

在我们开始渲染之前还有一件重要的事情要做，我们必须告诉OpenGL渲染窗口的尺寸大小，即视口(Viewport)，这样OpenGL才只能知道怎样根据窗口大小显示数据和坐标。我们可以通过调用glViewport函数来设置窗口的维度(Dimension)。

我们实际上也可以将视口的维度设置为比GLFW的维度小，这样子之后所有的OpenGL渲染将会在一个更小的窗口中显示，这样子的话我们也可以将一些其它元素显示在OpenGL视口之外。

然而，当用户改变窗口的大小的时候，视口也应该被调整。我们可以对窗口注册一个回调函数(Callback Function)，它会在每次窗口大小被调整的时候被调用。

当窗口被第一次显示的时候framebuffer_size_callback也会被调用。对于视网膜(Retina)显示屏，width和height都会明显比原输入值更高一点。

我们还可以将我们的函数注册到其它很多的回调函数中。比如说，我们可以创建一个回调函数来处理手柄输入变化，处理错误消息等。我们会在创建窗口之后，渲染循环初始化之前注册这些回调函数。

## 你好循环

我们可不希望只绘制一个图像之后我们的应用程序就立即退出并关闭窗口。我们希望程序在我们主动关闭它之前不断绘制图像并能够接受用户输入。因此，我们需要在程序中添加一个while循环，我们可以把它称之为渲染循环(Render Loop)，它能在我们让GLFW退出前一直保持运行。我们用一个死循环来不停地渲染.退出条件是窗口被用户关闭.

glfwPollEvents函数检查有没有触发什么事件（比如键盘输入、鼠标移动等）、更新窗口状态，并调用对应的回调函数（可以通过回调方法手动设置）。

glfwSwapBuffers函数会交换颜色缓冲（它是一个储存着GLFW窗口每一个像素颜色值的大缓冲），它在这一迭代中被用来绘制，并且将会作为输出显示在屏幕上。

{% note info %}
**双缓冲**(Double Buffer)

应用程序使用单缓冲绘图时可能会存在图像闪烁的问题。 这是因为生成的图像不是一下子被绘制出来的，而是按照从左到右，由上而下逐像素地绘制而成的。最终图像不是在瞬间显示给用户，而是通过一步一步生成的，这会导致渲染的结果很不真实。为了规避这些问题，我们应用双缓冲渲染窗口应用程序。前缓冲保存着最终输出的图像，它会在屏幕上显示；而所有的的渲染指令都会在后缓冲上绘制。当所有的渲染指令执行完毕后，我们交换(Swap)前缓冲和后缓冲，这样图像就立即呈显出来，之前提到的不真实感就消除了。

{% endnote %}

## 你好输入

我们同样也希望能够在GLFW中实现一些输入控制，这可以通过使用GLFW的几个输入函数来完成。我们将会使用GLFW的glfwGetKey函数，它需要一个窗口以及一个按键作为输入。这个函数将会返回这个按键是否正在被按下。我们将创建一个processInput函数来让所有的输入代码保持整洁。

这里我们检查用户是否按下了返回键(Esc)（如果没有按下，glfwGetKey将会返回GLFW_RELEASE。如果用户的确按下了返回键，我们将通过glfwSetwindowShouldClose使用把WindowShouldClose属性设置为 true的方法关闭GLFW。下一次while循环的条件检测将会失败，程序将会关闭。

我们在渲染循环的每一个迭代中调用processInput.再做自己的渲染操作.现在你啥都不会,所以我们调用api来清屏吧,设置屏幕颜色

如果你看见了一个非常无聊的窗口，那么就对了！如果你没得到正确的结果，或者你不知道怎么把所有东西放到一起，可以去github上下载配套项目<https://github.com/xiaoqide/opengl-note-code>

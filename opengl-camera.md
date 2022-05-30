---
title: '从零开始学FPS摄像机'
date: '2021/8/9 22:46:25'
tags: [CG,OpenGL,camera]
abbrlink: opengl-camera
gitrep: opengl-note
mathjax: true
---
从现在开始,我们就要绘制大量三角形了.这里我们引入一种更高效的条带绘制方式.

然后才是本节核心camara.这里不会对其中的原理和细节做推导,直接给出结论.

如果感兴趣,可以看这篇[博文](https://blog.ours1984.top/posts/games101-04)

这里着重阐述camara在OpenGL中的FPS应用场景,怎么使用它来构造我们的camera.

<!--more-->

## 七种基本图元

之前我们学过`glDrawArrays(GL_TRIANGLES, 0, 3)`这是画三角形的基本方式,其实OpenGL里有以下其中图元

|       图元        |            描述             |
|-----------------|---------------------------|
|    GL_POINTS    |      每个顶点在屏幕上都是单独的点       |
|    GL_LINES     |        每两个点组成一条线段         |
|  GL_LINE_STRIP  |  从第一个点一次经过每一个后续顶点而绘制的线条   |
|  GL_LINE_LOOP   |把GL_LINE_STRIP最后一个点和第一个点相连 |
|  GL_TRIANGLES   |        每三个点组成一个三角形        |
|GL_TRIANGLE_STRIP| 从第三个点开始,每一个点和前两个点组成一个三角形  |
| GL_TRIANGLE_FAN | 以第一个点为中心,和后面每两个相邻点组成一个三角形 |

![20220527231249](https://pic.ours1984.top/img/20220527231249.png)

![20220527231002](https://pic.ours1984.top/img/20220527231002.png)

### 面剔除

由于我们看一个立方体不管从任何方向看最多看到3个平面，那么我们就没有必要去绘制剩下的三个平面，并且这样做将为性能带来超过50%的优化

OpenGL里面开启面剔除使用glEnable(GL_CULL_FACE),然后需要设置剔除背面glCullFace(GL_BACK).

也许你还需要通过glFrontFace(GL_CW)设置正面的环绕方式，GL_CW顺时针，GL_CCW逆时针

![20220527231031](https://pic.ours1984.top/img/20220527231031.png)

如果你开启了面剔除，那么你就得保证看得见的面为逆时针，看不见的面为顺时针.三角化的时候需要去考虑顶点顺序

![20220527231530](https://pic.ours1984.top/img/20220527231530.png)

## 坐标系统

OpenGL希望在每次顶点着色器运行后，我们可见的所有顶点都为标准化设备坐标(Normalized Device Coordinate, NDC)。也就是说，每个顶点的x，y，z坐标都应该在-1.0到1.0之间，超出这个坐标范围的顶点都将不可见。我们通常会自己设定一个坐标的范围，之后再在顶点着色器中将这些坐标变换为标准化设备坐标。然后将这些标准化设备坐标传入光栅器(Rasterizer)，将它们变换为屏幕上的二维坐标或像素。

将坐标变换为标准化设备坐标，接着再转化为屏幕坐标的过程通常是分步进行的，也就是类似于流水线那样子。在流水线中，物体的顶点在最终转化为屏幕坐标之前还会被变换到多个坐标系统(Coordinate System)。将物体的坐标变换到几个过渡坐标系(Intermediate Coordinate System)的优点在于，在这些特定的坐标系统中，一些操作或运算更加方便和容易，这一点很快就会变得很明显。对我们来说比较重要的总共有5个不同的坐标系统：

- 局部空间(Local Space，或者称为物体空间(Object Space))
- 世界空间(World Space)
- 观察空间(View Space，或者称为视觉空间(Eye Space))
- 裁剪空间(Clip Space)
- 屏幕空间(Screen Space)
这就是一个顶点在最终被转化为片段之前需要经历的所有不同状态。

### 概述

为了将坐标从一个坐标系变换到另一个坐标系，我们需要用到几个变换矩阵，最重要的几个分别是模型(Model)、观察(View)、投影(Projection)三个矩阵。我们的顶点坐标起始于局部空间(Local Space)，在这里它称为局部坐标(Local Coordinate)，它在之后会变为世界坐标(World Coordinate)，观察坐标(View Coordinate)，裁剪坐标(Clip Coordinate)，并最后以屏幕坐标(Screen Coordinate)的形式结束。下面的这张图展示了整个流程以及各个变换过程做了什么：

![20220527231625](https://pic.ours1984.top/img/20220527231625.png)

- 局部坐标是对象相对于局部原点的坐标，也是物体起始的坐标。
- 下一步是将局部坐标变换为世界空间坐标，世界空间坐标是处于一个更大的空间范围的。这些坐标相对于世界的全局原点，它们会和其它物体一起相对于世界的原点进行摆放。
- 接下来我们将世界坐标变换为观察空间坐标，使得每个坐标都是从摄像机或者说观察者的角度进行观察的。
坐标到达观察空间之后，我们需要将其投影到裁剪坐标。裁剪坐标会被处理至-1.0到1.0的范围内，并判断哪些顶点将会出现在屏幕上。
- 最后，我们将裁剪坐标变换为屏幕坐标，我们将使用一个叫做视口变换(Viewport Transform)的过程。视口变换将位于-1.0到1.0范围的坐标变换到由glViewport函数所定义的坐标范围内。最后变换出来的坐标将会送到光栅器，将其转化为片段。

当需要对物体进行修改的时候，在局部空间中来操作会更说得通；如果要对一个物体做出一个相对于其它物体位置的操作时，在世界坐标系中来做这个才更说得通，等等。如果我们愿意，我们也可以定义一个直接从局部空间变换到裁剪空间的变换矩阵，但那样会失去很多灵活性。

### 局部空间

局部空间是指物体所在的坐标空间，即对象最开始所在的地方。想象你在一个建模软件（比如说Blender）中创建了一个立方体。你创建的立方体的原点有可能位于(0, 0, 0)，即便它有可能最后在程序中处于完全不同的位置。甚至有可能你创建的所有模型都以(0, 0, 0)为初始位置（译注：然而它们会最终出现在世界的不同位置）。所以，你的模型的所有顶点都是在局部空间中：它们相对于你的物体来说都是局部的。

### 世界空间

世界空间中的坐标正如其名：是指顶点相对于（游戏）世界的坐标。如果你希望将物体分散在世界上摆放（特别是非常真实的那样），这就是你希望物体变换到的空间。物体的坐标将会从局部变换到世界空间；该变换是由模型矩阵(Model Matrix)实现的。

模型矩阵是一种变换矩阵，它能通过对物体进行位移、缩放、旋转来将它置于它本应该在的位置或朝向。你可以将它想像为变换一个房子，你需要先将它缩小（它在局部空间中太大了），并将其位移至郊区的一个小镇，然后在y轴上往左旋转一点以搭配附近的房子。你也可以把上一节将箱子到处摆放在场景中用的那个矩阵大致看作一个模型矩阵；我们将箱子的局部坐标变换到场景/世界中的不同位置。

### 观察空间

观察空间经常被人们称之OpenGL的摄像机(Camera)（所以有时也称为摄像机空间(Camera Space)或视觉空间(Eye Space)）。观察空间是将世界空间坐标转化为用户视野前方的坐标而产生的结果。因此观察空间就是从摄像机的视角所观察到的空间。而这通常是由一系列的位移和旋转的组合来完成，平移/旋转场景从而使得特定的对象被变换到摄像机的前方。这些组合在一起的变换通常存储在一个观察矩阵(View Matrix)里，它被用来将世界坐标变换到观察空间。

![20220527185318](https://pic.ours1984.top/img/20220527185318.png)

$$
\quad M_{view}=R_{view}T_{view}=
\left[\begin{array}{cccc}
x_{\hat{g} \times \hat{t}} & y_{\hat{g} \times \hat{t}} & z_{\hat{g} \times \hat{t}} & -x_{e} \\
x_{t} & y_{t} & z_{t} & -y_{e} \\
x_{-g} & y_{-g} & z_{-g} & -z_{e} \\
0 & 0 & 0 & 1
\end{array}\right]
$$

### 裁剪空间

在一个顶点着色器运行的最后，OpenGL期望所有的坐标都能落在一个特定的范围内，且任何在这个范围之外的点都应该被裁剪掉(Clipped)。被裁剪掉的坐标就会被忽略，所以剩下的坐标就将变为屏幕上可见的片段。这也就是裁剪空间(Clip Space)名字的由来。

因为将所有可见的坐标都指定在-1.0到1.0的范围内不是很直观，所以我们会指定自己的坐标集(Coordinate Set)并将它变换回标准化设备坐标系，就像OpenGL期望的那样。

为了将顶点坐标从观察变换到裁剪空间，我们需要定义一个投影矩阵(Projection Matrix)，它指定了一个范围的坐标，比如在每个维度上的-1000到1000。投影矩阵接着会将在这个指定的范围内的坐标变换为标准化设备坐标的范围(-1.0, 1.0)。所有在范围外的坐标不会被映射到在-1.0到1.0的范围之间，所以会被裁剪掉。在上面这个投影矩阵所指定的范围内，坐标(1250, 500, 750)将是不可见的，这是由于它的x坐标超出了范围，它被转化为一个大于1.0的标准化设备坐标，所以被裁剪掉了。

由投影矩阵创建的**观察箱**(Viewing Box)被称为**平截头体**(Frustum)，每个出现在平截头体范围内的坐标都会最终出现在用户的屏幕上。将特定范围内的坐标转化到标准化设备坐标系的过程（而且它很容易被映射到2D观察空间坐标）被称之为投影(Projection)，因为使用投影矩阵能将3D坐标投影(Project)到很容易映射到2D的标准化设备坐标系中。

一旦所有顶点被变换到裁剪空间，最终的操作——透视除法(Perspective Division)将会执行，在这个过程中我们将位置向量的x，y，z分量分别除以向量的齐次w分量；透视除法是将4D裁剪空间坐标变换为3D标准化设备坐标的过程。这一步会在每一个顶点着色器运行的最后被自动执行。

## 谁是真正MVP

### 使用GLM

```c
#include <glm/vec3.hpp> // glm::vec3
#include <glm/vec4.hpp> // glm::vec4
#include <glm/mat4x4.hpp> // glm::mat4
#include <glm/ext/matrix_transform.hpp> // glm::translate, glm::rotate, glm::scale
#include <glm/ext/matrix_clip_space.hpp> // glm::perspective
#include <glm/ext/scalar_constants.hpp> // glm::pi

glm::mat4 camera(float Translate, glm::vec2 const& Rotate)
{
    glm::mat4 Projection = glm::perspective(glm::pi<float>() * 0.25f, 4.0f / 3.0f, 0.1f, 100.f);
    glm::mat4 View = glm::translate(glm::mat4(1.0f), glm::vec3(0.0f, 0.0f, -Translate));
    View = glm::rotate(View, Rotate.y, glm::vec3(-1.0f, 0.0f, 0.0f));
    View = glm::rotate(View, Rotate.x, glm::vec3(0.0f, 1.0f, 0.0f));
    glm::mat4 Model = glm::scale(glm::mat4(1.0f), glm::vec3(0.5f));
    return Projection * View * Model;
}
```

GLM是OpenGL Mathematics的缩写，它是一个只有头文件的库，也就是说我们只需包含对应的头文件就行了，不用链接和编译。GLM可以在它们的[网站](https://glm.g-truc.net/0.9.8/index.html)上下载。把头文件的根目录复制到你的includes文件夹，然后你就可以使用这个库了。

### 投影变换

将观察坐标变换为裁剪坐标的投影矩阵可以为两种不同的形式，每种形式都定义了不同的平截头体。我们可以选择创建一个正射投影矩阵(Orthographic Projection Matrix)或一个透视投影矩阵(Perspective Projection Matrix)。

![20220528205019](https://pic.ours1984.top/img/20220528205019.png)

你可以看到，使用透视投影的话，远处的顶点看起来比较小，而在正射投影中每个顶点距离观察者的距离都是一样的。

![20220528010114](https://pic.ours1984.top/img/20220528010114.png!shuiyin)

平截头体（视口坐标）中的3D点被映射到立方体（NDC）; 从[l，r]到[-1,1]的x坐标范围，从[b，t]到[-1,1]的y坐标和来自[-n，-f]的z坐标 到[-1,1]。

请注意，视口坐标是在右手坐标系中定义的，但NDC使用左手坐标系。 也就是说，原点处的相机沿着视口空间中的-Z轴看，但它在NDC中沿着+ Z轴看。

### 正射投影

正射投影矩阵定义了一个类似立方体的平截头箱，它定义了一个裁剪空间，在这空间之外的顶点都会被裁剪掉。创建一个正射投影矩阵需要指定可见平截头体的宽、高和长度。在使用正射投影矩阵变换至裁剪空间之后处于这个平截头体内的所有坐标将不会被裁剪掉。

$$
M_{\text {ortho }}=\left[\begin{array}{cccc}
\frac{2}{r-l} & 0 & 0 & 0 \\
0 & \frac{2}{t-b} & 0 & 0 \\
0 & 0 & \frac{2}{n-f} & 0 \\
0 & 0 & 0 & 1
\end{array}\right]
\left[\begin{array}{cccc}
1 & 0 & 0 & -\frac{r+l}{2} \\
0 & 1 & 0 & -\frac{t+b}{2} \\
0 & 0 & 1 & -\frac{n+f}{2} \\
0 & 0 & 0 & 1
\end{array}\right]=
\left[\begin{array}{cccc}
\frac{2}{r-l} & 0 & 0 & -\frac{r+l}{r-l} \\
0 & \frac{2}{t-b} & 0 & -\frac{t+b}{t-b} \\
0 & 0 & \frac{2}{n-f} & \frac{n+f}{n-f} \\
0 & 0 & 0 & 1
\end{array}\right]
$$

要创建一个正射投影矩阵，我们可以使用GLM的内置函数`glm::ortho`

```c
glm::ortho(0.0f, 800.0f, 0.0f, 600.0f, 0.1f, 100.0f);
```

前两个参数指定了平截头体的左右坐标，第三和第四参数指定了平截头体的底部和顶部。通过这四个参数我们定义了近平面和远平面的大小，然后第五和第六个参数则定义了近平面和远平面的距离。这个投影矩阵会将处于这些x，y，z值范围内的坐标变换为标准化设备坐标。

![20220528223810](https://pic.ours1984.top/img/20220528223810.png)

正射投影矩阵直接将坐标映射到2D平面中，即你的屏幕，但实际上一个直接的投影矩阵会产生不真实的结果，因为这个投影没有将透视(Perspective)考虑进去。所以我们需要透视投影矩阵来解决这个问题。

### 透视投影

投影矩阵将给定的平截头体范围映射到裁剪空间，除此之外还修改了每个顶点坐标的w值，从而使得离观察者越远的顶点坐标w分量越大。被变换到裁剪空间的坐标都会在-w到w的范围之间（任何大于这个范围的坐标都会被裁剪掉）。OpenGL要求所有可见的坐标都落在-1.0到1.0范围内，作为顶点着色器最后的输出，因此，一旦坐标在裁剪空间内之后，透视除法就会被应用到裁剪空间坐标上：

![20220528211033](https://pic.ours1984.top/img/20220528211033.png!shuiyin)

$$
M_{\text {persp }}=
\left[\begin{array}{cccc}
\frac{2}{r-l} & 0 & 0 & -\frac{r+l}{r-l} \\
0 & \frac{2}{t-b} & 0 & -\frac{t+b}{t-b} \\
0 & 0 & -\frac{2}{f-n} & -\frac{n+f}{f-n} \\
0 & 0 & 0 & 1
\end{array}\right]
\left[\begin{array}{cccc}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & n+f & -nf \\
0 & 0 & -1 & 0
\end{array}\right]=
\left[\begin{array}{cccc}
\frac{2n}{r-l} & 0 & \frac{r+l}{r-l} & 0 \\
0 & \frac{2n}{t-b} & \frac{t+b}{t-b} & 0 \\
0 & 0 & \frac{n+f}{n-f} & \frac{2nf}{f-n} \\
0 & 0 & 1 & 0
\end{array}\right]
$$

顶点坐标的每个分量都会除以它的w分量，距离观察者越远顶点坐标就会越小。这是也是w分量非常重要的另一个原因，它能够帮助我们进行透视投影。最后的结果坐标就是处于标准化设备空间中的。

要创建一个透视投影矩阵，我们可以使用GLM的内置函数`glm::perspective`

```c
glm::mat4 proj = glm::perspective(glm::radians(45.0f), (float)width/(float)height, 0.1f, 100.0f);
```

它的第一个参数定义了fov的值，它表示的是视野(Field of View)，并且设置了观察空间的大小。如果想要一个真实的观察效果，它的值通常设置为45.0f，但想要一个末日风格的结果你可以将其设置一个更大的值。第二个参数设置了宽高比，由视口的宽除以高所得。第三和第四个参数设置了平截头体的近和远平面。我们通常设置近距离为0.1f，而远距离设为100.0f。所有在近平面和远平面内且处于平截头体内的顶点都会被渲染。

### 组合MVP

我们为上述的每一个步骤都创建了一个变换矩阵：模型矩阵、观察矩阵和投影矩阵。一个顶点坐标将会根据以下过程被变换到裁剪坐标

$$
V_{clip}=M_{projection}\cdot M_{view}\cdot M_{model}\cdot V_{local}
$$

注意每个矩阵被运算的顺序是相反的(记住我们需要从右往左乘上每个矩阵)。最后的顶点应该被赋予顶点着色器中的gl_Position且OpenGL将会自动进行透视划分和裁剪。

{% note info %}
顶点着色器的输出需要所有的顶点都在裁剪空间内，而这是我们的转换矩阵所做的。OpenGL然后在裁剪空间中执行透视划分从而将它们转换到标准化设备坐标。OpenGL会使用glViewPort内部的参数来将标准化设备坐标映射到屏幕坐标，每个坐标都关联了一个屏幕上的点(在我们的例子中屏幕是800 *600)。这个过程称为视口转换。
{% endnote %}

```c
glm::mat4 model(1.0f);
model = glm::rotate(model, glm::radians(-55.0f), glm::vec3(1.0f, 0.0f, 0.0f));

glm::mat4 view(1.0f);
// 注意，我们将矩阵向我们要进行移动场景的反方向移动。
view = glm::translate(view, glm::vec3(0.0f, 0.0f, -3.0f));

glm::mat4 projection(1.0f);
projection = glm::perspective(glm::radians(45.0f), screenWidth / screenHeight, 0.1f, 100.0f);
```

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main()
{
    // 注意乘法要从右向左读
    gl_Position = projection * view * model * vec4(aPos, 1.0);
    
}
```

如果你在哪儿卡住了，可以到[这里]((https://github.com/xiaoqide/opengl-note-code/blob/main/src/test_mvp.cpp))查看源码。

## 摄像机

![20220529001614](https://pic.ours1984.top/img/20220529001614.png)

看图,已知现在是在世界坐标,如图所示摆放摄像机(位置已知,反方向已知,上方向已知),要得到摄像机视角下的坐标,我们首先需要把摄像机移动到移动,也就需要一个反位置;然后需要把反方向转换到z轴,把上方向转换到y轴,把右方向转换到x轴,写成矩阵就是以下方式,

$$
\operatorname{Look} A t=\left[\begin{array}{cccc}
R_{x} & R_{y} & R_{z} & 0 \\
U_{x} & U_{y} & U_{z} & 0 \\
D_{x} & D_{y} & D_{z} & 0 \\
0 & 0 & 0 & 1
\end{array}\right] *\left[\begin{array}{cccc}
1 & 0 & 0 & -P_{x} \\
0 & 1 & 0 & -P_{y} \\
0 & 0 & 1 & -P_{z} \\
0 & 0 & 0 & 1
\end{array}\right]
$$

R 是右向量，U是上向量，D是方向向量P是摄像机位置向量。注意，位置向量是相反的，因为我们最终希望把世界平移到与我们自身移动的相反方向。使用这个LookAt矩阵坐标观察矩阵可以很高效地把所有世界坐标变换为观察坐标LookAt矩阵就像它的名字表达的那样：它会创建一个观察矩阵looks at(看着)一个给定目标。

幸运的是，GLM已经提供了这些支持。我们要做的只是定义一个摄像机位置，一个目标位置(计算反方向的钟)和一个表示上向量的世界空间中的向量(我们使用上向量计算右向量)。接着GLM就会创建一个LookAt矩阵，我们可以把它当作我们的观察矩阵：

```C
GLfloat radius = 10.0f;
GLfloat camX = sin(glfwGetTime()) * radius;
GLfloat camZ = cos(glfwGetTime()) * radius;
glm::mat4 view(1.0f);
view = glm::lookAt(glm::vec3(camX, 0.0, camZ), glm::vec3(0.0, 0.0, 0.0), glm::vec3(0.0, 1.0, 0.0)); 
```

### 移动相机

```c
void key_callback(GLFWwindow* window, int key, int scancode, int action, int mode)
{
    ...
    GLfloat cameraSpeed = 0.05f;
    if(key == GLFW_KEY_W)
        cameraPos += cameraSpeed * cameraFront;
    if(key == GLFW_KEY_S)
        cameraPos -= cameraSpeed * cameraFront;
    if(key == GLFW_KEY_A)
        cameraPos -= glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
    if(key == GLFW_KEY_D)
        cameraPos += glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;  
}

view = glm::lookAt(cameraPos, cameraPos + cameraFront, cameraUp);
```

这是因为大多数事件输入系统一次只能处理一个键盘输入，它们的函数只有当我们激活了一个按键时才被调用。大多数GUI系统都是这样的，它对摄像机来说用并不合理。我们可以用一些小技巧解决这个问题。

这个技巧是只在回调函数中跟踪哪个键被按下/释放。在游戏循环中我们读取这些值，检查那个按键被激活了，然后做出相应反应。我们只储存哪个键被按下/释放的状态信息，在游戏循环中对状态做出反应，我们来创建一个布尔数组代表按下/释放的键：

```c
bool keys[1024];
if(action == GLFW_PRESS)
    keys[key] = true;
else if(action == GLFW_RELEASE)
    keys[key] = false;

void do_movement()
{
  // 摄像机控制
  GLfloat cameraSpeed = 0.01f;
  if(keys[GLFW_KEY_W])
    cameraPos += cameraSpeed * cameraFront;
  if(keys[GLFW_KEY_S])
    cameraPos -= cameraSpeed * cameraFront;
  if(keys[GLFW_KEY_A])
    cameraPos -= glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
  if(keys[GLFW_KEY_D])
    cameraPos += glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
}
```

### 移动速度

实际情况下根据处理器的能力不同，有的人在同一段时间内会比其他人绘制更多帧。也就是调用了更多次do_movement函数。每个人的运动速度就都不同了。当你要发布的你应用的时候，你必须确保在所有硬件上移动速度都一样。

图形和游戏应用通常有回跟踪一个deltaTime变量，它储存渲染上一帧所用的时间。我们把所有速度都去乘以deltaTime值。当我们的deltaTime变大时意味着上一帧渲染花了更多时间，所以这一帧使用这个更大的deltaTime的值乘以速度，会获得更高的速度，这样就与上一帧平衡了。使用这种方法时，无论你的机器快还是慢，摄像机的速度都会保持一致，这样每个用户的体验就都一样了。

```c
GLfloat currentFrame = glfwGetTime();
deltaTime = currentFrame - lastFrame;
lastFrame = currentFrame; 

GLfloat cameraSpeed = 5.0f * deltaTime;
```

### 视角移动

为了能够改变方向，我们必须根据鼠标的输入改变cameraFront向量。然而，根据鼠标旋转改变方向向量有点复杂，需要更多的三角学知识。

欧拉角(Euler Angle)是可以表示3D空间中任何旋转的3个值，由莱昂哈德·欧拉(Leonhard Euler)在18世纪提出。一共有3种欧拉角：俯仰角(Pitch)、偏航角(Yaw)和滚转角(Roll)，下面的图片展示了它们的含义：

![20220529005934](https://pic.ours1984.top/img/20220529005934.png)

俯仰角是描述我们如何往上或往下看的角，可以在第一张图中看到。第二张图展示了偏航角，偏航角表示我们往左和往右看的程度。滚转角代表我们如何翻滚摄像机，通常在太空飞船的摄像机中使用。每个欧拉角都有一个值来表示，把三个角结合起来我们就能够计算3D空间中任何的旋转向量了。

对于我们的摄像机系统来说，我们只关心俯仰角和偏航角，所以我们不会讨论滚转角。给定一个俯仰角和偏航角，我们可以把它们转换为一个代表新的方向向量的3D向量。通过旋转pitch、yaw、roll角度，并将相机移到指定位置eye，那么对应的视变换矩阵为$\text { view }=\left(T * R_{\text {roll }} * R_{\text {yaw }} * R_{\text {pitch }}\right)^{-1}$

![20220529185550](https://pic.ours1984.top/img/20220529185550.png)

对相机进行pitch和yaw角度的旋转后，我们需要重新计算相机的forward向量，以及side向量用来完成相机的前后左右移动。这两个向量都是在世界坐标系下给定的。我们可以计算出相机的坐标系下的点经过旋转后，在世界坐标系下的值。计算得到:

$$
\begin{aligned}
R &=R_{\text {yaw }} R_{\text {pitch }} \\
&=\left[\begin{array}{cccc}
\cos _{\text {yaw }} & 0 & \sin _{\text {yaw }} & 0 \\
0 & 1 & 0 & 0 \\
-\sin _{\text {yaw }} & 0 & \cos _{\text {yaw }} & 0 \\
0 & 0 & 0 & 1
\end{array}\right] *\left[\begin{array}{cccc}
1 & 0 & 0 & 0 \\
0 & \cos _{\text {pitch }} & -\sin _{\text {pitch }} & 0 \\
0 & \sin _{\text {pitch }} & \cos _{\text {pitch }} & 0 \\
0 & 0 & 0 & 1
\end{array}\right] \\
&=\left[\begin{array}{cccc}
\cos _{\text {yaw }} & \sin _{\text {yaw }} \sin _{\text {pitch }} & \sin _{\text {yaw }} \cos _{\text {pitch }} & 0 \\
0 & \cos _{\text {pitch }} & -\sin _{\text {pitch }} & 0 \\
-\sin _{\text {yaw }} & \cos _{\text {yaw }} \sin _{\text {pitch }} & \cos _{\text {yaw }} \cos _{\text {pitch }} & 0 \\
0 & 0 & 0 & 1
\end{array}\right]
\end{aligned}
$$

通过矩阵R可以计算得到原始的forward=(0,0,-1,0)向量变换后的向量，计算结果为上述矩阵R第三列求反的结果，表示为：

```c
// direction代表摄像机的前轴(Front)，这个前轴是和本文第一幅图片的第二个摄像机的方向向量是相反的
Front.x = -cos(glm::radians(Pitch))*sin(glm::radians(Yaw));
Front.y = sin(glm::radians(Pitch));
Front.z = -cos(glm::radians(Pitch))*cos(glm::radians(Yaw));
Front = glm::normalize(Front);
```

### 鼠标输入

偏航角和俯仰角是通过鼠标（或手柄）移动获得的，水平的移动影响偏航角，竖直的移动影响俯仰角。它的原理就是，储存上一帧鼠标的位置，在当前帧中我们当前计算鼠标位置与上一帧的位置相差多少。如果水平/竖直差别越大那么俯仰角或偏航角就改变越大，也就是摄像机需要移动更多的距离。

首先我们要告诉GLFW，它应该隐藏光标，并捕捉(Capture)它。捕捉光标表示的是，如果焦点在你的程序上，光标应该停留在窗口中（除非程序失去焦点或者退出）。我们可以用一个简单地配置调用来完成：`glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED);`

在处理FPS风格摄像机的鼠标输入的时候，我们必须在最终获取方向向量之前做下面这几步：

1. 计算鼠标距上一帧的偏移量。
2. 把偏移量添加到摄像机的俯仰角和偏航角中。
3. 对偏航角和俯仰角进行最大和最小值的限制。
4. 计算方向向量。

```c
void mouse_callback(GLFWwindow* window, double xpos, double ypos)
{
    if(firstMouse)
    {
        lastX = xpos;
        lastY = ypos;
        firstMouse = false;
    }

    float xoffset = xpos - lastX;
    float yoffset = lastY - ypos; 
    lastX = xpos;
    lastY = ypos;

    float sensitivity = 0.05;
    xoffset *= sensitivity;
    yoffset *= sensitivity;

    yaw   += xoffset;
    pitch += yoffset;

    if(pitch > 89.0f)
        pitch = 89.0f;
    if(pitch < -89.0f)
        pitch = -89.0f;

    glm::vec3 front;
    front.x = cos(glm::radians(yaw)) * cos(glm::radians(pitch));
    front.y = sin(glm::radians(pitch));
    front.z = sin(glm::radians(yaw)) * cos(glm::radians(pitch));
    cameraFront = glm::normalize(front);
}
```

### 缩放

视野(Field of View)或fov定义了我们可以看到场景中多大的范围。当视野变小时，场景投影出来的空间就会减小，产生放大(Zoom In)了的感觉。我们会使用鼠标的滚轮来放大。与鼠标移动、键盘输入一样，我们需要一个鼠标滚轮的回调函数：

```c
void scroll_callback(GLFWwindow* window, double xoffset, double yoffset)
{
  if(fov >= 1.0f && fov <= 45.0f)
    fov -= yoffset;
  if(fov <= 1.0f)
    fov = 1.0f;
  if(fov >= 45.0f)
    fov = 45.0f;
}
```

当滚动鼠标滚轮的时候，yoffset值代表我们竖直滚动的大小。当scroll_callback函数被调用后，我们改变全局变量fov变量的内容。因为45.0f是默认的视野值，我们将会把缩放级别(Zoom Level)限制在1.0f到45.0f

### 摄像机类

不多说,看代码

如果你在哪儿卡住了，可以到[这里]((https://github.com/xiaoqide/opengl-note-code/blob/main/src/test_camera.cpp))查看源码。

---
title: OpenGL Note
date: 2021/08/02
tags: 
- graphics
- opengl
categories:
- note
- grpahics
- opengl
top: -1
keywords: opengl
---

## 核心模式和立即渲染模式

早期opengl是Immediate mode，没有提供太多的控制，易于使用但效率不够。从3.2版本开始，废弃了这个模式，推荐在core-profile下进行开发，这个模型更加灵活高效，但学习成本增加。

在使用core-profile模式下，使用早期的函数会抛出错误，因此，当存在问题是，应该优先查看使用的模式、opengl版本和使用的函数版本是否对应。

## 状态机

OpenGL是一个巨大的状态机，里面存储了许多状态，如缓冲区，绘制方式等等，每次绘制是都是根据当前的状态进行绘制的。所以，需要经常对buffer进行绑定和解绑。

## 几个数据

顶点数组对象，Vertex Array Object(VAO)：VAO存储了所有顶点数据属性的状态结合，它存储了顶点数据的格式以及顶点数据所需的VBO对象的引用。

顶点缓冲对象，Vertex Buffer Object(VBO)：GPU上有一部分内存用于存储顶点数据，VBO用于管理这部分内存，如何解释这个内存，如何发送给显卡。

索引缓冲对象，Element Buffer Object(EBO)或Index Buffer Object(IBO)：存储顶点的索引，根据索引来绘制三角形。当绘制大场景时，多个几何形状会复用顶点，若为每个集合都存储一个顶点数据，会浪费大量空间，而使用索引则可以节省空间，使用索引复用多个节点。

## glsl输入和输出

glsl使用in和out来指定输入和输出。两个着色器之间只要输出变量和输入变量的关键字匹配就可以传递下去。

vertexShader是第一个shader，直接从显存中获取数据，所以使用location来指定输入变量。

对于fragmentShader，需要一个vec4的输出，否则物体会是黑色或白色。

uniform是一种从CPU中的应用向GPU中的着色器发送数据的方式，uniform是全局的，uniform是独一无二的且可以在任意阶段被访问。若声明了一个uniform变量但是却没有使用，则编译是会将这个变量移除，导致一些错误。

## 纹理环绕方式

GL_REPEAT：重复纹理图像，在边缘是不平滑。

GL_MIRRORED_REPEAT：纹理图像镜像地重复。

GL_CLAMP_TO_EDGE：超出纹理范围的值取边缘的值。

GL_CLAMP_TO_BORDER：超出纹理坐标范围的值取用户设置的值。





所有的对象在opengl中都有一个id，opengl通过id来访问对象。


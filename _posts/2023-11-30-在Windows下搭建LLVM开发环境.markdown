---
layout: post
title: "在Windows下搭建LLVM开发环境"
date: 2023-11-30 20:47
categories: LLVM Compiler
---

最近在跟做LLVM官方教程[Kaleidoscope](https://llvm.org/docs/tutorial/)，完成Lexer和Parser两章后，就需要使用LLVM来生成IR了。在Linux和MacOS上安装LLVM并不难，只需要使用包管理器安装好LLVM，就能在项目中引用头文件了，但是在Windows上就没有这么简单。这几天用来编程的时间里，我啥都没做，就光研究怎么把LLVM装好，终于在今天下午搞定了，总共花了我大概一个星期的时间。下文我将写下安装、配置LLVM的详细步骤，方便日后再次安装时查阅。

# 准备工作

1. 因为我选择从源码构建，所以首先就要去Github需要下载[LLVM的源代码](https://github.com/llvm/llvm-project)；
2. 下载安装[Python3](https://www.python.org/downloads/)和[Visual Studio](https://visualstudio.microsoft.com/)。我们还需要[CMake](https://cmake.org/)来构建程序，不过因为Visual Studio已经内置了这个软件，所以不必单独安装。
3. 最后，我们需要更改一下系统设置，一次打开控制面板→”区域“设置→“管理”页，将“非Unicode程序的语言”改成“英文(美国)”。这个是成功编译的关键，否则编译器会抱怨源代码中存在某些非法Unicode，而报语法错误（详见这个Issue：[https://github.com/llvm/llvm-project/issues/60549](https://github.com/llvm/llvm-project/issues/60549)）。更改之后命令行将不能正常显示中文，安装完后记得把它改回来。

# 步骤

1. 去阅读这篇文章[Getting Started with the LLVM System using Microsoft Visual Studio](https://llvm.org/docs/GettingStartedVS.html)，重点阅读`Getting Started`一节。这篇文章详细说明了如何使用Visual Studio编译LLVM。编译出来的文件会放在`build`目录中；
2. `cd`进入`build`目录中，使用`cmake --build . --target install`安装LLVM。默认情况下这个命令将build目录中所需的文件安装到`C:\Program Files (x86)`中；
3. 安装完毕后，我们将LLVM的安装路径添加到系统环境变量`CMAKE_PREFIX_PATH`中（如果没有就新建一个）。以便我们能够`cmake`的`find_package`命令来引入LLVM；
4. 最后，按照文章[Building LLVM with CMake中的Embedding LLVM in your project](https://llvm.org/docs/CMake.html#embedding-llvm-in-your-project)一节，配置好`CMakeLists.txt`文件；
5. 完成，现在你可以在源代码中`#include "llvm/..."`了。

# 备注

1. 必须使用Visual Studio内的命令行才能使用cmake；
2. 编译和安装都挺耗时的，编译安装过程CPU会满载，从而让电脑很卡，所以建议将电脑扔一边干其他事情去。
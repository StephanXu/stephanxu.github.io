---
title: 利用vcpkg轻松安装C++库
categories:
  - [整理, 计算机]
date: 2019-12-10 14:18:47
tags:
author: 徐梓涵
abbrlink: 65004
---

在C++上配置各种库这件事情是不太容易的。并且在各种平台上可以有各种不同的配置方法。通常我们在使用一个C++库时都具有下面的几个步骤：

1. 下载库源代码
2. 编译库
3. 将库引入到项目中

虽然只有区区几步，但是在实现这些步骤时却需要做很多事情，并且稍有不慎可能就会导致最终的结果完全错误。

基于这个问题，微软开源了跨平台的包管理工具vcpkg，它可以在Windows、Linux和MacOS上运行，只需要用一行命令就可以安装C++库并自动地引入到项目中。

<!-- more -->

在下面的例子当中我们将尝试自己动手安装一个带有各种支持的OpenCV库。

这篇文章主要针对于Windows系统，如果你需要在Linux、MacOS上配置Vcpkg，请参考这篇微软的[文档](https://docs.microsoft.com/zh-cn/cpp/build/vcpkg)：

## 配置Vcpkg

### 下载并初始化vcpkg

vcpkg的整个程序托管在GitHub上，你可以在下面的链接当中把vcpkg克隆到本地或者直接下载它的压缩包解压到某个你认为正确的地方。之后vcpkg安装的各类库都会放在这个目录下。

```
https://github.com/Microsoft/vcpkg
```

我们在vcpkg的根目录（即可以看到`bootstrap-vcpkg.bat`文件的目录）的地址栏中直接输入`powershell`并按下回车，这将打开PowerShell窗口，在其中请输入下面的命令并回车初始化vcpkg：

```powershell
./bootstrap-vcpkg.bat
```

在执行完毕后，你可以尝试键入下面的命令并回车来验证你的vcpkg已经初始化完成：

```powershell
./vcpkg version
```

不出意外你应该可以看到一些版本信息的输出。这可以验证你的vcpkg根目录下已经生成了`vcpkg.exe`这一文件。请你右击这个文件查看属性，并在`常规->位置`这一项上复制其后的地址，在之后，我会将这个路径称为**vcpkg的根目录路径**。

### 安装集成

集成是为了让Visual Studio能够直接访问我们安装的库而不需要其他配置，请按下`win`+`x`键，然后在弹出的菜单中选择`Windows PowerShell (管理员)`，并输入命令（请将命令中文字“根目录路径”替换为刚刚复制的vcpkg的根目录路径，**注意请保留两侧的引号**）：

```PowerShell
cd "根目录路径"
```

例如我的命令看起来像这样：
```PowerShell
cd "C:\libs\vcpkg"
```

按下回车后，你应该可以看到命令行光标前的目录已经改变。此时，你应该关闭Visual Studio后输入下面的命令并回车：

```powershell
./vcpkg integrate install
```

此时你已经安装好了Visual Studio的集成，在Visual Studio创建的项目VC++项目当中你可以直接引入头文件调用该库。如果你的项目是CMake项目，你可能需要将`CMAKE_TOOLCHAIN_FILE`设置为`vcpkg根目录路径/scripts/buildsystems/vcpkg.cmake`。这个提示通常在你执行上面的命令后获得的输出的最后一行出现。

## 安装库

此时vcpkg已经在计算机中被安装好了，请在vcpkg的根目录的地址栏中直接输入`powershell`并按下回车，这将再次打开PowerShell窗口（你可以注意到PowerShell窗口的标题栏上没有“管理员”字样），在其中请输入下面的命令搜索OpenCV库：

```powershell
./vcpkg search opencv
```

可以看到得到的结果中OpenCV后有中括号，其中带有各类支持选项，例如可以选择安装`contrib`或`cuda`支持。此处我们可以按照下面的选项在vcpkg中下安装OpenCV（这不是唯一的方法）：

```PowerShell
./vcpkg install opencv[nonfree,contrib,ffmpeg]:x64-windows
```

注意，上面的命令中开启了三个支持开关，你可以自由地删去或增加一些。另外，我们利用冒号`:`描述了这个库的安装选项，即在`Windows`平台下使用`x64`进行编译，如果不特别描述，vcpkg会使用默认的`x86-windows`设置。在其他系统（例如Linux、MacOS）下也可以设置到对应的操作系统。这一项设置对下面在项目中使用非常重要，如果项目设置为x64平台而库的安装使用了x86平台，由于项目与库的平台不兼容，之后当项目尝试使用库时IntelliSense将不会列出这个库的头文件。

此时按下回车，vcpkg将按照OpenCV的依赖依次安装需要的库，所以不需要担心可能有任何路径上或者依赖上的问题。vcpkg会自动从待安装库的源代码仓库拉去源代码，而通常库的源代码会被托管在GitHub上，由于大陆的网络环境比较特殊，这可能带来几次在下载途中的失败，你可以尝试重新运行上面的命令，也可以为计算机配置代理。安装的过程通常需要持续数分钟到数十分钟。

## 在项目中引用

在OpenCV安装完成后，你可以打开某个项目或创建某个新项目，并在项目中直接使用安装好的OpenCV库，值得注意的是，如果此前安装的是x64编译选项的库，那么它仅在平台设置为x64的项目中有效，同理，如果安装的是x86选项的库则对应平台设置为x86的项目：

![在项目中使用](引入头文件.gif)

在编译完成后，vcpkg与Visual Studio的集成将能够把所有依赖的动态链接库文件（Dll文件）拷贝到输出的可执行文件（Exe文件）的目录下。所以你的输出目录可能如下图：

![输出文件](输出文件夹.png)

如果你只是像上面的动图一样只引用了头文件而没有写任何使用到OpenCV的代码，你的输出目录仍然可能不包括上面的这些动态链接库文件（Dll文件）。这时你可以利用下面的示例代码来输出OpenCV的版本号。

```C++
#include <iostream>
#include <opencv2/opencv.hpp>
int main()
{
	std::cout << CV_VERSION << std::endl;
}
```

此时，你已经利用vcpkg完成了对OpenCV库的安装。你可以继续尝试安装或使用其他的库。
---
title: Caffe 以及 Caffe-SSD 的配置
categories:
  - [整理, 计算机]
abbrlink: 17811
date: 2018-11-15 00:46:49
tags:
author: 徐梓涵
---

这段时间装了很多次Caffe，总结了一些方法和经验。此处笔者安装的是带GPU支持的Caffe。但是CPU ONLY的方法与GPU安装差异不大。笔者会在其中说明。

<!-- more -->

## 第一部分 Nvidia 环境安装

这一部分我们需要安装NVIDIA的基本环境，所以，**并非使用 Nvidia GPU 的读者以及使用CPU的读者可以跳过这一部分**。这一部分主要包括：

- Nvidia 驱动程序
- CUDA（此处我们安装8.0版本）
- cudnn（此处我们安装6.1版本）

### Nvidia 驱动程序

在Ubuntu中，你可以在你的 设置->软件和更新->附加驱动 中看到Nvidia的驱动，你可以尝试安装此驱动。当然，如果你希望手动安装，可以到[Nvidia的网站](https://www.nvidia.cn/Download/index.aspx?lang=cn)选择你的型号来下载驱动程序。

![nvidia-select](C:\Users\xuzih\OneDrive\文档\博客\Caffe 以及 Caffe-SSD 在Ubuntu16.04上的配置\nvidia-select.png)

在Ubuntu 16.04中由于系统已经提供了较好的驱动可以直接使用，所以此处推荐直接使用该驱动。驱动安装完成以后可以在终端运行命令`nvidia-smi`查看GPU信息。

### CUDA

首先你需要到 Nvidia的网站[下载CUDA](https://developer.nvidia.com/cuda-downloads?)。截至本文撰写前，CUDA Toolkit的最新版本是10.0。此处笔者点击`Legacy Releases`找到了8.0的版本下载。为了安装方便，笔者选择下载`run`文件。

![download-cuda](C:\Users\xuzih\OneDrive\文档\博客\Caffe 以及 Caffe-SSD 在Ubuntu16.04上的配置\download-cuda.png)

下载完成以后，按`Ctrl+Alt+F1`进入字符终端界面，按照正常方式登陆之后，运行下面的命令：

```
$ sudo service lightdm stop #关闭图形界面
$ sudo bash cuda_8.0.61_375.26_linux.run
```

此时会出现安装协议，只需要按空格键即可快速跳过，接受协议。之后会询问是否安装驱动。由于我们现在已经安装了 Nvidia驱动，所以已经不需要再安装驱动（注：笔者在不安装Nvidia驱动的情况下直接采用此驱动也会安装失败）。安装完成后，就可以启动图形界面了。

```
$ sudo service lightdm start
```

要检查CUDA是否安装完成，可以直接在终端运行`nvcc -V`即可输出CUDA版本信息。现在我们开始安装cudnn，首先解压cudnn的压缩包到当前文件夹（即一个名为cuda文件夹）

```
$ cd cuda
$ cd include
$ sudo cp cudnn.h /usr/local/cuda/include/ #复制头文件
$ cd ..
$ cd lib64
$ sudo cp lib* /usr/local/cuda/lib64/ #复制动态链接库
$ cd /usr/local/cuda/lib64/
$ sudo chmod +r libcudnn.so.6.0.21 #此处注意笔者安装的是6.0.21版本，你的版本可能与此不同
$ sudo ln -sf libcudnn.so.6.0.21 libcudnn.so.6 #链接到大版本
$ sudo ln -sf libcudnn.so.6 libcudnn.so
$ sudo ldconfig #使链接生效
```

到此时，我们就已经安装好了基本的CUDA环境。接下来，我们需要设定几个环境变量来使得我们的设置生效。如果想要这个设置对所有用户生效，就在命令行运行

```
sudo gedit /etc/profile
```

如果仅仅想要对本用户有效，则运行：

```
sudo gedit ~/.bashrc
```

然后在文件中添加如下内容，其中的cuda版本需要根据你的实际情况进行替换：

```
export PATH=$PATH:/usr/local/cuda-8.0/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-8.0/lib64
export LIBRARY_PATH=$LIBRARY_PATH:/usr/local/cuda-8.0/lib64
```

保存好之后重启终端或直接运行`source ~/.bashrc`或`source /etc/profile`。

到这里CUDA就已经安装完成了，接下来我们就开始安装Caffe。

## Caffe 安装

### 安装依赖

首先运行以下命令安装好对应的依赖：

```
$ sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler
$ sudo apt-get install --no-install-recommends libboost-all-dev
$ sudo apt-get install libopenblas-dev liblapack-dev libatlas-base-dev
$ sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
$ sudo apt-get install git cmake build-essential
```

安装的同时也可以下载对应的Caffe的程序：

如果是安装原版的Caffe：

```
$ git clone https://github.com/BVLC/caffe
```

如果是安装Caffe-SSD：

```
$ git clone https://github.com/weiliu89/caffe.git caffe-ssd
$ cd caffe-ssd
$ git checkout ssd
```

此时caffe文件夹（如果安装的Caffe-SSD即为caffe-ssd）就能够下载到终端当前的文件夹下。依赖安装完成以后，进入到`caffe/python`文件夹下，运行下面的命令安装python的依赖：

```
$ pip install -r requirements.txt
```

### 编译配置

安装完成后回退到Caffe的根目录，制作配置文件：

```
sudo cp Makefile.config.example Makefile.config
sudo gedit Makefile.config
```

编辑其中的内容，将：

- #USE_CUDNN := 1 前的井号去掉
- #OPENCV_VERSION := 3 前的井号去掉
- #WITH_PYTHON_LAYER := 1 前的井号去掉

针对一些包含库的路径，将：

```
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib 
```

替换为：

```
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial  
```

保存文件后，接下来在根目录下运行下面的命令创建我们的编译目录并打开图形界面的cmake：

```
$ mkdir build
$ cmake-gui
```

将`Where is the source code`设置为Caffe的根目录，将`Where to build the binaries`设置为我们刚刚创建的build文件路径。点击`Configure`按钮列出选项。

值得注意的是，列表中的python_version选项，如果你需要使用python3，则需要将默认的2修改为3。然后勾选上面的`Advance`多选框。在左侧的搜索框中搜索python，然后修改`PYTHON_EXECUTABLE`、`PYTHON_INCLUDE_DIR`、`PYTHON_LIBRARY`。下面是一个简单的例子。

![cmake-config1](C:\Users\xuzih\OneDrive\文档\博客\Caffe 以及 Caffe-SSD 在Ubuntu16.04上的配置\cmake-config1.png)

接下来再搜索`CMAKE_CXX_FLAGS`并在其中添加`-std=c++11`以启用编译器C++11标准的支持。

最后点击`Generate`按钮，即可完成编译的配置。

### 编译

编译时先进入到`build`目录，运行下面的命令：

```
$ make all -j4 #j之后的数字是线程数，可根据cpu的性能进行设置，此处笔者使用的设备为两颗i7 8700所以常设为12或者16。
$ sudo make test
$ sudo make runtest
```

最后当编译完成，runtest也提示pass之后即说明Caffe的安装基本成功。

### 编译问题说明

在编译过程中有可能有如下问题，笔者会根据读者的反馈进行增加：

- libTiff异常：make的最后的链接阶段提示`@LIBTIFF_4.0未定义的引用`，此时读者应检查opencv的版本是否异常，例如笔者自行编译的opencv3.4.3就与自带的2.4版本有冲突，需要在之前cmake-gui设置的时候把`OpenCV_DIR`设置为你的OpenCV路径（注意，此路径下需要包含OpenCVConfig.cmake文件，通常是源码安装的OpenCV中的build文件夹下）。如果不是该问题则请检查libTiff是否安装。

### 扫尾工作

如果需要在Python中使用Caffe接口，则请运行：

为所有用户设置：

```
sudo gedit /etc/profile
```

为当前用户设置：

```
sudo gedit ~/.bashrc
```

在其中添加如下内容（注意其中路径需要替换）：

```
export PYTHONPATH=/path/to/caffe/python:$PYTHONPATH
```

最后重启终端，或者运行`source ~/.bashrc`或`source /etc/profile`即可完成安装。在python中直接：

```
import caffe
```

若无报错即安装成功。
---
title: LXD虚拟化服务器
categories:
  - [计算机]
abbrlink: 36478
date: 2020-07-05 02:37:24
tags:
author: 徐梓涵
---

本文档记录了简单的LXD虚拟化方案，能够实现

- 在物理机上创建多个隔离的容器环境
- 在容器环境中运行桌面
- 将GPU接入容器环境中，并在多个容器间共享

<!-- more -->

## 快速配置

### 宿主机

快速初始化宿主机

```bash
# 安装对应依赖
sudo apt install lxd zfsutils-linux bridge-utils

# 创建存储池
sudo lxc storage create <storage-poll-name> zfs size=256GB

# 初始化lxd
sudo lxd init

# 添加存储池
sudo lxc profile device add default root disk path=/ pool=<storage-poll-name>

# 添加清华镜像
sudo lxc remote add tuna-images https://mirrors.tuna.tsinghua.edu.cn/lxc-images/ --protocol=simplestreams --public
```

快速创建新容器

```bash
# 创建容器（此处对应ubuntu/18.04镜像）
sudo lxc launch tuna-images:ubuntu/18.04 <container-name>

# 添加GPU（此处添加第一个GPU）
sudo lxc config device add <container-name> gpu0 gpu id=0
```

### 容器

```bash
# 设置密码，此处对应的是ubuntu/18.04镜像
passwd root
passwd ubuntu

# 安装SSH
apt update
apt install openssh-server

# 安装图形界面
apt install --no-install-recommends ubuntu-desktop

# 安装NVIDIA驱动（如果有）
sudo sh ./NVIDIA-Linux-X86_64-<driver-version>.run --no-kernel-module

# 安装rdp
wget "https://www.c-nergy.be/downloads/xrdp-installer-1.2.zip"
unzip xrdp-installer-1.2.zip
chmod +x xrdp-installer-1.2.sh
./xrdp-installer-1.2.sh
```

## 常用命令

查看清华源中的镜像列表：

```bash
sudo lxc image list tuna-images:
```

显示容器信息

```bash
sudo lxc info <container-name>
```

进入容器

```bash
sudo lxc exec <container-name> /bin/bash
```

推送文件到容器中

```bash
sudo lxc file push <source> <container-name>/path/to/destination
```

从容器中取出文件

```bash
sudo lxc file pull <container-name>/path/to/source <destination>
```

保存当前容器为镜像

```bash
sudo lxc publish <container-name> --alias <image-name> --public
```

生成快照

```bash
sudo lxc snapshot <container-name> <snapshot-name?>
```

还原快照

```bash
sudo lxc restore <container-name> <snapshot-name>
```

导出镜像

```bash
sudo lxc export <image-name> <file-path>
```

导入镜像

```bash
sudo lxc import <file-path> --alias <image-name>
```

## 端口映射办法

默认SSH映射在外网：6X22

默认RDP映射在外网：6X89

## 桥接网络配置

```
  eth0:
    name: eth0
    nictype: macvlan
    parent: enp34s0
    type: nic
```
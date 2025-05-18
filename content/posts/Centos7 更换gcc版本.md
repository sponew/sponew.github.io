---
date: '2025-05-18T14:07:36+08:00'
draft: flase
title: 'Centos7 更换 GCC 版本'
tags: ["CentOS", "CentOS7", "GCC", "SCL Repo"]
---





**Centos7 更换 GCC 版本**

`CentOS7`上面经常需要安装一些新的应用，可能软件仓库或者软件本身已经没有`rpm`包了，就只能通过`GCC`编译安装了，而有些新的软件需要使用新版本的`GCC`版本。

### 通过 SCL 安装

启用CentOS7 `SCL`仓库，这个仓库允许多版本共存，不影响旧版本的`GCC`。

```bash
# 安装 SCL 工具链
yum install centos-release-scl

# 安装 GCC 11
yum install devtoolset-11-gcc devtoolset-11-gcc-c++

# 启用 GCC 11
scl enable devtoolset-11 bash

# 查看 GCC 版本
gcc --version
```
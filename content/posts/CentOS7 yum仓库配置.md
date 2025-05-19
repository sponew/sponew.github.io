---
date: '2025-05-19T17:07:13+08:00'
draft: flase
title: 'CentOS7 yum仓库配置'
tags: ["CentOS", "CentOS7", " Repo", "本地仓库", "Mirrors"]
---

[TOC]

# CentOS 7 软件仓库配置与加速指南

CentOS 7 的软件包管理器使用的是`yum`,它能够自动解决软件包的依赖关系，并从指定的软件仓库（Repository）下载和安装软件。


- **[阿里云 CentOS 镜像说明](https://developer.aliyun.com/mirror/centos)**


## 1. 备份原有仓库
   
```bash
sudo mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```

## 2. 下载阿里云镜像加速

```bash

# 使用 wget 下载
sudo wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

# 或者使用 curl 下载
sudo curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

```bash
# 非阿里云ECS用户使用
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
```

## 3. 清除原有缓存并新建缓存

```bash
# 清空原有缓存
sudo yum clean all

# 生成缓存
sudo yum makecache

# 测试 yum 源
yum repolist
```
如果能看到 `mirrors.aliyun.com` 源，说明配置成功。

## 4. 补充： 配置代理上网

有些客户环境上网条件限制比较大，如果提供了代理上网选项，可以尝试通过代理来拉取镜像

1. 仓库全局配置代理

假设我们想要为 `/etc/yum.repos.d/` 下的所有仓库设置代理，则可以修改 `/etc/yum.conf` 文件，在该文件尾部添加如下一行：

```ini
proxy=http://ip:port
```

2. 特定全局配置代理
   
假设 `CentOS-Base.repo`中有三个仓库：base、updates和extras。我们只想给base仓库设置代理，则只需要在对应的仓库后面添加一行`proxy=http://ip:port`，如下：
```
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
proxy=http://ip:port
```

3. 特定仓库不走代理

假设我们在`/etc/yum.conf`中设置了全局代理，但是又想某个仓库不走代理，此时，我们只需要在特定的仓库后面添加一行 `proxy=_none_` 即可。比如 `CentOS-Base.repo`中有三个仓库，我们希望base仓库不走代理，则设置如下：
```
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
proxy=_none_
```

</br>
</br>

---

</br>


# CentOS7 配置本地 YUM 仓库（使用本地 ISO 或光盘作为软件源）

## 1. 挂载ISO镜像

假设你的ISO镜像文件为 `CentOS-7-x86_64-DVD-xxxx.iso`，并且放在 `/root` 目录下。

```bash
# 创建挂载目录
mkdir -p /mnt/iso

# 挂载ISO镜像
mount -o loop /root/CentOS-7-x86_64-DVD-xxxx.iso /mnt/iso
```

## 2. 备份原有本地YUM源

```bash
mv /etc/yum.repos.d/local.repo /etc/yum.repos.d/local.repo.bak
```

## 3. 创建本地YUM源配置文件

新建一个本地repo文件 `/etc/yum.repos.d/local.repo`

```ini
[local]
name=CentOS7-Local
baseurl=file:///mnt/iso
enabled=1
```

## 4. 清理并生成缓存

```bash
yum clean all
yum makecache
```

## 5. 测试YUM源

```bash
yum repolist
```

如果能看到 `local` 源，说明配置成功。

## 6. 安装软件包

安装本地 RPM 包（自动解决依赖）：
```bash
yum localinstall xxx.rpm
```

</br>
</br>

---

</br>

# CentOS7 本地 repo 制作和 rpm 导入

## 1. 搭建本地 YUM 源

1. 安装 `createrepo` 工具
```bash
yum install -y createrepo
# 如果没有网络环境，可以提前准备rpm包安装
rpm -ivh createrepo.rpm
```

2. 创建本地仓库目录
```bash
mkdir -p /data/yumrepo
cd /data/yumrepo
```

3. 拷贝 RPM 包到仓库目录
将所有需要的 RPM 包放到 `/data/yumrepo` 目录下。

4. 生成仓库元数据
```bash
createrepo .
```

5. 备份原有YUM源

```bash
mv /etc/yum.repos.d/local.repo /etc/yum.repos.d/local.repo.bak
```

6. 配置本地 YUM 源

编辑 `/etc/yum.repos.d/local.repo` 文件，内容如下：

```ini
[local]
name=Local YUM Repo
baseurl=file:///data/yumrepo
gpgcheck=0
enabled=1
```

7. 清理并生成缓存
```bash
yum clean all
yum makecache
```

8. 查询已配置的 YUM 源：
```bash
yum repolist all
```

## 2. 导入 RPM 包到本地仓库

1. 将新的 RPM 包复制到 `/data/yumrepo` 目录。

*需将所有依赖包一并放入本地仓库*

2. 重新生成仓库元数据：

```bash
createrepo --update /data/yumrepo
```

3. 再次执行 `yum makecache`。

## 3. 安装软件包

安装本地 RPM 包（自动解决依赖）：
```bash
yum localinstall xxx.rpm
```
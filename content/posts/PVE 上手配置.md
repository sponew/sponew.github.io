---
date: '2025-02-18T12:24:03+08:00'
draft: true
title: 'PVE 上手配置'
#categories: ["通用技术"]
tags: ["PVE", "虚拟化", "Server", "All in Boom"]
---




***Proxmox Virtual Environment (PVE) 是一个开源的虚拟化平台，集成了 KVM 和 LXC 技术，提供易于管理的虚拟机和容器解决方案。***

# 1. 修改 PVE 软件源

Proxmox VE 基于 debian发行版，使用APT作为软件包管理器。

将PVE的软件仓库源替换为国内镜像源，将PVE的企业源替换为社区源，适用于PVE8

`/etc/apt/sources.list`文件和`/etc/apt/sources.list.d/`目录下的`.list`文件，定义了软件仓库源列表。



## 备份现有的 apt 源配置文件

```bash
#备份软件仓库
cp /etc/apt/sources.list /etc/apt/sources.list.bak
#备份企业仓库
cp /etc/apt/sources.list.d/pve-enterprise.list /etc/apt/sources.list.d/pve-enterprise.list.bak
#备份 Ceph 仓库
cp /etc/apt/sources.list.d/ceph.list /etc/apt/sources.list.d/ceph.list.bak
```



## 禁用企业源

编辑 pve-enterprise.list 文件，将其内容注释掉：

```
vi /etc/apt/sources.list.d/pve-enterprise.list
```

```TEXT
#deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise
```



## 配置清华镜像源

### 更换软件仓库镜像源

```bash
vi /etc/apt/sources.list
```

```TEXT
#清华 Proxmox 源 适用于 Debian12
# Proxmox 社区源
deb https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian/pve bookworm pve-no-subscription

# Debian 源
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib

# 安全更新
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bookworm-security main contrib
```

### 更换 Ceph 仓库镜像源

```bash
vi /etc/apt/sources.list.d/ceph.list
```

```TEXT
# Proxmox Ceph 社区源
deb https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian/ceph-quincy bookworm main
```



## 更新 apt 密钥

添加清华镜像源的 GPG 密钥

```bash
wget -qO - https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian/proxmox-release-bookworm.gpg | gpg --dearmor -o /usr/share/keyrings/proxmox-release-bookworm.gpg
```



## 更新软件包列表及系统

```bash
apt update && apt upgrade
```



## 去除订阅弹窗

PVE 在没有订阅的情况下，每次登录 Web 界面时会弹出订阅提示弹窗。订阅弹窗是由 PVE 的 Web 界面 JS 文件控制的，我们可以修改这个文件去掉弹窗。

```bash
#备份 js 文件
cp /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js.bak
#文件中的 checked_command: function(orig_cmd) 函数所执行的弹窗逻辑
#注释 Ext.Msg.show 方法
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
#重启进程
systemctl restart pveproxy.service
```

执行后清理浏览器缓存，重新登录验证。

# 2. QEMU/KVM

## 内核模块

将以下内容添加到配置文件`/etc/modules`中，确保内核加载相应模块

```TEXT
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

然后还需要更新initramfs。命令行如下：

```bash
update-initramfs -u -k all
```



## IOMMU

### 开启 IOMMU

进入 BIOS 设置界面，查找以下选项并启用

```TEXT
Intel CPU: VT-d (Intel Virtualization Technology for Directed I/O)
AMD CPU: IOMMU 或 SVM
```

如果你的内核版本较旧（低于 6.8），可能需要手动通过内核参数启用 IOMMU。

### 确认 IOMMU 是否已启用

```bash
dmesg | grep -e DMAR -e IOMMU
```

应该有一行类似于 `DMAR： IOMMU enabled`。如果没有输出，则说明有问题。

### 开启 IOMMU Pass-Through 模式

**优点：** 	

**性能提升**：减少 Hypervisor 处理 DMA 请求的开销，降低延迟。	

**更接近原生的设备性能**：VM 中的设备可以直接访问硬件资源。

**更高的隔离性**：利用 IOMMU 实现物理设备与内存的直接映射，避免越界访问。

**开启方法：**

```bash
vi /etc/default/grub
```

```TEXT
#找到 GRUB_CMDLINE_LINUX_DEFAULT 行并添加以下内容：
# intel
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
# amd
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on iommu=pt"
```

更新 GRUB 配置

```bash
update-grub
```

重启系统

```bash
reboot
```

验证是否生效

```bash
dmesg | grep -e DMAR -e IOMMU
```

### 验证 IOMMU 中断重新映射是否已启用

如果不进行中断重新映射，则无法使用 PCI 直通。设备分配将失败，并显示“无法分配设备 '[设备名称]”： 不允许操作“或”找不到中断重映射硬件，将设备传递到非特权域不安全”。

所有使用 Intel 处理器和芯片组的系统，如果支持用于定向 I/O （VT-d） 的 Intel 虚拟化技术，但不支持中断重新映射，则会看到此类错误。较新的处理器和芯片组（AMD 和 Intel）提供中断重映射支持。

要确定您的系统是否支持中断重新映射：

```bash
dmesg | grep 'remapping'
```

### 加载vifo系统模块

将以下内容添加到配置文件`/etc/modules`中，确保内核加载相应模块

```TEXT
# Modules required for PCI passthrough
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

更新 initramfs

```bash
update-initramfs -u -k all
```



## 中介设备 

### Intel GVT-g

#### 主机配置

Intel的GVG-g驱动已经集成在Linux内核，在第5、6、7代Intel Core CPU和E3 v4 v5 v6版本的Xeon CPU可以直接使用

安装对应软件包

```bash
apt install -y intel-opencl-icd
```

将`i915.enable_gvt=1`添加到配置文件/etc/default/grub`中，确保内核加载相应模块

```TEXT
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt i915.enable_gvt=1
```

更新 GRUB 配置

```bash
update-grub
```

加载GVT模块

将以下内容添加到配置文件`/etc/modules`中，确保内核加载相应模块

```TEXT
# Modules required for Intel GVT
kvmgt
exngt
Vfio-mdev
```

更新 initramfs

```bash
update-initramfs -u -k all
```

重启

```bash
reboot
```

确认是否加载 GVT

```bash
dmesg | grep -i "i915"	
```

#### 虚拟机配置

```bash
#通过sysfs查看所支持的硬件设备 以0000:00:02.0为例
ls /sys/bus/pci/devices/0000:00:02.0/mdev_supported_types

qm set VMID -hostpci0 00:02.0,mdev=i915-GVTg_V5_4

```

# 3. PVE 常用命令

**`qm` 命令**

`qm` **命令是** `Proxmox VE`**（虚拟化平台）中的一个命令行工具，用于管理虚拟机。它主要用于创建、管理、配置虚拟机以及执行一些常见的虚拟化操作。以下是一些常用的命令：**

```bash
# 创建虚拟机
qm create <vmid> --name <vm_name> --memory <memory_size> --net0 virtio,<network_bridge> --ide0 <storage>:<disk_size>
# 启动虚拟机
qm start <vmid>
# 停止虚拟机
qm stop <vmid>
# 重启虚拟机
qm reboot <vmid>
# 查看虚拟机状态
qm status <vmid>
# 查看虚拟机配置
qm config <vmid>
# 导入虚拟机镜像
qm importdisk <vmid> <disk_image.img> <storage>
# 克隆虚拟机
qm clone <vmid> <new_vmid> --name <new_vm_name>
# 删除虚拟机
qm destroy <vmid>
# 备份虚拟机
qm backup <vmid> <backup_path>
```

**`pvesh`命令及其他常用命令**

```bash
# 查看所有虚拟机列表
pvesh get /nodes/<node>/qemu
# 查看集群状态
pvecm status
# 查看节点的硬件信息
pvesh get /nodes/<node>/hardware
# 启动虚拟机
pvesh create /nodes/<node>/qemu/<vmid>/status/start
# 停止虚拟机
pvesh create /nodes/<node>/qemu/<vmid>/status/stop
# 重新启动虚拟机
pvesh create /nodes/<node>/qemu/<vmid>/status/reboot
# 创建虚拟机备份
vzdump <vmid> --mode snapshot --storage <storage>
# 查看 PVE 存储列表
pvesh get /nodes/<node>/storage
# 创建新的存储池
pvesh create /nodes/<node>/storage --storage <storage_name> --type <storage_type> --content <content_type> --path <path_to_storage>
# 更新 PVE 节点
apt update && apt upgrade
# 查看磁盘使用情况
df -h
# 查看系统日志
journalctl -xe
```



# 参考

[Proxmox VE Documentation](https://pve.proxmox.com/pve-docs/)

[Promxox VE 中文文档](https://pve-doc-cn.readthedocs.io/zh-cn/latest/index.html#)


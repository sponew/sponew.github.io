---
date: '2025-05-18T14:16:05+08:00'
draft: true
title: 'Centos7 更换阿里云归档仓库镜像'
tags: ["CentOS", "CentOS7", "Mirrors", "Repo", "Aliyun"]
---


**centos7 更换阿里云归档仓库镜像**

### 备份原有仓库配置
```
mkdir ~/yum.repos.d.backup
mv /etc/yum.repos.d/* ~/yum.repos.d.backup/
```

### 修改 CentOS 7 的归档仓库配置
编辑 /etc/yum.repos.d/CentOS-Base.repo，添加以下内容：
```
[base]
name=CentOS-7 - Base (Vault)
baseurl=https://mirrors.aliyun.com/centos-vault/7.9.2009/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[updates]
name=CentOS-7 - Updates (Vault)
baseurl=https://mirrors.aliyun.com/centos-vault/7.9.2009/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[extras]
name=CentOS-7 - Extras (Vault)
baseurl=https://mirrors.aliyun.com/centos-vault/7.9.2009/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[sclo]
name=CentOS-7 - SCLo (Vault)
baseurl=https://mirrors.aliyun.com/centos-vault/7.9.2009/sclo/$basearch/sclo/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo

[sclo-rh]
name=CentOS-7 - SCLo rh (Vault)
baseurl=https://mirrors.aliyun.com/centos-vault/7.9.2009/sclo/$basearch/rh/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo
```
### 清理并重建 Yum 缓存
yum clean all
yum makecache
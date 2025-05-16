---
date: '2025-02-11T15:23:01+08:00'
draft: false
title: '自建 RustDesk 远程桌面连接服务'
tags: ["RustDesk", "Remote Desktop", "Server"]
---

# **安装指南**



## **步骤 1：下载服务器端程序**

使用 Docker 镜像 [rustdesk/rustdesk-server](https://hub.docker.com/r/rustdesk/rustdesk-server/tags) 进行安装：

```bash
docker pull rustdesk/rustdesk-server
```



## **步骤 2：安装服务器端程序**

### **方式 1：通过 Docker 安装**

如果在运行 Docker 时要求输入注册码，说明你下载的是旧版本。国内的 Docker 镜像缓存可能未及时更新，请确保拉取最新版镜像。

运行以下命令启动服务（其中 hbbs 和 hbbr 分别对应服务端和中继服务器）：

```bash
sudo docker image pull rustdesk/rustdesk-server

sudo docker run --name hbbs \
  -p 21115:21115 \
  -p 21116:21116 \
  -p 21116:21116/udp \
  -p 21118:21118 \
  -v `pwd`:/root \
  -td --net=host \
  rustdesk/rustdesk-server hbbs 

sudo docker run --name hbbr \
  -p 21117:21117 \
  -p 21119:21119 \
  -v `pwd`:/root \
  -td --net=host \
  rustdesk/rustdesk-server hbbr
```

### **方式 2：使用 Docker Compose**

**说明：**

参数 `--net=host` 仅适用于 Linux 系统，它可以让 hbbs/hbbr 获取到对方真实 IP，而不是固定的容器 IP（例如 172.17.0.1）。如果 `--net=host` 正常运行，则端口映射（`-p` 选项）将失效，此时可移除 `-p` 参数。

*如果在非 Linux 系统上遇到连接问题，请去掉* `--net=host` *参数。*

**以下是一份 `Docker Compose` 示例配置（请根据需要自定义端口和挂载目录）：**

```bash
networks:
  rustdesk-net:
    external: false

services:
  hbbs:
    container_name: hbbs
    image: rustdesk/rustdesk-server
    command: hbbs 
    ports:
      - 21115:21115
      - "<hbbs_port>:21116"      # 自定义 hbbs 映射端口
      - "<hbbs_port>:21116/udp"  # 自定义 hbbs 映射端口
    volumes:
      - "<mount_path>:/root"    # 自定义挂载目录
    networks:
      - rustdesk-net
    depends_on:
      - hbbr
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 64M

  hbbr:
    container_name: hbbr
    image: rustdesk/rustdesk-server
    command: hbbr
    ports:
      - "<hbbr_port>:21117"      # 自定义 hbbr 映射端口
    volumes:
      - "<mount_path>:/root"     # 自定义挂载目录
    networks:
      - rustdesk-net
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 64M
```



## **步骤 3：在客户端设置 hbbs/hbbr 地址**

1. 在客户端中点击 ID 右侧的菜单按钮，然后选择 “ID/中继伺服器”。

2. 在 “ID 服务器” 输入框中（适用于被控端和主控端）填写 hbbs 主机名或 IP 地址，其他两个地址可留空（RustDesk 会自动推导，除非做了特殊设置）。

 3. “中继服务器” 指定为 hbbr 所在地址，默认使用 21117 端口。

    

    例如:

```
hbbs.example.com
```

​		或指定端口：

```
hbbs.example.com:21116
```





## **在可执行文件名中嵌入配置（仅限 Windows）**

你可以将 `rustdesk.exe` 重命名为包含配置信息的文件名，格式如下：

```
rustdesk-host=<host-ip-or-name>,key=<public-key-string>.exe
```

例如：

```
rustdesk-host=192.168.1.137,key=xfdsfsd32=32.exe
```

启动后，可在 “About” 窗口中看到配置结果。



**注意：**

​	•	host 和 key 参数必须同时提供，否则配置将无法生效。

​	•	如果密钥（key）中包含文件名不允许的字符，请删除 `id_ed25519` 文件，然后重启 hbbs/hbbr。系统会重新生成 `id_ed25519.pub` 文件。重复此过程，直至生成的密钥没有无效字符。



## **关于 Key**

与之前的版本不同，新版本强制要求使用 Key，但无需手动设置。在第一次运行时，hbbs 会自动生成一对密钥（id_ed25519 和 id_ed25519.pub 分别位于运行目录下），用于通信加密。

如果在客户端配置中未填写 Key:（即未使用 `id_ed25519.pub` 中的内容），仍可完成连接，但通信将不会加密。

### 查看公钥内容：

```bash
cat ./id_ed25519.pub
```

若需更换 Key，请删除 `id_ed25519` 和 `id_ed25519.pub` 文件，然后重新启动 hbbs/hbbr，系统会自动生成新的密钥对。
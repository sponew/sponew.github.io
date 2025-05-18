---
date: '2025-05-18T16:58:39+08:00'
draft: false
title: 'Ubuntu 24.04 LTS 安装 Zabbix 7'
tags: ["zabbix", "监控", "Server", "告警"]
---


### 1. Zabbix 组件选型

[Zabbix 官网](https://www.zabbix.com/cn/download?zabbix=7.0&os_distribution=ubuntu&os_version=24.04&components=server_frontend_agent_2&db=mysql&ws=nginx)

基于 Ubuntu 24.04 LTS 版本，MySQL 数据库，Nginx 安装。
Server 的自我监控 Agent 采用 Agent2 监控 Zabbix Server

### 2. 新建 Zabbix 用户
```
# 创建用户组
groupadd --system zabbix

# 创建用户并加入 zabbix 组
useradd -m -d /home/zabbix -s /bin/bash -g zabbix zabbix

# 创建密码
passwd zabbix

# ubuntu 赋予 sudo 权限
usermod -aG sudo zabbix

# 确认创建成功
id zabbix
ls -ld /home/zabbix
```
### 3. 在线安装 Zabbix Server 端

**参考官方在线安装手册 **
[手册](https://www.zabbix.com/cn/download?zabbix=7.0&os_distribution=ubuntu&os_version=24.04&components=server_frontend_agent_2&db=mysql&ws=nginx)

a. 安装 Zabbix 官方仓库
```
# wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
# dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
# apt update
```

b. 安装 Zabbix server, frontend, agent2
```
# apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts zabbix-agent2
```

c. 安装 Zabbix agent 2 插件
```
# apt install zabbix-agent2-plugin-mongodb zabbix-agent2-plugin-mssql zabbix-agent2-plugin-postgresql
```

d. 创建初始数据库
```
# mysql -uroot -p
password
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> set global log_bin_trust_function_creators = 1;
mysql> quit;
```

导入初始架构和数据，系统将提示您输入新创建的密码。
```
# zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

导入数据库架构后， 禁用 log_bin_trust_function_creators 选项。
```
# mysql -uroot -p
password
mysql> set global log_bin_trust_function_creators = 0;
mysql> quit;
```


e. 为Zabbix server配置数据库
编辑配置文件 /etc/zabbix/zabbix_server.conf
```
DBPassword=password
```

f. 启动Zabbix server和agent进程
启动Zabbix server和agent进程，并为它们设置开机自启：
```
# systemctl restart zabbix-server zabbix-agent2 nginx php8.3-fpm
# systemctl enable zabbix-server zabbix-agent2 nginx php8.3-fpm
```



### 4. 字体乱码问题
1. 下载开源字体 `NotoSansCJKjp-Regular.ttf`

2. zabbix 字体配置文件路径 `/usr/share/zabbix/ui/include/defines.inc.php`

```conf
# 字体路径
define('ZBX_FONTPATH',                          realpath('assets/fonts')); // where to search for font (GD > 2.0.18)

# 修改字体配置
define('ZBX_GRAPH_FONT_NAME',           'NotoSansCJKjp-Regular'); // font file name
```

3. 参考配置文件中字体存放路径，并存放字体 `/usr/share/zabbix/assets/fonts`


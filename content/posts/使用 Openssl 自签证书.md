---
date: '2025-02-18T12:06:28+08:00'
draft: false
title: '使用 Openssl 自签证书'
tags: ["SSL", "HTTPS"]
---


**使用 `openssl` 自签证书**

### 1. 生成私钥文件 (private key)

```bash
openssl genrsa -out mykey.key 2048
```

生成一个名为 `mykey.key` 的私钥文件。也可以根据需要调整密钥长度（例如 4096 位）。



### 2. 根据私钥生成证书签名请求文件 (CSR文件)

```bash
openssl req -new -key mykey.key -out myrequest.csr
```

基于 mykey.key 文件生成一个 CSR 文件 myrequest.csr。

```TEXT
在执行过程中，OpenSSL 会提示你输入一些信息，如：
	•	国家代码 (Country Name)
	•	省/市 (State or Province Name)
	•	地区 (Locality Name)
	•	组织 (Organization Name)
	•	组织单位 (Organizational Unit Name)
	•	公共名称 (Common Name) —— 通常填写域名
	•	电子邮件地址 (Email Address)
这些信息会嵌入到证书中，作为身份标识。
```



### 3. 使用私钥对证书申请进行签名从而生成证书文件 (pem文件)

使用私钥对 CSR 进行签名，生成一个自签名证书:

```bash
openssl x509 -req -days 365 -in myrequest.csr -signkey mykey.key -out mycert.crt
```

```TEXT
  	• 	-req 表示输入的是一个证书签名请求文件。
	•	-days 365 指定证书的有效期为 365 天（你可以根据需要调整有效期）。
	•	-in myrequest.csr 指定之前生成的 CSR 文件。
	•	-signkey mykey.key 使用之前生成的私钥进行签名。
	•	-out mycert.crt 指定生成的证书文件名为 mycert.crt。
```



***也可以将这三行命令合并成一个命令，这将不会生成 CSR 文件。***

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout mykey.key -out mycert.crt \
  -subj "/C=US/ST=State/L=City/O=Organization/OU=Unit/CN=example.com"
```

```TEXT
	•	-x509					让 req 命令生成自签名证书而不是证书签名请求（CSR）。
	•	-nodes					表示不对生成的私钥进行加密（不设置密码）。
	•	-days 365				设置证书有效期为 365 天。
	•	-newkey rsa:2048		同时生成一个 2048 位的 RSA 私钥。
	•	-keyout mykey.key		指定生成的私钥文件名为 mykey.key。
	•	-out mycert.crt			指定生成的证书文件名为 mycert.crt。
	•	-subj					用于提供证书中的主题信息，避免在交互模式下逐项输入。
```




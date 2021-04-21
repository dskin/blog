---
title: 使用Let's Encrypt申请配置免费HTTPS证书
date: 2021-04-21 14:45:20
tags: https
---

为了网络连接的安全性，目前大部分网站都已步入了全站 HTTPS 的时代。[Lets Encrypt](https://letsencrypt.org/zh-cn/getting-started/)是一家免费为用户提供数字证书的认证机构，申请使用也较为方便，这里简单记录一下申请部署过程。

<!--more-->

## 下载 ACME 客户端

官方推荐使用[Certbot](https://certbot.eff.org/)，这里我们选择使用更为简单方便的[acme.sh](https://github.com/acmesh-official/acme.sh)。acme.sh 也是实现了 acme 协议的客户端，它也可以帮我们从 lets encrypt 申请免费的证书。

## 配置 acme.sh

`acme.sh`的配置使用十分方便，这里摘取[官方 wiki](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)的步骤做简短记录。

### 安装 acme.sh

```bash
curl  https://get.acme.sh | sh
```

此命令将安装在`home`目录下，创建 bash alias 方便命令调用`alias acme.sh=~/.acme.sh/acme.sh`。

### 生成证书

acme.sh 提供了两种生成证书的方式，分别是`http`验证和`dns`验证，这里记录 http 验证的使用方式。

```bash
acme.sh  --issue  -d mydomain.com -d www.mydomain.com  --webroot  /home/wwwroot/mydomain.com/
```

只需要指定网站所在的根目录，acme.sh 将会自动完成验证，web 服务器若是使用`nginx`的话，可以更方便的执行

```bash
acme.sh --issue  -d mydomain.com   --nginx
```

### 复制并安装证书

默认证书生成位置在安装目录`~/.acme.sh`中，我们可以使用`--install-cert`指定目标位置进行复制安装。nginx 服务器使用示例：

```bash
# nginx
acme.sh --install-cert -d example.com \
--key-file       /path/to/keyfile/in/nginx/key.pem  \
--fullchain-file /path/to/fullchain/nginx/cert.pem \
--reloadcmd     "service nginx force-reload"
```

打开 nginx 配置文件，进行证书配置，我们添加两个`server`块，强制全站 https。

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com;

    ssl_certificate /path/to/fullchain/nginx/cert.pem;
    ssl_certificate_key /path/to/keyfile/in/nginx/key.pem;
}
```

这里需要注意的 nginx 配置中`ssl_certificate`需要使用`--fullchain-file`命令指定生成的证书。

### 更新证书

目前证书会每 60 天自动进行更新

### 更新 acme.sh

目前由于 acme 协议和 letsencrypt CA 都在频繁的更新, 因此 acme.sh 也经常更新以保持同步。我们可以使用以下命令开启自动升级：

```bash
acme.sh  --upgrade  --auto-upgrade
```

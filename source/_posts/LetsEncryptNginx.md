---
title: let's encrypt nginx
date: 2022-01-10
tags:
  - lets encrypt
  - nginx
categories:
  - 运维
---

### 环境

* centos 7

* nginx 1.16

### 下载 letsencrypt 客户端工具

* 如果你已经百度过相关教程，你一定发现有人使用的是 certbot，而有人则从 github 克隆了 letsencrypt 仓库，本着官方优先的原则我选择了克隆 letsencrypt 仓库：
    ```shell
    git clone https://github.com/letsencrypt/letsencrypt
    cd letsencrypt/
    ```

### 申请证书

* 如果你已经百度过相关教程，你一定发现很多帖子即使使用了同一种客户端工具，具体命令却是五花八门，比如有人这样写：

    ```shell
    ./letsencrypt-auto certonly --standalone --email abc@qq.com -d *.crag.top
    ```
    我这样做之后却遇到了报错：
    > Client with the currently selected authenticator does not support any combination of challenges that will satisfy the CA. 
    >
    > You may need to use an authenticator plugin that can do challenges over DNS.
    
    这是因为正在试图让 DNS 服务商自动验证你的域名和服务器，但仅部分 DNS 服务商支持，官方文档给了以下名单：
    > cloudflare，cloudxns，digitalocean，dnsimple，dnsmadeeasy，google，linode，luadns，nsone，ovh，rfc2136，route53
    
    毫无疑问这是专为老外行的方便，国内像我等使用阿里云解析的就省省吧。
    
* 如果你去百度那个错误，你会找到另一条命令：
  ```shell
  ./letsencrypt-auto certonly --preferred-challenges dns --manual --email conlay@qq.com -d *.crag.top -d crag.top --server https://acme-v02.api.letsencrypt.org/directory
  ```
  这条命令执行过程中会两次要求我们添加 DNS 解析记录，尽管照做就好，可以成功的：
  >    Congratulations! Your certificate and chain have been saved at:
  >    /etc/letsencrypt/live/crag.top/fullchain.pem
  >    Your key file has been saved at:
  >    /etc/letsencrypt/live/crag.top/privkey.pem
  >    Your cert will expire on 2020-07-13. To obtain a new or tweaked
  >    version of this certificate in the future, simply run
  >    letsencrypt-auto again. To non-interactively renew *all* of your
  >    certificates, run "letsencrypt-auto renew"

### 配置 nginx

* 设置证书示例：
    ```
    server {
          listen 443 ssl;
          server_name crag.top;
          ssl_certificate     /etc/letsencrypt/live/crag.top/fullchain.pem;
          ssl_certificate_key /etc/letsencrypt/live/crag.top/privkey.pem;
          ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
          ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
          ssl_prefer_server_ciphers on;
          location / {
               .......
          }
    }
    ```
* 将所有80端口的请求重定向到443：
    ```
    server {
            listen       80;
            server_name  _;
            return 301 https://$http_host$request_uri;
    }
    ```

### 证书续期

* 众所周知，letsencrypt 免费证书只有3个月有效期，那么怎么续签呢？
* 网上很多帖子说通配符的域名续签比较麻烦，各种写脚本，但是我经过实践发现：只需要重新执行一次上面的那条 ./letsencrypt-auto certonly XXXX 命令就可以了，并且不需要再次添加DNS解析记录来验证！

------
![福利](/images/骚图/三国杀/SP蔡文姬1.jpg)

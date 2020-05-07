---
title: let's encrypt
date: 2015-12-14 23:35:49
---

[Let's Encrypt][1] 已经公开测试，不需要再提交测试域名表单，直接就能申请  
小项目以后都能用这玩意开 https 不用花钱买证书哦啦啦

照 [文档][2] 做一遍给域名签上证书还挺简单的

![ssl](images/ssl.jpg)

```bash
# 获取项目代码
git clone https://github.com/letsencrypt/letsencrypt
cd letsencrypt

# 安装依赖
./letsencrypt-auto

# 获取证书
./letsencrypt-auto certonly --standalone -d www.example.com -d example.com
```

```bash
# 配置 nginx
server {
  listen 443 ;
  ssl on;
  ssl_certificate_key /etc/letsencrypt/live/youdomain/privkey.pem;
  ssl_certificate /etc/letsencrypt/live/youdomain/fullchain.pem;
}
```

~~需要注意的是 dnspod 等国内服务解析域名有问题~~  
我这里直接切回 domains.google.com 就行了  
使用 standalone 模式需要先停掉默认的 nginx  
文档里提到可以使用 webroot 模式不用停
但我创建验证文件失败了 :(  
默认90天过期，建议 crontab 定时更新


###Update


使用 [acme.sh][3]

- An ACME protocol client written purely in Shell (Unix shell) language.
- Fully ACME protocol implementation.
- Simple, powerful and very easy to use. You only need 3 minutes to learn.
- Bash, dash and sh compatible. 
- Simplest shell script for Let's Encrypt free certificate client.
- Purely written in Shell with no dependencies on python or Let's Encrypt official client.
- Just one script, to issue, renew and install your certificates automatically.

或者 [caddyserver][4]

- Caddy is a web server like Apache, nginx, or lighttpd, but with different goals, features, and advantages.
- The purpose of Caddy is to streamline an authentic web development, deployment, and hosting workflow so that anyone can host their own web sites without requiring special technical knowledge.
- Caddy is also the first and only web server to serve all live sites over HTTPS by default.



[1]: https://letsencrypt.org
[2]: https://letsencrypt.readthedocs.org/en/latest/using.html
[3]: https://github.com/Neilpang/acme.sh
[4]: https://caddyserver.com

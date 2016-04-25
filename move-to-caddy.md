---
title: move to caddy
date: 2016-04-26 06:41:13
---

去年用 Hexo 搭建的日志已经好几个月没更新了，最近休假有点空闲就继续更新吧。  
先从 Github 迁移回自己的 Linode，然后安装一个 Caddyserver[1] 来渲染 markdown

> Caddy is a unique web server with a modern feature set. Think nginx or Apache, but written in Go. With Caddy, you can serve your websites over HTTP/2. It can act as a reverse proxy and load balancer. Front your PHP apps with it. You can even deploy your site with git push. Cool, right?[2]



## Download and install systemd
```bash
wget -O 'caddy.tar.gz' 'https://caddyserver.com/download/build?os=linux&arch=amd64&features=git%2Cipfilter'
tar zxf caddy.tar.gz
mkdir /opt/caddy
mkdir /var/run/caddy/
chown caddy:caddy -R /opt/caddy
chown caddy:caddy -R /var/run/caddy
mv caddy /opt/caddy/
useradd -d /var/run/caddy --system caddy

chmod 644 /etc/systemd/system/caddy.service
systemctl enable caddy.service
systemctl start caddy.service
systemctl status caddy.service
```
<https://github.com/caddyserver/examples/blob/master/systemd%2Fcaddy.service>


## Caddyfile

配置文件相当之简洁，加上 git hook 之后只需要本地写好日志往上 push 即可

```
# /etc/caddy/Caddyfile
localhost:8080
gzip
log /tmp/caddy.access.log

git github.com/username/repo {
  branch caddy
  hook   /webhook
}

markdown / {
  ext      .md .txt
  template templates/blog.html
  template index templates/index.html
}
```


不过 let's encrypt 还不支持 idn，暂时没有替换掉 nginx，

`failed to get certificate: acme: Error 400 - urn:acme:error:malformed - Error creating new authz :: Internationalized domain names (starting with xn--) not yet supported`

[1]: https://caddyserver.com/
[2]: https://blog.gopheracademy.com/caddy-a-look-inside/

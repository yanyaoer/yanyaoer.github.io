---
title: "Cloudflare Gost"
date: 2020-05-07T16:00:01+08:00
---

从清明节开始，稳定运行好几年的 ss 服务器终于阵亡了，所有端口全挂。一直蹭公司的 vpn 查资料也挺到了五一，实在拖延够够的就再另外开了一台机器中转过去迁移数据，不过嘛年纪大了又开始犯懒，企图拯救获得资格认证的机器，通过一番网上冲浪学习到了目前(实测可用)能满足我需求的方案。简单来说就是：cloudflare[后文简称为 cf] + websockets over gost，实际的客户端通过 cdn 代理再接入服务


有几个需要注意的地方：

- gost 启动时绑定的 localhost 不直接对外访问，走了 caddy 的转发，而这一步和 cf 的 ssl 证书配置会造成不停的重定向跳转，需要将 cf 加密模式配置为 flexible

- 然后修改 caddy 的域名配置为 http://domain.com https://domain.com { ... } 阻止 cf 和 caddy 之间的 http -> https

- gost 服务端监听 ws 协议，本地的 gost 客户端转发 wss 协议连接 cf_domain:443

- 需要鉴权的方案使用 socks5+wss://username:password@domain:port

- android 客户端的设置，因为使用了 ws 协议，所以需要将域名写入到插件的配置里，直接用域名变量无法解析

具体配置参考官方文档，**一切浪费的时间都是源于没认真仔细看文档**

- https://github.com/haoel/haoel.github.io
- https://github.com/ginuerzh/gost
- https://github.com/xausky/ShadowsocksGostPlugin



**update 2021-03-01** 由于 shawdowsocks 的 android 客户端升级导致插件不可用，另外部署了 [brook](https://github.com/txthinking/brook) wsserver 给手机使用

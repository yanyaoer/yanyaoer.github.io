---
title: "Self Host Maddy Mail Server"
date: 2020-12-01T20:34:44+08:00
tags: [self-host, mail-server, maddy, degoogle]
---


## maddy
> Composable all-in-one mail server. <https://github.com/foxcpp/maddy>

准备自建系列服务替换掉 google 全家桶，先试了传统的 Postfix, Dovecot, OpenDKIM, OpenSPF, OpenDMARC 套餐，但是本人水平菜，机器配置也不高，折腾两天还没跑起来，正好看到订阅的 changelog 推荐了 maddy [^changelog] 于是搓搓手气试试这个邮局方案，以下是满足个人喜好的优点
- 基于 golang 生态依赖少，方便打镜像
- 验证数据存在 sqlite 里也很轻量
- 没有 web 端，只需要 imap + thunderbird 即可
- 配置文件相对简单


## install

```bash
mkdir -p /data/maddy

# hard link caddy's crt and key to maddy
ln var/lib/caddy/.local/share/caddy/certificates/acme-v02.api.letsencrypt.org-directory/${domain}/${domain}.crt /data/maddy/${domain}.crt
ln var/lib/caddy/.local/share/caddy/certificates/acme-v02.api.letsencrypt.org-directory/${domain}/${domain}.key /data/maddy/${domain}.key


# foxcpp/maddy:v0.4.2
podman run \
  --name maddy \
  -v /data/maddy:/data \
  -p 25:25 \
  -p 143:143 \
  -p 587:587 \
  -p 993:993 \
  -p 465:465 \
  yanyaoer/maddy:master


# add user
podman run --rm -it -v /data/maddy:/data --entrypoint /bin/maddyctl yanyaoer/maddy:master creds create yanyao@mail.yadanhe.com
```

参考文档配置[^setting-up] 完后发送邮件对方可收，本地 thunderbird 接收回复邮件时有问题
- [ ] spf 验证无法通过，暂时禁用待修复
- [x] 接收时报错 "Unexpected EOF when reciving from Steam"，等 0.4.3 发布 [^issues-300]

## workaround
- 使用 master 分支打包镜像 maddy 0.5.0-dev15+ge4ad3bd 到 `yanyaoer/maddy:master` 临时使用
- go 1.15.5 打包镜像时报错 `go build runtime/cgo: invalid flag in go:cgo_ldflag: -Wl,-z,relro,-z,now`，在 builds.sh 里添加参数通过 [^issues-299]
```
diff --git a/build.sh b/build.sh
index a76f6b6..b6e2eb2 100755
--- a/build.sh
+++ b/build.sh
@@ -119,6 +119,7 @@ read_config() {
         export MADDY_SRC=
     fi

+    export CGO_LDFLAGS_ALLOW='-Wl,-z,relro,-z,now'
     export CGO_CFLAGS="-g -O2 -D_FORTIFY_SOURCE=2 $CFLAGS"
     export CGO_CXXFLAGS="-g -O2 -D_FORTIFY_SOURCE=2 $CXXFLAGS"
     export LDFLAGS="-Wl,-z,relro,-z,now $LDFLAGS"
```


[^changelog]: https://changelog.com/news/maddy-a-composable-allinone-mail-server-ljEe
[^setting-up]: https://foxcpp.dev/maddy/tutorials/setting-up/
[^issues-299]: https://github.com/foxcpp/maddy/issues/299#issuecomment-735862091
[^issues-300]: https://github.com/foxcpp/maddy/issues/300


## alternative
以下方案不太合口味，日后有缘再试
- <https://github.com/sovereign/sovereign>
- <https://github.com/ajgon/self-hosted-mailserver/blob/master/docs/nsa-proof-your-e-mail-in-2-hours.md>
- <https://github.com/Mailu/Mailu>
- <https://github.com/mailcow/mailcow-dockerized>
- <https://github.com/tonioo/modoboa>
- <https://github.com/sovereign/sovereign>
- <https://github.com/mail-in-a-box/mailinabox>
- <https://github.com/iredmail/iRedMail>

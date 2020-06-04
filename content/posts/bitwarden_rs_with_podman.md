---
title: "Deploy bitwarden_rs with podman"
date: 2020-06-04T19:05:12+08:00
---

bitwarden_rs[^1] 是个用 rust/rocket 编写的非官方 api 实现，
[dotnet](https://github.com/bitwarden/server) 版本有点儿不太适合手头的部署环境

这里用比较现代的 podman[^2] 来运行 docker 服务


```bash
podman run -d --name bitwarden -v /bw-data/:/data/:Z -e ROCKET_PORT=8080 -p 8080:8080 bitwardenrs/server:latest
podman generate systemd --name bitwarden --files
mv container-bitwarden.service /etc/systemd/system/

systemctl --user enable /etc/systemd/system/container-bitwarden.service
systemctl --user start container-bitwarden.service
```

然后用 nginx 或者 caddy 代理一下 8080 端口即可连接客户端

```bash
brew install bitwarden-cli  # npm install -g @bitwarden/cli
bw config server https://bw.myserver.com
```

[^1]: https://github.com/dani-garcia/bitwarden_rs/wiki/Using-Podman
[^2]: https://podman.io/getting-started/installation.html

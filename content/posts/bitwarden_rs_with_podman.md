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

**update 2021-04-01** build bitwaden_rs from source

```
# install rust
apt install git make gcc libssl-dev pkg-config curl
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

# build bitwarden_rs
git clone https://github.com/dani-garcia/bitwarden_rs/
cd bitwarden_rs/
cargo build --features sqlite --release
cp target/release/bitwarden_rs /usr/local/bin/

# download bw_web_vault from https://github.com/dani-garcia/bw_web_builds/releases
```

edit  /etc/systemd/system/bitwarden.service
```
[Unit]
Description=Bitwarden Server (Rust Edition)
Documentation=https://github.com/dani-garcia/bitwarden_rs
After=network.target

[Service]
EnvironmentFile=/data/bitwarden_rs/config.env
ExecStart=/usr/local/bin/bitwarden_rs
LimitNOFILE=1048576
LimitNPROC=64
PrivateTmp=true
PrivateDevices=true
ProtectHome=true
ProtectSystem=strict
WorkingDirectory=/data/bitwarden_rs/
ReadWriteDirectories=/data/bitwarden_rs/
ReadWriteDirectories=/data/bitwarden_rs/
AmbientCapabilities=CAP_NET_BIND_SERVICE
Restart=always
RestartSec=5
StartLimitBurst=3
StartLimitInterval=60
StandardOutput=null
StandardError=null

[Install]
WantedBy=multi-user.target
```

[^1]: https://github.com/dani-garcia/bitwarden_rs/wiki/Using-Podman
[^2]: https://podman.io/getting-started/installation.html

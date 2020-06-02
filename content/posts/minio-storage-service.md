---
title: "Minio Storage Service"
date: 2020-06-02T09:47:48+08:00
---

[min.io](https://min.io) 来自前 glusterFS 团队的分布式存储项目，
兼容 aws s3 / google cloud storage 接口，支持多磁盘、多节点，伸缩扩容方便，
golang 编写+单执行文件部署，非常适合用 k8s 编排复制来搭建私有对象存储服务

**没有生产环境的使用经验，以下仅用于业余尝鲜** ~~六一节礼物~~

## GNU/Linux 下载安装
```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio -O /usr/local/bin/minio
chmod +x /user/local/bin/minio
```

## 添加用户、组和配置文件
```bash
groupadd --system minio
useradd --system --gid minio --shell /usr/sbin/nologin --comment "Minio file server" minio
mkdir -p /data/minio
chown -R minio:minio /data/minio

# replace minio.service with your own config, eg. User,Group
wget https://raw.githubusercontent.com/minio/minio-service/master/linux-systemd/minio.service -O /etc/systemd/system/minio.service
```

## 配置端口和密钥
```bash
# optional. run `uuidgen` to creates AK/SK

cat <<EOF >> /tmp/minio
MINIO_VOLUMES="/data/minio/"
MINIO_OPTS="--address :9199"
MINIO_ACCESS_KEY=`uuidgen`
MINIO_SECRET_KEY=`uuidgen`

EOF
```

## 启动服务
```bash
systemctl enable minio.service
systemctl start minio.service
```

[客户端 mc](https://docs.min.io/docs/minio-client-complete-guide.html)
使用说明参考官方文档好了，和 s3 命令行也没太多区别，
甚至可以用来做 gcs/s3 代理用

## BONUS
之前断断续续申请了若干次的 oraclecloud 免费版，信用卡扣了钱也没通过简直过分，
直到 2020-06-01 当天终于成功开启 *happy children's day*

[always free plan](https://www.oracle.com/cloud/free/#always-free)
2个虚拟机各有 1/8 ocpu 和 1G 内存，两块磁盘共 100G，#真香警告#

开启机器后默认的安全策略只打开了 22 端口，自行添加 80/443
之后折腾半天还是无法连接，直接添加 iptables 放行即可，
搜索出来的那些直接打开全部端口的谜之操作请谨慎参考。
另外吐槽下这个双层防火墙配制有点费头发，还是 aws 那种操作一次就好...


```bash
iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp -m tcp –-dport 443 -j ACCEPT

iptables-save > /etc/iptables/rules.v4
```


---
title: "Mastodon"
date: 2020-05-15T09:45:08+08:00
tags: [mastodon, decentralized]
---

`Mastodon 长毛象`[^wiki] -- 基于 rubyonrails/reactjs/nodejs 开发的分布式 &
去中心化 twiter clone。利用空闲时间在 aws lightsail 上开了个实例把服务跑了起来

一开始走了些弯路，因为选机房和省钱的缘故，重建了若干次操作系统，最后的选择是
tokyo+cloudflare，没错我又套了 cdn，实在是海外线路到北京联通不稳定

安装步骤没有使用 docker 而是参考文档从源码安装[^ifs]，原因和解决方案如下：

- 机器用 $3.5/mo 512mem 最便宜的那档消费降级，出于 net/io 性能考虑就不使 docker 啦

- 内存问题，`RAILS_ENV=production bundle exec rake mastodon:setup`
    这一步骤执行到 `rails assets:precompile`，
    不管是在 docker 里跑还是直接运行都会报 swap 分区不足，找到两个方案来解决:

```bash
# create swapfile <https://linuxize.com/post/create-a-linux-swap-file/>
$ sudo fallocate -l 2G /swapfile
$ sudo chmod 600 /swapfile
$ sudo mkswap /swapfile
$ sudo swapon /swapfile

# verify active
$ sudo swapon --show
# optional: low value is better for production
$ sudo sysctl vm.swappiness=10
```

---

```bash
# By default the memory limit in Node. js is 512 mb, 
# use command —- max-old-space-size to increase the memory limit
$ NODE_OPTIONS="--max-old-space-size=4096" RAILS_ENV=production bundle exec rails assets:precompile
```

- 直接配置 localhost 发送邮件无效，先用 [sendgrid](https://sendgrid.com/) 免费版顶着
    一天 100 封邮件目前够用

[^wiki]: https://github.com/tootsuite/mastodon
[^ifs]: https://github.com/tootsuite/documentation/blob/master/content/en/admin/install.md

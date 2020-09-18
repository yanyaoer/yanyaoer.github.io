---
title: "Rss2email"
date: 2020-06-23T18:58:30+08:00
---

距离上次从 <https://blogtrottr.com> 迁移 rss 订阅到 feedly 差不多五六年了，
这会网络不可用的状况又变得严重些，干脆还是用自建服务来接收感兴趣的内容，
当然备选方案也不少:

- https://github.com/miniflux/miniflux
- https://github.com/SSilence/selfoss
- https://github.com/FreshRSS/FreshRSS
- https://git.tt-rss.org/fox/tt-rss

出于复古心理顺便减少对应的 app 安装选择问题，又选择了回归 email 方式订阅更新


## 安装配置
rss2email 的安装到简单，参考 [官方文档](https://github.com/rss2email/rss2email)
直接 `apt install rss2email`，默认配置会发送文本邮件，修改 `html-mail=True`，
在我的环境里 smtp over sendgrid 很容易超配额，sendmail 基本发送不出去，
就配上 imap.google 账户自己发给自己啦～

从 feedly 导出 opml 文件然后 `r2e opmlimport` 完事大吉

需要注意的是，在 `r2e run` 的执行过程中对配置文件进行修改非常容易被覆盖 [^issue]，不要同时搞太多操作就好，

**update: 2020-09-18 17:34:49**

修改参数 `force-from = True`，默认会从 feed 里优先提取作者或者发布站点的邮箱，导致 gmail 过滤器失败


## 定时任务
现代 linux 发行版本(比如这里用的 ubuntu 20.04)，默认不带 cronjob 服务，改为 systemd 来管理各种服务，定时任务的写法稍微麻烦了些，先用当前 user 权限来 `$HOME/.config/systemd/user` 写入定时任务配置

定时器 `rss2email.timer`

```bash
[Unit]
Description=rss2email

[Service]
Type=oneshot
ExecStart=/usr/bin/r2e run
```

执行文件 `rss2email.service`

```bash
[Timer]
OnCalendar=00/3:00
Unit=rss2email.service
[Install]
WantedBy=multi-user.target
```

启动服务

```bash
systemctl --user daemon-reload
systemctl --user enable rss2email.timer
systemctl --user start rss2email.timer

systemctl --user list-timers
NEXT                        LEFT          LAST PASSED UNIT            ACTIVATES
Tue 2020-06-23 12:00:00 UTC 1h 32min left n/a  n/a    rss2email.timer rss2email.service
```

## 定制邮件样式

默认的 configparser 参数会将 `#` 设为注释，于是 `rss2email/config.py` 和
`$HOME/.config/rss2email.cfg` 中关于 `#entry {...} #body {...}`
这几行一直显示不出来，临时解决方案把它们合并成一行可以先凑合着

```
css = ...
    .footer {
    background: #c3d9ff;
    border-top: solid 4px #c3d9ff;
    padding: 5px;
    margin-bottom: 0px;
    } #entry { border: solid 4px #c3d9ff; } #body { margin-left: 5px; margin-right: 5px; }
    img { max-width: 90% } /* 避免邮件里展示原图将页面撑得太宽 */

```

[Lines removed in stylesheet and optionally link to external one](https://github.com/rss2email/rss2email/issues/7)

[^issue]: https://github.com/rss2email/rss2email/issues/110

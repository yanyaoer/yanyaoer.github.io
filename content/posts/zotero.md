---
title: "Zotero as personal knowledge management"
date: 2020-07-02T22:55:53+08:00
---

[Zotero](https://www.zotero.org/) 常用于科研的文献管理，由 firefox 浏览器插件发展到现在有了独立的桌面和[移动客户端](https://github.com/mickstar/Zoo-For-Zotero)。

- 本身具备简易的 rss 订阅功能可以用来追踪论文的更新
- 注册在线账户后支持导出资料库给其他同学订阅
- 浏览器插件 [zotero connectors](https://github.com/zotero/zotero-connectors) 则类似各类笔记应用的 web clipper，方便导入各种资料。**注意科学上网加载完整页面后再进行保存**
- 通过插件可以扩展出许多其他功能，比如文件同步、高级档案管理


## quickstart

```
brew cask install zotero
```

这里简单用 [caddy2 搭建 webdav](https://github.com/yanyaoer/caddy2-webdav) 来同步附件，就没用高级的 [zotfile](http://zotfile.com/)

至于自带的笔记功能怎么说呢～ 已经适应不能这种风格了


| add-ons | link |
| --- | --- |
| quicklook | https://github.com/mronkko/ZoteroQuickLook/releases |
| markdown | https://addons.thunderbird.net/zh-cn/thunderbird/addon/markdown-here-xul/ |


## subscription

- https://arxiv.org/list/cs/recent
康奈尔大学提供的免费论文预览平台，主要订阅 cs.SD 音频相关更新

- https://scholar.google.com/
- https://academic.microsoft.com/
谷歌学术、微软学术也可以用来搜索一些关键词并订阅推送

- https://www.storkapp.me/ 提供关键词订阅推送到邮箱

- https://sci-hub.tw/ 实在找不到的下载就靠毛子的共享服务


将码农常用的文档工具 gitbook 转为 pdf

```bash
brew cask install calibre

sudo npm install -g gitbook-cli

gitbook install  # for gitbook plugin

git clone https://github.com/prometheus/docs.git

gitbook pdf
```

ps: gitbook 开源版本已经很久不维护了，可以换到 [honkit](https://github.com/honkit/honkit) fork 的版本


## reference

- https://github.com/qiyuangong/How_to_Search_and_Read_a_Paper
- https://github.com/ying-zhang/ying-zhang.github.io/blob/blog/source/_posts/we-love-paper.md
- http://www.hanlindong.com/2018/zotero-citation-manager/

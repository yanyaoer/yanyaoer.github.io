---
title: "Dash Replacement with tmux"
date: 2020-05-07T18:54:37+08:00
---

通过 tmux 快捷集成替换 dash.app 查询开发文档


```
$ brew install dasht

$ dasht-docsets | tr 'A-Z' 'a-z'
go
javascript
python_3
rust
tornado
```

```
# tmux.conf quick start
bind -n S-up command-prompt -p 'docset:' "splitw -h -fb -l 80 dasht '%%'"

```

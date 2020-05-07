---
title: create CNAME with internationalized domain name
date: 2015-10-04 20:52:56
tags: [idn, cname, gh-pages]
---

前段时间买了个 [idn][1]: yányào.com 闲置了很长时间没动
趁着十一长假无所事事的机会, 把玩了一下 [hexo][2] 挂到 yanyaoer.github.io

然后 CNAME 的时候掉坑了, 看到有人说30分钟生效傻傻等就不提了
实际上这个域名的 CNAME 内容应该用编码后的字符串而不是 yányào.com

<https://github.com/yanyaoer/yanyaoer.github.io/commit/7c0e4e6863904442d368e3ad5c822f8f189bb7fc#diff-adc4bfdb0829dae99e3699393e3fbaa4>

```diff
diff --git a/CNAME b/CNAME
index 6cb647c..92d166c 100644
--- a/CNAME
+++ b/CNAME
@@ -1 +1 @@
-yányào.com
+xn--ynyo-2nad.com
```

[1]: https://en.wikipedia.org/wiki/Internationalized_domain_name
[2]: https://hexo.io

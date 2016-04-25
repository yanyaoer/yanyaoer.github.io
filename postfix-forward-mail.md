---
title: forward email by postfix
date: 2014-10-06 01:25:06
---

```bash
sudo aptitude install postfix
hostname -f


# sudo vim /etc/postfix/main.cf
myhostname = example.com
myorigin = example.com
mydestination = example1.com, example2.com, ...
virtual_alias_maps = hash:/etc/postfix/virtual

# sudo vim /etc/postfix/virtual
@example1.com    name@forward.com
@example2.com    name@forward.com


sudo postmap /etc/postfix/virtual
sudo /etc/init.d/postfix reload
``` 

<https://wiki.debian.org/Postfix#Forward_Emails>

<https://www.linode.com/docs/email/postfix/basic-postfix-email-gateway-on-debian-6-squeeze>

<https://www.debian-administration.org/article/243/Handling_mail_for_multiple_virtual_domains_with_postfix>

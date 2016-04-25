---
title: mitmproxy
date: 2014-08-20 12:05:38
---

[mitmproxy](http://mitmproxy.org) 是个命令行下查看/修改 http 请求的交互式工具


#截图
![screenshoot](http://mitmproxy.org/doc/screenshots/mitmproxy.png)
![screenshoot](http://mitmproxy.org/doc/screenshots/mitmproxy-flowview.png)


#安装

    sudo apt-get install python-dev libffi-dev
    pip install mitmproxy
    

#使用

**ubuntu 上启动 mitmproxy**  
    mitmproxy --host
    
**手机 设置 -> WLAN -> 代理**  
主机名: ubuntu 的 ip  
端口: 8080

然后访问网络就会在 mitmproxy 里看到请求记录(如截图)


#快捷键

j,k 上下移动  
enter 进入  
tab 切换 request/response  


#参考
<http://mitmproxy.org/doc/mitmproxy.html>  
<http://blog.philippheckel.com/2013/07/01/how-to-use-mitmproxy-to-read-and-modify-https-traffic-of-your-phone/>

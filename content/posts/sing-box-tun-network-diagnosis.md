---
title: "sing-box TUN 模式网络慢问题诊断与优化"
date: 2026-01-10T23:00:00+08:00
tags: ["sing-box", "network", "proxy", "macOS"]
---

记录一次 sing-box TUN 模式下网络访问慢的诊断过程

## 问题现象

- 环境：macOS + sing-box TUN 模式
- 现象：能连接公网，但访问 Google 等网站非常慢或完全无法访问
- 国内网站访问正常

## 诊断过程

### 1. 初步测试

```bash
# 测试国内网站 - 正常
curl -w "Total: %{time_total}s\n" -o /dev/null -s https://www.baidu.com
# Total: 0.086s

# 测试 Google - 失败 (SSL 错误)
curl -w "Total: %{time_total}s\n" -o /dev/null -s https://www.google.com
# exit code 60, SSL certificate problem
```

### 2. DNS 污染检测

```bash
# 系统 DNS 返回错误 IP (Meta 的 IP，不是 Google)
dig +short www.google.com
# 31.13.94.36

# sing-box DNS 返回正确 IP
dig +short www.google.com @127.0.0.1
# 142.250.194.164
```

**发现问题 1**：系统 DNS 被 GFW 污染，返回了错误的 IP 地址。

### 3. 检查 TUN 路由配置

```json
// sing-box config.json
"route_exclude_address_set": ["geoip-cn"]
```

**发现问题 2**：`route_exclude_address_set: ["geoip-cn"]` 将中国 IP 排除在 TUN 外，导致：
- 系统 DNS 服务器 (114.114.114.114) 在 geoip-cn 里
- DNS 查询绕过了 TUN，直接走本地网络
- 被 GFW 污染返回错误 IP

### 4. 检查 DNS detour 配置

```json
"servers": [
  {
    "server": "1.1.1.1",
    "tag": "remote",
    "type": "udp",
    "detour": "节点选择"  // selector 类型
  }
]
```

**发现问题 3**：DNS detour 指向 selector（节点选择），可能导致循环依赖：
1. DNS 查询需要走代理
2. 代理需要解析代理服务器域名
3. 解析又需要 DNS...

### 5. 检查 sing-box DNS 服务

```bash
# sing-box DNS 查询超时
dig +short google.com @127.0.0.1
# connection timed out

# 日志显示
# ERROR dns: exchange failed for google.com: dial tcp 1.1.1.1:443: i/o timeout
```

**发现问题 4**：sing-box 进程本身的连接不会被自己的 TUN 捕获（设计特性，防止无限循环）。所以 DNS 查询（即使是 DoH）直接发出，被 GFW 阻断。

### 6. IPv6 问题

```bash
# curl 优先使用 IPv6
curl -v https://www.google.com
# Trying [2607:f8b0:4007:804::2004]:443...
# SSL_ERROR_SYSCALL

# 强制 IPv4 成功
curl -4 https://www.google.com
# 成功
```

**发现问题 5**：代理节点不支持 IPv6，但系统优先使用 IPv6 连接。

## 问题总结

| 问题 | 原因 | 影响 |
|------|------|------|
| TUN MTU 过大 | 设置为 9000 | 可能导致分片 |
| geoip-cn 排除 | DNS 服务器在排除列表 | DNS 查询被污染 |
| DNS detour 循环 | selector 依赖 DNS | DNS 查询超时 |
| sing-box 出站不走 TUN | 设计特性 | DoH 被 GFW 阻断 |
| IPv6 连接失败 | 代理不支持 IPv6 | 连接超时 |

## 修复方案

### 1. 调整 TUN MTU

```json
// 从 9000 改为 1400
"mtu": 1400
```

### 2. 移除 geoip-cn 排除

```json
// 删除这行，让所有流量过 TUN
// "route_exclude_address_set": ["geoip-cn"]
```

国内流量仍会直连，因为 route.rules 中有规则：
```json
{
  "action": "route",
  "outbound": "DIRECT",
  "rule_set": ["geosite-cn", "geoip-cn"]
}
```

### 3. DNS detour 指向具体节点

```json
{
  "tag": "remote",
  "type": "udp",
  "server": "1.1.1.1",
  "detour": "具体代理节点"  // 不是 selector
}
```

### 4. 设置系统 DNS

```bash
sudo networksetup -setdnsservers Wi-Fi 127.0.0.1
```

### 5. 禁用 TUN IPv6

```json
// 移除 IPv6 地址
"address": [
  "172.19.0.1/30"
  // 删除: "2001:0470:f9da:fdfa::1/64"
]
```

## 最终配置要点

```json
{
  "dns": {
    "strategy": "prefer_ipv4",
    "servers": [
      {
        "tag": "remote",
        "type": "udp",
        "server": "1.1.1.1",
        "detour": "具体代理节点"  // 不要用 selector
      },
      {
        "tag": "cn",
        "type": "udp",
        "server": "223.5.5.5"  // 国内 DNS 不需要 detour
      }
    ]
  },
  "inbounds": [{
    "type": "tun",
    "address": ["172.19.0.1/30"],  // 仅 IPv4
    "mtu": 1400,
    "route_exclude_address": [
      "192.168.0.0/16",
      "10.0.0.0/8"
    ]
    // 不要 route_exclude_address_set: ["geoip-cn"]
  }]
}
```

## 优化效果

| 网站 | 优化前 | 优化后 |
|------|--------|--------|
| Google | 超时/SSL错误 | 1.27s |
| GitHub | 超时 | 0.29s |
| Baidu | 0.09s | 0.09s |

## 经验总结

1. **DNS 是网络问题的常见根源**：优先检查 DNS 解析是否正确
2. **理解 TUN 模式的工作原理**：sing-box 自身的出站流量不走 TUN
3. **避免 DNS 循环依赖**：detour 应指向具体节点而非 selector
4. **IPv6 可能是隐藏的坑**：如果代理不支持 IPv6，应禁用
5. **geoip 排除要谨慎**：可能意外排除了 DNS 服务器等关键 IP

---

*使用 Claude Code 辅助诊断*

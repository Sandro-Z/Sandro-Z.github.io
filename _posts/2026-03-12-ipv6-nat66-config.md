---
title: "NAT66配置"
date: 2026-03-12 19:00:00 +0800
excerpt_separator: "<!--more-->"
last_modified_at: 2026-03-12T19:00:00+08:00
categories:
  - Network
tags:
  - Network
  - nat
---

背景：西安交通大学学生住宿区拨号上网默认会下发一个/128的IPv6地址，由于前缀过短，无法划分子网给路由器下的设备使用，因此只能给子网设备分配内网地址，并且通过路由器的NAT66让所有子网设备使用同一个公网v6地址。

# 第一步：开启IPv6内网ip下发
在OpenWRT的Network-Interfaces-Global network options中，确保IPv6 ULA-Prefix存在。可以直接留空用默认值，也可以修改。
![ipv6-ula](/assets/images/image%20copy.png)

> 按照标准，以 fd 开头的 IPv6 地址在电脑/手机上的优先级**低于**IPv4。这意味着即使你配置成功了，你的设备也会优先走 IPv4，导致 IPv6 测试网站不通过。  
> 解决方法：把这里的 fd 改成 dd（例如 `dd12:3456:7890::/48`），伪装成公网 IP 段，强制设备优先使用 IPv6。

**接下来**：我们要配置LAN口下发v6.这里的LAN口包含了路由器的物理LAN口以及WiFi。

1. 在 网络 -> 接口 中，找到 LAN 口，点击 修改 (Edit)。
切换到 高级设置 (Advanced Settings)，确保 IPv6 分配长度 (IPv6 assignment length) 设为 64。
![assignment-length](/assets/images/image%20copy%202.png)
2. 切换到页面下方的 DHCP 服务器 (DHCP Server) -> IPv6 设置 (IPv6 Settings)，按照以下设置：

>> 路由通告服务 (Router Advertisement-Service)：设置为 服务器模式 (server mode)  
>> DHCPv6 服务 (DHCPv6-Service)：设置为 服务器模式 (server mode)  
>> NDP 代理 (NDP-Proxy)：设置为 禁用 (disabled)
![dhcp-settings](/assets/images/image%20copy%203.png)

3. 切换到“IPv6 RA Settings”并将Default Router设置为`forced`。
![default-router](/assets/images/image%20copy%204.png)
>> 对于内网 ULA 前缀，部分设备不会将路由器设为默认网关，必须强制通告.

点击“保存并应用”。

# 第二步：关闭IPv6源地址路由
OpenWrt 默认有一套严格的策略路由：它规定只有源 IP 是运营商下发的公网前缀时，才允许走默认路由上网；如果你是用内网地址（如你的 dd2c），路由器发现源 IP 不合规，就会直接把包丢弃，并向电脑发送 unreachable。 开启 NAT66 必须关掉这个限制。

1. 进入 网络 (Network) -> 接口 (Interfaces) ->你的wan口。
在页面上方，点击Advanced Settings。
2. 找到 IPv6 源路由 (IPv6 source routing) 这个选项。
取消勾选它（把勾去掉）。
3. 点击右下角的 保存并应用 (Save & Apply)。
![disable-source-routing](/assets/images/image%20copy%205.png)

# 第三步：开启NAT66
1. 进入 网络 (Network) -> 防火墙 (Firewall) -> 常规设置 (General Settings)。
2. 在下方的 区域 (Zones) 列表中，找到 wan 区域（通常包含 wan 和 wan6）。
点击 wan 区域右侧的 编辑 (Edit)。
3. 切换到 高级设置 (Advanced Settings) 选项卡。
找到 IPv6 伪装 (IPv6 Masquerading) 或者 强制 NAT66 的勾选项。
勾选它，保存并应用。
![v6masquerading](/assets/images/image%20copy%206.png)

恭喜，完成上述步骤后，子网设备就可以通过这个/128前缀的IPv6访问互联网了！
![success](/assets/images/image%20copy%207.png)

XJTU的IPv6没有50M教育网出口速度限制，直接爽翻天（然而我路由器只有百兆……）
![speedtest](/assets/images/image%20copy%208.png)
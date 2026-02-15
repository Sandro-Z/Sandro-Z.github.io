---
title: "frp无法连接排查"
date: 2026-02-15 10:17:00 +0800
excerpt_separator: "<!--more-->"
last_modified_at: 2026-02-15T10:17:02-05:00
categories:
  - Network
tags:
  - Network
  - frp
---

**起因：** 发现一国内机器连接到日本出口vps的frpc永远无法连接，报错： `login to the server failed:connection write timeout. With loginFailExit enabled, no additional retries will be attempted.`

国内机为4C6G，出口为1C1G，问题发生时资源占用均未爆满。

**排查过程：** 
1. 首先检查出口机器的防火墙是否开启了frps的端口: `firewall-cmd --list-ports|grep 7000`，确认不是防火墙问题

2. 尝试直接连接该端口：`telnet <出口ip> 7000`，显示connected，说明网络连接没有问题

3. 使用tcpdump抓握手包：`tcpdump -ni any host <ip>`，先运行此命令开始抓包，之后启动frpc尝试连接，抓包处可以看到存在发来的数据包，说明不是此处问题

4. 大神请教下，想到可能是网速问题，运行iperf3测速，国内机器运行 `iperf3 -c <ip>`，同时出口运行`iperf3 -s`（注意该机器防火墙需要开通5201端口）。测速结果：两机器直接传输速度0kb/s，frp无法连接问题应该由此产生。

5. 发现虽然远程开了bbr，但是国内的机器并没有开。后续所有服务器均应该开启bbr，可以起到网络加速的作用。
  
  ```
  touch /etc/sysctl.conf
  echo "net.core.default_qdisc=fq" | sudo tee -a /etc/sysctl.conf
  echo "net.ipv4.tcp_congestion_control=bbr" I sudo tee -a /etc/sysctl.conf
  sudo sysctl -p
  ```

  同时，考虑到国内机器所处的网络环境，使用ipv6可以获得更快的连接速度。因此frpc配置中改为使用出口的ipv6进行连接。

**修复结果：** 
使用ipv6测速速度可以达到200M，属正常水平。之后systemd中启动frp，成功连接。

**反思：** 
此次事件初步比较难以排查，因为使用同样的国内机和出口的情况下，几天前仍然连接一切正常，并且更换出口为其他出口的时候连接也一切正常。测速发现问题后，认为应当是买到了及其便宜的“高性价比神机”导致的（见图）。

![“番茄特供”超售性价比神机](/assets/images/151be31f-ea41-4e63-a165-4b944cee75e9.jpg "“番茄特供”超售性价比神机")

这种机器在未超售时（对应前几天正常运行）可用性较强，但是商家低价超售后会有很严重的抢带宽问题，因此应避免选择这种出口。

一般来讲，1C1G的机器价格在5$左右比较合理，超低价机器很可能有坑。
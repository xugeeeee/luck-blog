---
title: 香橙派 AI Pro SSH 远程连接完整指南
date: 2026-01-19 20:00:00
tags:
  - 香橙派
  - SSH
  - Linux
  - 嵌入式开发
  - 问题排查
categories:
  - 技术
cover: 
description: 详细记录 Windows 远程 SSH 连接香橙派 AI Pro 的完整操作流程，包括踩坑经验和问题排查方法。
---

# 🍊 香橙派 AI Pro SSH 远程连接完整指南

> **设备**：香橙派 AI Pro 20T（昇腾310B）  
> **系统**：Ubuntu 22.04 (aarch64)  
> **日期**：2026-01-19

---

## 一、Windows SSH 远程连接香橙派（完整操作）

### 1. 前提条件

- Windows 10/11（自带 OpenSSH 客户端）
- 香橙派已开机并连接网络
- 知道香橙派的 IP 地址

### 2. 获取香橙派 IP 地址

在香橙派终端执行：

```bash
ip addr
# 或
ifconfig
```

找到 `wlan0`（WiFi）或 `eth0`（有线）对应的 `inet` 地址，例如：`192.168.251.220`

### 3. Windows 连接命令

打开 PowerShell 或 CMD，执行：

```powershell
ssh HwHiAiUser@<香橙派IP地址>
```

例如：

```powershell
ssh HwHiAiUser@192.168.251.220
```

### 4. 首次连接确认

首次连接会提示确认主机密钥：

```
The authenticity of host '192.168.251.220 (192.168.251.220)' can't be established.
ED25519 key fingerprint is SHA256:xxxxx
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

输入 `yes` 回车。

### 5. 输入密码

默认密码通常是：`Mind@123`

> ⚠️ 如果密码错误，可尝试：`orangepi` 或查阅官方文档

### 6. 连接成功

看到以下界面表示连接成功：

```
  ___                                    ____   _
 / _ \  _ __  __ _  _ __    __ _   ___  |  _ \ (_)
| | | || '__|/ _` || '_ \  / _` | / _ \ | |_) || |
| |_| || |  | (_| || | | || (_| ||  __/ |  __/ | |
 \___/ |_|   \__,_||_| |_| \__, | \___| |_|    |_|
                           |___/
Welcome to Orange Pi Ai Pro 20T
(base) HwHiAiUser@orangepiaipro-20t:~$
```

---

## 二、本次连接踩过的坑（问题排查记录）

### 问题1：SSH服务被卸载

**现象**：尝试连接时直接被拒绝

**原因**：不小心卸载了 openssh-server

**解决方案**：

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl start ssh
sudo systemctl enable ssh
```

---

### 问题2：apt 源无法访问

**现象**：

```
Could not connect to repo.huaweicloud.com:80 (222.211.219.x)
connect (113: No route to host)
```

**原因**：华为云源服务器不支持 IPv6，而系统的 IPv4 路由有问题

**解决方案**：换成清华源

```bash
sudo sed -i 's|repo.huaweicloud.com|mirrors.tuna.tsinghua.edu.cn|g' /etc/apt/sources.list
sudo apt update
```

---

### 问题3：nano 命令不存在

**现象**：

```
sudo: nano: command not found
```

**原因**：精简系统未预装 nano 编辑器

**解决方案**：使用 `vi` 或 `echo` 命令替代

```bash
# 使用 echo 直接写入
echo "内容" | sudo tee /path/to/file

# 或使用 vi
sudo vi /path/to/file
```

---

### 问题4：缺少 /run/sshd 目录

**现象**：

调试模式 `sudo /usr/sbin/sshd -d` 显示：

```
Missing privilege separation directory: /run/sshd
```

**原因**：重装 openssh-server 后未自动创建权限分离目录

**解决方案**：

```bash
sudo mkdir -p /run/sshd
sudo chmod 755 /run/sshd
sudo systemctl restart ssh
```

**永久解决**（防止重启后消失）：

```bash
echo "d /run/sshd 0755 root root -" | sudo tee /etc/tmpfiles.d/sshd.conf
sudo systemd-tmpfiles --create
```

---

### 问题5：端口22被占用

**现象**：

```
Bind to port 22 on 0.0.0.0 failed: Address already in use
Cannot bind any address.
```

**原因**：多次启动 sshd 导致进程冲突

**解决方案**：

```bash
sudo killall sshd
sudo systemctl start ssh
```

---

### 问题6：连接仍被服务器关闭（最终问题）

**现象**：

```
kex_exchange_identification: Connection closed by remote host
Connection closed by 172.168.50.116 port 22
```

SSH 服务正常运行，配置正确，但连接在密钥交换阶段被关闭。

**原因**：**原 WiFi 网络（172.168.50.x）存在限制**

可能原因：

- 路由器防火墙阻止了端口22
- AP 隔离（客户端之间不能互相访问）
- 网络设备对 SSH 连接有特殊处理

**最终解决方案**：换成手机热点网络

1. 手机开启热点
2. 香橙派和 Windows 都连接手机热点
3. 获取香橙派新 IP：`ip addr | grep wlan0`
4. 使用新 IP 连接：`ssh HwHiAiUser@192.168.251.220`

---

### 问题7：路由优先级冲突导致网络不通

**现象**：

WiFi 已连接，IP 地址正常，但 ping 外网显示：

```
From orangepiaipro-20t (192.168.0.2) icmp_seq=1 Destination Host Unreachable
```

**原因**：usb0 网络接口配置了静态IP和默认路由，且优先级（metric）比 wlan0 更高

```bash
$ ip route
default via 192.168.0.1 dev usb0 proto static metric 100 linkdown  # 优先级高但断开
default via 192.168.251.76 dev wlan0 proto dhcp metric 600         # 实际工作的
```

系统优先通过已断开的 usb0 发包，导致网络不通。

**临时解决方案**：

```bash
# 删除 usb0 的默认路由
sudo ip route del default via 192.168.0.1 dev usb0

# 验证
ping baidu.com
```

**永久解决方案**：

```bash
# 禁用 usb0 静态连接并取消自动连接
sudo nmcli connection down usb0-static
sudo nmcli connection modify usb0-static autoconnect no
```

**诊断命令**：

```bash
# 查看所有网络接口
ip addr

# 查看路由表（注意 metric 值和 linkdown 状态）
ip route

# 查看活跃的网络连接
nmcli connection show --active
```

---

## 三、SSH 调试技巧

### 1. Windows 端调试

```powershell
ssh -v HwHiAiUser@<IP地址>
```

`-v` 参数显示详细连接过程，帮助定位问题。

### 2. 香橙派端调试

```bash
# 停止正常服务
sudo systemctl stop ssh

# 调试模式启动（会显示详细错误）
sudo /usr/sbin/sshd -d
```

然后从 Windows 连接，香橙派终端会显示具体错误原因。

### 3. 查看 SSH 配置

```bash
# 查看生效的配置
sudo sshd -T | grep -i password

# 检查主机密钥
ls -la /etc/ssh/ssh_host_*
```

### 4. 查看系统日志

```bash
sudo journalctl -u ssh -n 50
```

---

## 四、常见问题速查表

| 错误信息 | 可能原因 | 解决方案 |
|----------|----------|----------|
| `Connection refused` | SSH服务未启动 | `sudo systemctl start ssh` |
| `Connection timed out` | 网络不通/IP错误 | 检查IP，`ping`测试 |
| `Connection closed by remote host` | 服务端配置问题/网络限制 | 用 `sshd -d` 调试，或换网络 |
| `Permission denied` | 密码错误 | 确认密码，尝试 `Mind@123` |
| `Host key verification failed` | 主机密钥变化 | 删除 `~/.ssh/known_hosts` 中对应条目 |

---

## 五、推荐的网络连接方式

| 方式 | 优点 | 缺点 |
|------|------|------|
| **手机热点** | 简单可靠，无额外配置 | 消耗手机流量 |
| **USB 直连** | 稳定，不依赖WiFi | 需要配置 USB 网络 |
| **有线网络** | 最稳定 | 需要网线和路由器 |
| **WiFi** | 方便 | 可能有路由器限制 |

---

## 六、连接成功后的常用操作

```bash
# 查看 NPU 状态
npu-smi info

# 查看系统信息
uname -a
cat /etc/os-release

# 查看摄像头设备
ls /dev/video*

# 查看磁盘空间
df -h

# 查看内存
free -h
```

---

> 💡 **经验总结**：遇到诡异的 SSH 问题时，`sshd -d` 调试模式是神器；如果配置都对但就是连不上，考虑换个网络环境测试！

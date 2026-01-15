+++
date = '2026-01-15T09:02:47+08:00'
draft = false
title = 'Test Deploy'
+++
# 🔥 Firewalld 权威参考手册 (Arch Linux 版)

| 属性 | 说明 |
| :--- | :--- |
| **适用系统** | Arch Linux, CentOS, Fedora, RHEL, openSUSE |
| **底层后端** | nftables (新版) / iptables (旧版) |
| **核心工具** | `firewall-cmd` |
| **配置文件路径** | `/etc/firewalld/` (用户配置), `/usr/lib/firewalld/` (默认配置) |
| **文档版本** | v2.0 (终极详细版) |

---

## 📖 1. 核心设计哲学 (必读)

在操作之前，必须理解 Firewalld 的两个核心概念，否则你的配置可能会重启失效，或者逻辑混乱。

### 1.1 运行时 (Runtime) vs 永久 (Permanent)
Firewalld 的配置分两层：
1.  **Runtime (内存中)**: 修改立刻生效，但**重启后丢失**。用于测试规则。
2.  **Permanent (硬盘中)**: 修改写入配置文件，**重载 (Reload) 后才生效**。用于持久化。

> **💡 最佳实践**: 永远使用 `--permanent` 参数，然后立即运行 `--reload`。除非你只是想临时测试一下怕把自己锁在外面。

### 1.2 区域 (Zones)
Firewalld 将网络划分为不同的“信任等级”。网卡 (Interface) 必须绑定在某个区域里。

| 区域 (Zone) | 信任度 | 默认行为 | 典型应用场景 |
| :--- | :--- | :--- | :--- |
| **drop** | 0% (黑洞) | 丢弃所有入站包，**不给任何回应**。 | 遭受攻击时 / 极高安全环境。 |
| **block** | 10% (拒绝) | 拒绝所有入站包，但会回复 `icmp-prohibited` (告诉对方被拒了)。 | 调试网络连通性时。 |
| **public** | 30% (默认) | 仅允许显式放行的端口 (如 ssh, dhcpv6-client)。 | 公共 Wi-Fi / 默认对外网卡。 |
| **external** | 40% | 类似 public，但默认开启 **IP 伪装 (Masquerade)**。 | 路由器 WAN 口 / 软路由出口。 |
| **home** | 70% | 信任大多设备，开放 mdns, ssh, samba。 | 家庭局域网。 |
| **work** | 60% | 类似 home，但限制更严。 | 公司内网。 |
| **trusted** | 100% | **允许所有流量** (无防火墙限制)。 | VPN 虚拟网卡 (tun0) / 容器内部通信。 |

---

## 🛠 2. 服务生命周期管理

```bash
# 启动防火墙
sudo systemctl start firewalld

# 设置开机自启
sudo systemctl enable firewalld

# 停止防火墙 (调试连通性第一步)
sudo systemctl stop firewalld

# 彻底禁用 (防止开机自启)
sudo systemctl disable firewalld

# 检查运行状态 (Running / Not running)
sudo firewall-cmd --state

# 重新加载配置 (修改 Permanent 后必须执行！)
sudo firewall-cmd --reload
```
## 🔍 3. 信息查看与审计 (Information)
```bash
在修改之前，先学会怎么“看”。
# 1. 查看默认区域的所有规则 (最常用)
sudo firewall-cmd --list-all

# 2. 查看所有区域的详细规则 (信息量极大)
sudo firewall-cmd --list-all-zones

# 3. 查看当前活跃的区域 (查看网卡绑定在哪)
sudo firewall-cmd --get-active-zones
# 输出示例:
# public
#   interfaces: wlan0 eth0
# trusted
#   interfaces: tun0

# 4. 查看系统预定义了哪些服务 (如 ssh, http, docker-registry 等)
sudo firewall-cmd --get-services
```
## ⚙️ 4. 端口与服务管理 (常规操作)

**场景**: 开放 Web 服务、远程桌面、文件共享。 _(注: 以下命令默认省略 `sudo`，实际使用请加上)_
### 4.1 基于服务的管理 (推荐)
不需要记端口号，直接用服务名。
```bash
# 永久允许 HTTP (80)
firewall-cmd --permanent --add-service=http

# 永久允许 HTTPS (443)
firewall-cmd --permanent --add-service=https

# 永久允许 Samba 文件共享
firewall-cmd --permanent --add-service=samba

# 永久允许 KDE Connect (手机传文件)
firewall-cmd --permanent --add-service=kdeconnect

# 移除服务 (关闭)
firewall-cmd --permanent --remove-service=http
```
### 4.2 基于端口的管理 (更灵活)
当服务名不存在，或者使用了非标端口时使用。
```bash
# 开放 TCP 8080 端口
firewall-cmd --permanent --add-port=8080/tcp

# 开放 UDP 53 端口 (DNS)
firewall-cmd --permanent --add-port=53/udp

# 开放一段端口范围 (如 VNC Server)
firewall-cmd --permanent --add-port=5900-5910/tcp

# 移除端口
firewall-cmd --permanent --remove-port=8080/tcp
```
## 🛡 5. 富规则 (Rich Rules) —— 高级安全控制
这是 Firewalld 最强大的地方。可以实现：**“只允许张三的 IP 访问我的 SSH，其他人滚蛋”。**
语法格式: `rule family="ipv4" source address="源IP" ... action`
### 场景 A: 限制 SSH 访问 (白名单)
只允许家里台式机 (192.168.1.50) 连接笔记本的 SSH。
```bash
# 1. 添加富规则：允许 192.168.1.50 访问 SSH
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.50" service name="ssh" accept'

# 2. 移除通用的 SSH 服务 (关键！否则上面那条没意义，因为通用规则允许了所有人)
firewall-cmd --permanent --remove-service=ssh

# 3. 重载
firewall-cmd --reload
```
### 场景 B: 封禁恶意 IP (黑名单)
发现 IP `123.45.67.89` 一直在扫描你的端口。
```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="123.45.67.89" drop'
```
### 场景 C: 记录日志 (审计)
记录所有来自 `192.168.1.0/24` 网段的 SSH 尝试，每分钟最多记录 3 次。
```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="ssh" log prefix="SSH_ATTEMPT: " level="info" limit value="3/m" accept'
```
## 🌐 6. 网卡与网络区域调整

**场景**: 你插了一个 USB 网卡，或者新建了 Sing-box 的虚拟网卡，需要调整它们所属的区域。
### 6.1 将网卡移动到特定区域
比如把 Sing-box 的 `tun0` 设为信任。
```bash
# 语法: --zone=目标区域 --change-interface=网卡名
firewall-cmd --permanent --zone=trusted --change-interface=tun0
```
### 6.2 修改默认区域
新插入的网卡默认会进入 public 区域。如果你希望默认更安全或更宽松：
```bash
# 将默认区域设为 home
firewall-cmd --set-default-zone=home
```

---
## 🔄 7. NAT 与 端口转发 (软路由/容器必备)

**场景**: Docker 容器上不了网，或者想把外网的 80 端口转发到内部虚拟机的 8080。
### 7.1 开启 IP 伪装 (Masquerade)
**这是 Docker/WSL/虚拟机 能够通过宿主机上网的前提。**
```bash
# 检查是否开启
firewall-cmd --zone=public --query-masquerade

# 开启 (针对 public 区域)
firewall-cmd --permanent --zone=public --add-masquerade
```
### 7.2 端口转发 (Port Forwarding)
将访问本机 `80` 端口的流量，转发到 `8080` 端口。
```bash
# 语法: --add-forward-port=port=源端口:proto=协议:toport=目标端口
firewall-cmd --permanent --zone=public --add-forward-port=port=80:proto=tcp:toport=8080
```
## 🚨 8. 紧急救灾模式 (Panic Mode)

**场景**: 你发现服务器被黑客入侵了，正在疯狂传数据，你想立刻切断一切连接（物理拔网线的软件版）。
```bash
# 开启恐慌模式 (切断所有网络连接，SSH 也会断！)
sudo firewall-cmd --panic-on

# 关闭恐慌模式 (恢复正常)
sudo firewall-cmd --panic-off

# 查询是否处于恐慌模式
sudo firewall-cmd --query-panic
```
## 💡 9. 调试技巧：为什么我的规则不生效？
1. **有没有 Reload？** 修改 `--permanent` 后必须 `firewall-cmd --reload`。
2. **网卡在哪个区域？** 用 `firewall-cmd --get-active-zones` 确认你的流量是从哪个网卡进来的，以及那个网卡属于哪个 Zone。如果在 `public` 区加了规则，但网卡在 `home` 区，规则是不会生效的。
3. **冲突检查** Rich Rules 的优先级很高。如果你同时设置了 `drop` 和 `accept`，需要仔细检查逻辑顺序。
4. **Docker 的干扰** Docker 会直接操作 iptables，有时会绕过 Firewalld。如果发现 Docker 端口关不掉，这是 Docker 的特性（需修改 Docker 启动参数）。
---

## 📝 附录： Sing-box 专属配置集

当前的 **Arch Linux + Sing-box (TUN)** 环境，需要保持的配置：
```bash
# 1. 允许 tun0 流量回流 (防止代理断网)
firewall-cmd --permanent --zone=trusted --add-interface=tun0

# 2. 开启伪装 (支持虚拟化网络)
firewall-cmd --permanent --zone=public --add-masquerade

# 3. 开放 KDE Connect (方便手机传文件)
firewall-cmd --permanent --zone=public --add-service=kdeconnect

# 4. 生效
firewall-cmd --reload
```
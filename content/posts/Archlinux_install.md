---
title: "Arch Linux 安装指南：从系统构建到图形环境"
date: 2026-01-15T13:00:00+08:00
draft: false
description: "Arch Linux 安装教程，包含从联网、分区到 KDE 桌面环境配置"
tags: ["Linux", "Arch Linux", "KDE", "Installation"]
---

# 🛠 Arch Linux 安装指南

欢迎来到 Arch Linux 的世界。本手册将带你完成从基础系统联网、分区到安装图形界面（GUI）及常用软件的全过程。

---

## 1. 环境准备与联网

在启动后的 Live 环境中，首先需要确保网络通畅。

```bash
# 1. 如果是无线网络，使用 iwd 联网
iwctl                                      # 进入交互式联网工具
# 在 iwctl 界面中依次执行：
# station wlan0 scan                       # 扫描网络
# station wlan0 get-networks               # 列出可用网络
# station wlan0 connect <SSID>             # 连接 WiFi
# quit                                     # 退出工具

# 2. 检查网络连通性
ping -c 4 google.com                       # 发送 4 个 ICMP 包测试网络

# 3. 更新系统时间（防止证书校验失败）
timedatectl set-ntp true                   # 开启网络时间同步
```

---

## 2. 磁盘分区与格式化

我们将采用主流的 **UEFI + GPT** 结构。



### 2.1 分区规划
使用 `cfdisk` 工具进行可视化操作：
```bash
cfdisk /dev/nvme0n1                        # 打开磁盘分区工具
# 选择 Label 类型为: gpt
# 分区 1: 512M -> 类型选择 EFI System
# 分区 2: 剩余空间 -> 类型选择 Linux filesystem
# 点击 Write 写入更改，然后 Quit 退出
```

### 2.2 格式化与挂载
```bash
# 格式化分区
mkfs.fat -F 32 /dev/nvme0n1p1              # EFI 分区
mkfs.ext4 /dev/nvme0n1p2                   # 根分区

# 挂载分区
mount /dev/nvme0n1p2 /mnt                  # 先挂载根目录
mkdir -p /mnt/boot                         # 创建 EFI 挂载点
mount /dev/nvme0n1p1 /mnt/boot             # 挂载 EFI 分区
```

---

## 3. 系统安装与基础配置

### 3.1 安装核心包
```bash
# 优化镜像源（中国区）
reflector --country China --protocol https --sort rate --save /etc/pacman.d/mirrorlist

# 安装基础系统
pacstrap /mnt base linux linux-firmware base-devel networkmanager vim
```

### 3.2 系统初始化
```bash
# 生成挂载表
genfstab -U /mnt >> /mnt/etc/fstab          

# 进入新系统 (Chroot)
arch-chroot /mnt                            

# 设置时区与本地化
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
# 编辑 /etc/locale.gen 取消 en_US.UTF-8 注释，然后运行：
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# 用户与权限
echo "my-arch" > /etc/hostname              # 设置主机名
passwd                                     # 设置 root 密码
useradd -m -G wheel myuser                 # 创建普通用户
passwd myuser
# 使用 visudo 取消 %wheel ALL=(ALL:ALL) ALL 的注释
```

### 3.3 引导程序 (GRUB)

```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## 🖥 4. 图形桌面环境安装 (KDE Plasma)

完成基础系统后，我们需要安装显示驱动和桌面环境。



```bash
# 1. 安装显卡驱动（根据你的显卡选择）
# 英特尔: pacman -S mesa lib32-mesa vulkan-intel
# AMD: pacman -S mesa lib32-mesa xf86-video-amdgpu
# 英伟达: pacman -S nvidia nvidia-utils
pacman -S mesa                             # 通用开源驱动

# 2. 安装 Xorg 和 字体
pacman -S xorg ttf-dejavu wqy-microhei      # 包含中文字体防止乱码

# 3. 安装 KDE Plasma 核心组件
pacman -S plasma-desktop sddm konsole dolphin 
# 解释：
# plasma-desktop: 桌面核心
# sddm: 登录管理器 (显示管理器)
# konsole: 终端
# dolphin: 文件管理器

# 4. 启用服务（开机进入图形界面）
systemctl enable sddm
systemctl enable NetworkManager
```

---

## 🚀 5. 推荐软件清单

系统装好后，建议安装以下生产力工具。

### 5.1 基础必装
| 软件名 | 用途 | 安装命令 |
| :--- | :--- | :--- |
| **Google Chrome** | 网页浏览器 | `yay -S google-chrome` |
| **VS Code** | 代码编辑器 | `sudo pacman -S code` |
| **Git** | 版本控制 | `sudo pacman -S git` |
| **Fastfetch** | 系统信息展示 | `sudo pacman -S fastfetch` |

### 5.2 系统维护
* **Yay**: AUR 助手（必装）。
* **Timeshift**: 系统快照（防止滚挂）。
* **Btop**: 实时资源监视器。

### 5.3 生产力与多媒体
* **Typora**: Markdown 编辑器。
* **VLC**: 全能播放器。
* **Telegram Desktop**: 通讯工具。
* **YesPlayMusic**: 高颜值网易云客户端。

---

## 6. 完成安装

```bash
exit                                       # 退出 chroot
umount -R /mnt                             # 卸载分区
reboot                                     # 重启进入新世界
```

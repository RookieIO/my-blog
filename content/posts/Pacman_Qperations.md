+++
date = '2026-01-15T11:57:29+08:00'
draft = false
title = 'Pacman_Qperations'
+++
📦 Arch Linux 包管理器 (Pacman) 权威运维手册
属性	详细说明
核心程序	pacman
配置文件	/etc/pacman.conf
镜像列表	/etc/pacman.d/mirrorlist
缓存路径	/var/cache/pacman/pkg/
文档版本	v3.0 (进阶详细版)
🛑 1. 安全红线 (The "Do Nots")
在操作前，请刻在脑子里，否则极易导致系统崩溃（滚挂）。

❌ 禁忌一：禁止部分更新 (Partial Upgrades)
错误操作: pacman -Sy firefox (只同步数据库，然后只安装 Firefox)
后果: 你的数据库是最新的，但系统里的核心库（如 glibc）是旧的。新 Firefox 调用新库时，因为找不到而崩溃。
正确做法: 永远同步并更新整个系统。
sudo pacman -Syu firefox
❌ 禁忌二：强制忽略依赖
错误操作: pacman -Rdd 软件名
后果: 强行删除了被其他软件依赖的包，导致其他软件打不开。
正确做法: 除非你非常清楚你在修 bug，否则只用 -Rs 或 -Rns。
⬇️ 2. 安装与系统更新 (Install & Update)
主参数: -S (Sync)

2.1 核心更新流程
这是你每天开机后应该执行的第一件事。

# 标准全量更新 (同步数据库 + 更新软件)
sudo pacman -Syu

# 强制刷新数据库 (仅在切换镜像源后使用，平时别用，伤服务器)
sudo pacman -Syyu
2.2 软件安装技巧
# 1. 安装单个或多个软件
sudo pacman -S vim git htop

# 2. 安装包组 (Package Group)
# 例如安装 KDE 全家桶，它会问你具体要选哪些，回车默认全选
sudo pacman -S plasma

# 3. 智能安装 (脚本常用)
# 作用：如果软件已经是最新版，跳过；如果没装或有更新，才执行。
sudo pacman -S --needed base-devel git
2.3 降级安装 (救命用)
当新版本有 Bug，你想回退到旧版本时。

# 安装本地缓存中的旧包 (路径通常在 /var/cache/pacman/pkg/)
sudo pacman -U /var/cache/pacman/pkg/package-old-version.pkg.tar.zst

# 或者安装网上下载的包
sudo pacman -U [https://archive.archlinux.org/.../package.pkg.tar.zst](https://archive.archlinux.org/.../package.pkg.tar.zst)
🗑️ 3. 卸载与清理 (Remove)
主参数: -R (Remove)

3.1 标准卸载流程
请养成使用 -Rns 的习惯，保持系统洁癖。

# 🟢 推荐：删除软件 + 没用的依赖 + 全局配置文件
sudo pacman -Rns gimp

# 🟡 普通：删除软件 + 没用的依赖 (保留配置文件)
sudo pacman -Rs gimp

# 🔴 不推荐：仅删除软件本身 (依赖和配置全留着，制造垃圾)
sudo pacman -R gimp
3.2 彻底清理 (Spring Cleaning)
Arch 不会自动删除旧包，缓存会无限膨胀。

# 1. 清理“孤儿包” (曾经作为依赖被安装，现在没人要了)
# 解释：pacman -Qtdq 列出孤儿名，pacman -Rns 删除它们
sudo pacman -Rns $(pacman -Qtdq)

# 2. 清理缓存 (释放硬盘空间)
# 删除所有旧版本的包，只保留当前安装的那个版本
sudo pacman -Sc

# 3. 清空缓存 (极端情况)
# 删除缓存文件夹里的所有东西 (警告：删了就没法降级了)
sudo pacman -Scc
🔍 4. 查询与搜索 (Query)
Pacman 的数据库分两部分：本地 (Local) 和 远程 (Sync)。

4.1 查远程 (搜软件)
# 搜索软件 (支持模糊搜索)
pacman -Ss 关键词  
# 例: pacman -Ss browser

# 查看远程软件的详细信息 (依赖、大小、构建日期)
pacman -Si 软件名
4.2 查本地 (已安装)
# 搜索已安装的软件
pacman -Qs 关键词

# 查看已安装软件的详细信息
pacman -Qi 软件名

# 列出软件的所有文件路径 (它到底装哪去了？)
pacman -Ql 软件名

# 🔥 法医鉴定 (Who owns this file?)
# 查某个文件属于哪个软件包 (排错神器)
pacman -Qo /usr/bin/vim
# 输出: /usr/bin/vim is owned by vim 9.0.xxxx
4.3 依赖树分析
# 查看它依赖谁
pactree 软件名

# 查看谁依赖它 (反向查询，卸载前看看有没有风险)
pactree -r 软件名
🛠️ 5. 故障排除 (Troubleshooting)
5.1 数据库锁定 (Failed to init transaction)
场景: 更新中途断电或强制关闭终端，再次运行提示 db.lck exists。
解决:

sudo rm /var/lib/pacman/db.lck
5.2 签名信任错误 (PGP Signature unknown trust)
场景: 很久没更新，提示签名无效或损坏。
原因: Arch 的密钥环 (Keyring) 更新非常频繁，你的本地密钥过期了。
解决:

# 1. 先单独更新密钥环
sudo pacman -S archlinux-keyring

# 2. 再更新系统
sudo pacman -Syu
5.3 文件冲突 (Exists in filesystem)
场景: 提示 /usr/bin/test exists in filesystem。
原因: 这个文件是你手动复制进去的，不属于 pacman 数据库，pacman 不敢覆盖。
解决:

# 强制覆盖该文件
sudo pacman -Syu --overwrite /usr/bin/test

# 或者强制覆盖所有冲突 (慎用)
sudo pacman -Syu --overwrite "*"
⚙️ 6. 性能优化 (pacman.conf)
建议编辑 /etc/pacman.conf 开启以下现代功能。

Ini, TOML

# sudo vim /etc/pacman.conf

[options]
# 🌈 开启彩色输出
Color

# 🍬 开启吃豆人进度条彩蛋
ILoveCandy

# 🚀 开启并行下载 (核心优化！)
# 默认是单线程，改成 5 后下载速度快 5 倍
ParallelDownloads = 5

# 🚫 忽略升级 (锁版本)
# 如果你不想更新内核或显卡驱动，填在这里
IgnorePkg = linux linux-headers
🆚 附录：Pacman vs Yay (AUR助手)
以下是两者的对应关系.

动作	官方命令 (Pacman)	AUR 助手 (Yay)	区别
系统更新	sudo pacman -Syu	yay	Yay 会同时检查 AUR 更新
安装软件	sudo pacman -S name	yay -S name	Yay 能装 AUR 里的软件
搜索	pacman -Ss name	yay -Ss name	Yay 同时搜官方库+AUR
清理孤儿	sudo pacman -Rns $(...)	yay -Yc	Yay 命令更短
打印统计	无	yay -Ps	查看安装数量和占用空间
📝 极简速查表 (Cheat Sheet)
更新系统: yay
装软件: sudo pacman -S xxx
删软件: sudo pacman -Rns xxx
清理垃圾: sudo pacman -Sc
这文件是谁的?: pacman -Qo 文件路径

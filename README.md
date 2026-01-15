# 🚀 My Tech Blog (Source Code)

![Hugo](https://img.shields.io/badge/Generator-Hugo-pink?style=flat-square&logo=hugo)
![Deploy](https://img.shields.io/badge/Deploy-Cloudflare%20Pages-orange?style=flat-square&logo=cloudflare)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)

> 这是我的个人技术博客源码仓库。
> 采用 **Hugo (Static Site Generator)** 构建，托管于 **GitHub**，并通过 **Cloudflare Pages** 实现自动 CI/CD 部署。

---

## 🏗 Architecture (架构说明)

Workflow 流程如下：

```mermaid
graph LR
    A[Arch Linux / Local] -->|hugo new & git push| B(GitHub Repository)
    B -->|Trigger Webhook| C{Cloudflare Pages}
    C -->|Build & Minify| D[Global CDN]
    D -->|Serve| E[User Browser]
Core: Hugo (Extended Version)Theme: PaperModHosting: Cloudflare Pages (自动监听 main 分支变动)🛠️ Prerequisite (环境准备)如果你更换了电脑，需要先安装以下基础环境：1. 安装 Hugo (Arch Linux)Bashsudo pacman -S hugo git
2. 克隆仓库 (含子模块)因为使用了 PaperMod 主题，必须加上 --recursive 参数：Bashgit clone --recursive git@github.com:RookieIO/my-blog.git
cd my-blog
⚡ Quick Start (极速上手)1. 写作 (Writing)在项目根目录下执行：Bash# 不需要加 content/ 前缀
hugo new posts/你的文件名.md
2. 状态变更 (Crucial Step!)⚠️ 重要：新建的文章默认是 draft: true。打开生成的 .md 文件，修改头部信息（Front Matter）：YAML---
title: "这里写文章标题"
date: 2026-01-15T...
draft: false  <-- 必须改为 false 才能上线！！
---
3. 本地预览 (Preview)开启本地服务器，实时查看效果：Bashhugo server -D
访问：http://localhost:1313🚀 Deployment (发布上线)无需手动构建，只需将源码推送到 GitHub。Bash# 1. 提交变更
git add .
git commit -m "feat: new post 'firewalld guide'"

# 2. 推送 (触发 Cloudflare 自动构建)
git push
构建耗时约 30s - 60s，完成后自动更新 CDN。🎨 Theme & Config (主题配置)配置文件: hugo.toml主题目录: themes/PaperMod (不要直接修改这里的代码，尽量在 assets/ 或 layouts/ 中覆盖)添加菜单: 修改 hugo.toml 中的 [[menu.main]] 字段。❓ Troubleshooting (故障排查)问题现象可能原因解决方案本地预览看不到文章文章仍处于 Draft 状态检查 .md 头部 draft: false，或运行 hugo server -D推送后线上未更新浏览器/CDN 缓存强制刷新 (Ctrl+F5) 或等待 1 分钟后尝试Git Push 报错权限/网络问题检查 SSH Key 或 Token，或使用 git config credential.helper store样式错乱子模块未拉取运行 git submodule update --init --recursive📝 Common Commands Cheat SheetBash# 新建文章
hugo new posts/mypost.md

# 本地预览
hugo server

# 强制拉取主题更新
git submodule update --remote --merge
Last Updated: 2026-01-15 by RookieIO
---


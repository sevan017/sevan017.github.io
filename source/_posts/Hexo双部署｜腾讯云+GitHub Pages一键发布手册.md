---
title: Hexo双部署｜腾讯云+GitHub Pages一键发布手册
date: 2026-07-18
tags: [Hexo,Butterfly,博客部署,腾讯云,GitHubPages]
categories: [博客搭建]
permalink: hexo-dual-deploy-cvm-githubpages/
copyright_url: https://sevan017.github.io/hexo-dual-deploy-cvm-githubpages/
---

---

**功能目标**：本地写完笔记，执行一条命令，**同时自动发布到腾讯云独立域名博客 \+ GitHub Pages镜像博客**

**运行环境**：Windows本地Hexo \+ 腾讯云Ubuntu22\.04

**编辑器规范**：全程使用vim，不使用nano

**已有基础**：GitHub Pages已长期正常部署，仅新增腾讯云服务器双部署

**密钥适配**：本地为RSA密钥 `id_rsa.pub`，无需重新生成密钥

---

## 一、腾讯云服务器一次性部署配置

### 1\. 创建Git裸仓库（接收Hexo推送）

用于接收本地Hexo静态文件，搭配钩子自动同步至网站根目录

```bash
# 创建裸仓库目录并初始化
mkdir -p /home/ubuntu/hexo.git
cd /home/ubuntu/hexo.git
git init --bare

# 创建博客网站根目录
sudo mkdir -p /var/www/blog
sudo chown -R ubuntu:ubuntu /var/www/blog
sudo chmod -R 755 /var/www/blog
```

### 2\. 配置自动同步钩子（核心自动更新功能）

作用：本地推送代码后，服务器自动同步文件到网站目录，无需手动操作

```bash
# 编辑钩子文件
vim /home/ubuntu/hexo.git/hooks/post-receive
```

按 `i` 进入编辑，写入以下脚本：

```bash
#!/bin/bash
GIT_WORK_TREE=/var/www/blog git checkout -f
```

按`Esc`，输入 `:wq` 保存退出。

**关键必做**：给钩子添加可执行权限（无权限会直接忽略钩子，无法自动更新）

```bash
chmod +x /home/ubuntu/hexo.git/hooks/post-receive
```

### 3\. Nginx完整HTTPS配置

```bash
sudo vim /etc/nginx/sites-available/welcomeme.conf
```

清空原有内容，粘贴完整配置：

```nginx
server {
  server_name welcomeme.cn www.welcomeme.cn;
  root /var/www/blog;
  index index.html index.htm;

  # 解决Hexo分页、标签、归档404问题
  location / {
      try_files $uri $uri/ =404;
  }

  # 静态资源缓存加速
  location ~* \.(css|js|png|jpg|jpeg|gif|svg|ico|woff2)$ {
      expires 30d;
      add_header Cache-Control "public, no-transform";
  }

  # 禁止访问隐藏文件
  location ~ /\. {
      deny all;
  }

  listen 443 ssl; # managed by Certbot
  ssl_certificate /etc/letsencrypt/live/welcomeme.cn/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/welcomeme.cn/privkey.pem; # managed by Certbot
  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    listen 80;
    server_name welcomeme.cn www.welcomeme.cn;
    # 全站HTTP强制跳转HTTPS
    if ($host = www.welcomeme.cn) {
        return 301 https://$host$request_uri;
    }
    if ($host = welcomeme.cn) {
        return 301 https://$host$request_uri;
    }
    return 404;
}
```

`Esc` → `:wq` 保存退出，校验并重载Nginx：

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 二、本地Hexo项目核心配置

**重要区分**：双部署配置仅修改 **`_config.yml`****项目根目录 **，**`_config.butterfly.yml`****无需修改主题配置 **

### 1\. 安装部署依赖插件

```powershell
npm install hexo-deployer-git --save
```

### 2\. 最终正确双部署配置（修复多仓库语法）

删除原有错误单仓库写法，替换为以下完整配置（禁止单独写branch字段）

```yaml
# 站点域名配置
url: https://welcomeme.cn
root: /

# 双部署：同时推送 GitHub Pages + 腾讯云服务器
deploy:
  type: git
  repo:
    # 原有GitHub镜像站（无需改动）
    github: git@github.com:sevan017/sevan017.github.io.git,gh-pages
    # 新增腾讯云服务器部署
    cvm: ubuntu@124.220.133.56:/home/ubuntu/hexo.git,master
```

**YAML规范**：缩进统一2个空格，禁止Tab，仓库与逗号、分支无空格

### 3\. SSH免密配置（仅配置腾讯云，GitHub跳过）

本地已有RSA密钥可正常推送GitHub，**严禁执行ssh\-keygen重新生成密钥**，避免覆盖旧密钥导致GitHub部署失效

PowerShell执行，推送本地已有公钥至腾讯云：

```powershell
type $env:USERPROFILE\.ssh\id_rsa.pub | ssh ubuntu@124.220.133.56 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

测试免密登录（无需输入密码即为成功）：

```powershell
ssh ubuntu@124.220.133.56
```

免密失效修复（服务器端执行）：

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

## 三、日常写笔记 \+ 一键双发布流程

### 1\. 新建博客文章

```powershell
hexo new post "文章标题"
```

编辑路径：`source/_posts/文章标题.md`，编写笔记后保存

### 2\. 本地预览（必做，规避部署错误）

```powershell
hexo s
```

浏览器访问 `http://localhost:4000` 确认样式、链接、内容正常，`Ctrl+C` 关闭服务

### 3\. 一键同步发布双平台

```powershell
hexo clean && hexo g -d
```

**执行逻辑**：

- 自动推送静态文件到GitHub `gh-pages` 分支，更新镜像站

- 自动推送至腾讯云裸仓库，触发钩子**自动同步**至网站目录

**访问地址**

- 腾讯云主站：`https://welcomeme.cn`

- GitHub镜像站：`https://sevan017.github.io/sevan017.github.io`

---

## 四、核心报错解决方案（实测落地修复）

### 问题1：推送成功但腾讯云不更新，提示 `hooks/post-receive hook was ignored`

**根因**：钩子无执行权限，Git直接忽略自动同步脚本

**修复命令**：

```bash
# 添加钩子执行权限
chmod +x /home/ubuntu/hexo.git/hooks/post-receive

# 手动同步一次已推送的文件
cd /home/ubuntu/hexo.git
GIT_WORK_TREE=/var/www/blog git checkout -f
```

### 问题2：同步报错 `Permission denied` 无法删除/创建文件

**根因**：网站目录文件归属为www\-data，ubuntu用户无读写删除权限

**永久修复方案**：

```bash
# 更改目录归属为ubuntu（适配Git钩子运行用户）
sudo chown -R ubuntu:ubuntu /var/www/blog
sudo chmod -R 755 /var/www/blog

# 重新执行同步
cd /home/ubuntu/hexo.git
GIT_WORK_TREE=/var/www/blog git checkout -f
```

### 问题3：博客分页、标签页、归档页404

确认Nginx配置包含以下规则：

```nginx
location / {
    try_files $uri $uri/ =404;
}
```

### 问题4：浏览器一直显示旧页面

- 服务端校验：`curl https://welcomeme.cn` 查看真实源码

- 客户端：无痕窗口访问 / `Ctrl+Shift+R` 强制刷新

---

## 五、应急兜底方案

钩子异常自动同步失效时，手动上传静态文件：

```powershell
# 替换为自己的Hexo项目路径
scp -r D:\你的Hexo项目路径\public\* ubuntu@124.220.133.56:/var/www/blog
```

---

## 附录：Vim常用快捷键

|操作功能|指令|
|---|---|
|进入编辑模式|`i`|
|退出编辑模式|`Esc`|
|保存并退出|`:wq`|
|不保存强制退出|`:q!`|

> （注：部分内容可能由 AI 生成）

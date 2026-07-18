# Hexo\+Butterfly\+GitHub 从零搭建个人博客（单仓库双分支\+源码永久备份）

## 前言

本文为**全网最干净的 Windows 从零搭建 Hexo 博客实录**，解决绝大多数新手踩坑问题：

- ✅ 仅使用 **一个 GitHub 仓库**（无需多仓库）

- ✅ **master 分支**：存放完整博客源码（永久备份）

- ✅ **gh\-pages 分支**：自动托管静态网页（对外访问博客）

- ✅ Butterfly 主题采用 **Git 子模块** 安装（不嵌套仓库、可一键更新）

- ✅ 彻底解决 Windows `cp` 命令报错、LF/CRLF 换行警告、仓库嵌套冲突、推送被拒等问题

环境：Windows 10/11 \+ Git \+ Node\.js \+ Hexo \+ Butterfly

## 一、基础环境安装（必备）

### 1\. 安装 Git

官网下载安装：[https://git\-scm\.com/downloads](https://git-scm.com/downloads)

安装全程默认下一步，安装完成后终端验证：

```bash
git --version
```

### 2\. 安装 Node\.js

安装 LTS 长期稳定版：[https://nodejs\.org/](https://nodejs.org/)

务必勾选自动配置环境变量，验证：

```bash
node -v
npm -v
```

### 3\. 配置 Git 全局账号

替换为自己的 GitHub 用户名和注册邮箱：

```bash
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub注册邮箱"
```

### 4\. 配置 GitHub SSH 密钥（核心）

生成密钥：

```bash
ssh-keygen -t ed25519 -C "你的GitHub注册邮箱"
```

连续三次回车（不设置密码），查看并复制公钥：

```bash
cat ~/.ssh/id_ed25519.pub
```

GitHub 头像 → Settings → SSH and GPG keys → New SSH key，粘贴公钥保存。

验证连通性：

```bash
ssh -T git@github.com
```

出现欢迎字样即配置成功。

## 二、初始化本地 Hexo 博客

### 1\. 创建博客目录并初始化

```bash
# 创建并进入博客文件夹
mkdir hexo-blog
cd hexo-blog

# 全局安装 Hexo 脚手架
npm install -g hexo-cli

# 初始化 Hexo 项目
hexo init

# 安装项目依赖
npm install
```

### 2\. 本地测试博客

```bash
hexo s
```

浏览器访问 `http://localhost:4000`，正常打开默认页面即为成功，`Ctrl+C` 关闭服务。

## 三、SSH 方式安装 Butterfly 主题（子模块版）

重点：采用 **Git 子模块** 安装，彻底解决嵌套仓库报错，支持后续一键更新主题。

### 1\. 下载主题

```bash
cd themes
git clone git@github.com:jerryc127/hexo-theme-butterfly.git butterfly
cd ..
```

### 2\. 启用主题

打开博客根目录 `_config.yml`，修改主题配置：

```yaml
theme: butterfly
```

### 3\. 分离主题配置文件（关键）

Windows 无 `cp` 命令，使用 PowerShell 命令复制：

```powershell
Copy-Item themes/butterfly/_config.yml ./_config.butterfly.yml
```

后续所有主题美化配置，仅修改 `_config.butterfly.yml`，升级主题不覆盖自定义配置。

### 4\. 安装主题依赖

```bash
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

### 5\. 预览美化后页面

```bash
hexo clean && hexo s
```

## 四、GitHub 仓库配置（单仓库双分支）

### 1\. 创建专属仓库

新建仓库，命名必须严格为：`你的用户名.github.io`

选择 Public，**不要勾选初始化 README**，直接创建。

### 2\. 安装 Hexo 部署插件

```bash
npm install hexo-deployer-git --save
```

### 3\. 修改部署配置

根目录 `_config.yml` 底部修改部署信息（适配单仓库双分支）：

```yaml
deploy:
  type: git
  repo: git@github.com:你的用户名/你的用户名.github.io.git
  branch: gh-pages  # 网页静态文件部署到 gh-pages 分支

```

## 五、本地 Git 初始化 \& 源码备份配置

### 1\. 初始化本地仓库

```bash
git init
git remote add origin git@github.com:你的用户名/你的用户名.github.io.git
```

### 2\. 配置 \.gitignore（过滤无用文件）

根目录新建 `.gitignore`，粘贴完整规则：

```gitignore
# Hexo 缓存与生成文件
public/
.deploy_git/
db.json
*.log

# 项目依赖（体积巨大，无需备份）
node_modules/

# 编辑器临时文件
.vscode/
.idea/
*.swp
*.swo
.DS_Store
Thumbs.db

# 本地草稿
source/_drafts/

```

### 3\. 修复主题嵌套仓库问题（核心踩坑解决）

删除错误的主题缓存，改用标准子模块管理：

```powershell
# 强制清除错误缓存
git rm --cached -f themes/butterfly
# 删除本地旧主题文件夹
rmdir /s /q themes\butterfly
# 新增 butterfly 子模块
git submodule add git@github.com:jerryc127/hexo-theme-butterfly.git themes/butterfly

```

### 4\. 消除 Windows 换行符警告

```bash
git config --global core.autocrlf true
```

### 5\. 首次提交源码并强制推送 master

远程仓库存在默认文件，需强制覆盖（仅首次执行）：

```bash
git add .
git commit -m "初始化Hexo博客，Butterfly子模块配置完成"
git push -f origin master
```

✅ 此时 **master 分支已完整备份所有博客源码**

## 六、部署博客网站（上线生效）

### 1\. 一键部署静态页面

```bash
hexo clean && hexo g && hexo d
```

执行完成后，静态网页自动推送至 **gh\-pages 分支**

### 2\. 开启 GitHub Pages

仓库 → Settings → Pages：

- Source：Deploy from a branch

- Branch：**gh\-pages**

- Folder：/ \(root\)

保存等待1分钟，通过 `你的用户名.github.io` 访问博客。

## 七、日常写作 \& 备份固定流程

### 1\. 新建文章

```bash
hexo new "文章标题"
```

文章路径：`source/_posts/标题.md`

### 2\. 写完更新全套流程

```bash
# 1. 本地预览检查
hexo clean && hexo s

# 2. 部署上线网站
hexo clean && hexo g && hexo d

# 3. 备份源码到 master 分支
git add .
git commit -m "更新：新增XX文章"
git push origin master

```

## 八、换电脑恢复完整博客（子模块专属）

新电脑克隆完整博客（必须执行子模块初始化）：

```bash
git clone git@github.com:你的用户名/你的用户名.github.io.git
cd 你的用户名.github.io
git submodule update --init --recursive
npm install
hexo s

```

## 九、Butterfly 主题更新方法

```bash
cd themes/butterfly
git pull
cd ../../
git add .
git commit -m "更新Butterfly主题至最新版本"
git push origin master

```

## 十、常见问题汇总

- **cp 命令不存在**：Windows 改用 PowerShell `Copy-Item` 命令

- **LF/CRLF 警告**：执行 `git config --global core.autocrlf true` 永久解决

- **仓库嵌套报错**：使用 git 子模块安装主题，不直接克隆嵌套仓库

- **推送 master 被拒**：首次使用 `git push -f origin master` 强制覆盖即可

- **换电脑主题为空**：克隆后必须执行子模块初始化命令

## 总结

最终实现完美闭环：**单仓库双分支隔离**

- **master**：源码备份、版本管理、主题配置、文章存储

- **gh\-pages**：纯静态网页、对外访问、自动部署

全程无冗余文件、无报错、可长期稳定维护！

> （注：部分内容可能由 AI 生成）

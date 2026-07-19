README.md

✨ Sevan's Blog | 个人技术博客**基于 Hexo + Butterfly 主题搭建的轻量化个人技术博客 | GitHub Pages 全自动托管**

### 📊 仓库状态

<img src="https:/img.shields.io/github/repo-size/sevan017sevan017.github.io?color=95e1d3&amp;style=flat-square" alt="Repo Size"/>



### 🛠️ 技术栈

### 🌐 在线访问

### 📖 项目简介

本仓库是 **Sevan 个人技术博客** 源码仓库，采用当下热门的 **Hexo 静态博客框架 + Butterfly 高颜值主题** 搭建。

博客依托 GitHub Pages 免费静态托管，无需服务器、零运维、访问稳定，主要用于记录技术学习笔记、实战踩坑总结、日常技术分享与个人成长记录。

项目采用 Git 子模块管理主题文件，结构整洁，升级主题、自定义配置、文章迁移便捷高效。

### 📁 仓库目录结构

```bash
sevan017.github.io
├── .github/            # GitHub 工作流配置
├── scaffolds/          # Hexo 文章模板文件
├── source/             # 博客源码核心目录（文章、页面、资源）
├── themes/             # 博客主题目录(Butterfly 子模块)
├── _config.yml         # Hexo 全局配置文件
├── _config.butterfly.yml # Butterfly主题自定义配置
├── _config.landscape.yml # 备用主题配置
├── .gitmodules         # Git子模块配置(主题管理)
├── .gitignore          # Git忽略配置
├── package.json        # 项目依赖配置
└── package-lock.json   # 依赖版本锁定
```

### 💻 本地运行部署

#### 1. 环境依赖

- Node.js >= 14.x
- Git

#### 2. 克隆项目（含主题子模块）

```bash
# 克隆仓库并拉取子模块主题
git clone --recursive https://github.com/sevan017/sevan017.github.io.git
cd sevan017.github.io

# 安装依赖
npm install
```

#### 3. 本地启动预览

```bash
# 本地热更新启动
hexo s
```

访问：`http://localhost:4000`

#### 4. 静态打包 & 部署

```bash
# 生成静态文件
hexo g

# 推送源码自动触发GitHub Pages部署
git add .
git commit -m "update: 更新博客内容"
git push
```

### ✨ 核心特性

- **高颜值界面**：基于 Butterfly 主题，界面简洁美观、适配移动端
- **轻量化极速访问**：纯静态页面，加载速度快，无后端依赖
- **规范模块化**：子模块管理主题，源码与主题分离，便于维护升级
- **免费稳定托管**：依托 GitHub Pages，全球CDN加速、永久免费
- **易于拓展**：支持自定义样式、插件、标签、分类、相册等功能

### 📝 开源许可

本项目基于 **MIT License** 开源，欢迎 Star、Fork，禁止商用。

**⭐ Star me！欢迎交流学习，持续更新技术干货** <img src="https:/img.shields.iobadge/Keep-Coding-ff7eb9?style=flat-square" alt="Coding"/>


---
title: "Zsh 和 Oh My Zsh 安装教程"
date: 2025-01-17T10:00:00+08:00
lastmod: 2025-01-17T10:00:00+08:00
draft: false
categories: ["开发环境"]
tags: ["Zsh", "Shell", "终端"]
---

# Zsh 和 Oh My Zsh 安装教程

## 1. 安装 Zsh

### 对于 Debian/Ubuntu 系统：
```bash
sudo apt update
sudo apt install zsh
```

### 对于 CentOS/RHEL 系统：
```bash
sudo yum install zsh
```

### 对于 Fedora 系统：
```bash
sudo dnf install zsh
```

安装完成后，使用以下命令检查 `zsh` 是否安装成功：
```bash
zsh --version
```

## 2. 将默认 Shell 设置为 Zsh

安装完成后，可以将默认的 Shell 改为 Zsh。使用以下命令：
```bash
chsh -s $(which zsh)
```

然后退出当前会话并重新登录，或者使用以下命令立即切换到 Zsh：
```bash
zsh
```

## 3. 安装 Oh My Zsh

Oh My Zsh 是一个流行的 Zsh 配置管理工具，提供许多插件、主题和功能。可以使用 `curl` 或 `wget` 来安装。

### 使用 curl 安装：
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 使用 wget 安装：
```bash
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

安装完成后，脚本会自动配置 Zsh 以使用 Oh My Zsh。

## 4. 安装插件

**Oh My Zsh** 支持许多插件，你可以选择安装一些常用插件，增强命令行体验。

### 安装 `zsh-autosuggestions` 插件：
这个插件可以在你输入命令时自动给出建议，提升效率。
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
```

### 安装 `zsh-syntax-highlighting` 插件：
该插件提供命令行语法高亮，帮助区分有效命令和错误命令。
```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
```

### 启用插件：
编辑 `~/.zshrc` 文件，启用上述插件。
```bash
nano ~/.zshrc
```
在 `plugins=(...)` 行中添加插件：
```bash
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

保存并退出编辑器后，使用以下命令使更改生效：
```bash
source ~/.zshrc
```

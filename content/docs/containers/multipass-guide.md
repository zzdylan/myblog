---
title: "Multipass 安装和使用指南"
date: 2024-09-14T10:00:00+08:00
lastmod: 2024-09-14T10:00:00+08:00
draft: false
categories: ["容器与云原生"]
tags: ["Multipass", "虚拟化", "VM"]
---

# Multipass 安装和使用指南

本指南将带您通过安装 Multipass，设置虚拟机，以及如何管理它们以进行开发或测试使用。

## 1. 安装 Multipass

### macOS
您可以使用 Homebrew 在 macOS 上安装 Multipass：

```bash
brew install --cask multipass
```

### Linux
对于支持 `snapd` 的 Ubuntu 或其他 Linux 发行版：

```bash
sudo snap install multipass
```

### Windows
对于 Windows，您可以从官方 Multipass 网站下载安装程序：
- [下载 Multipass for Windows](https://multipass.run/download/windows)

安装完成后，使用以下命令验证版本：

```bash
multipass --version
```

## 2. 启动虚拟机 (VM)

安装完成后，您可以启动一个新的虚拟机 (VM)。以下命令启动名为 `my-vm` 的虚拟机，并分配 4GB 内存和 40GB 磁盘空间：

```bash
multipass launch --name my-vm --memory 4G --disk 40G
```

### 注意：
如果看到有关 `--mem` 已弃用的警告，请将 `--mem` 替换为 `--memory`。

## 3. 检查运行中的虚拟机

要查看您创建的虚拟机列表及其状态，请使用：

```bash
multipass list
```

示例输出：
```bash
Name                    State             IPv4             Image
my-vm                   Running           192.168.64.2     Ubuntu 20.04 LTS
```

## 4. 访问虚拟机

要通过 shell 访问正在运行的虚拟机，可以使用 `multipass shell` 命令，后跟虚拟机的名称：

```bash
multipass shell my-vm
```

您将进入该虚拟机的终端会话。

## 5. 删除虚拟机

当不再需要某个虚拟机时，您可以将其删除。首先，停止虚拟机：

```bash
multipass stop my-vm
```

然后删除它：

```bash
multipass delete my-vm
```

如果您想彻底删除之前删除的虚拟机并清理磁盘空间：

```bash
multipass purge
```

## 6. 虚拟机与主机的网络互通

默认情况下，Multipass 虚拟机使用 **NAT** 网络。这使得虚拟机可以访问互联网和外部网络，但主机不能直接通过 `127.0.0.1` 访问虚拟机内的服务。

### 在虚拟机中访问主机的服务
如果您需要从虚拟机访问主机上运行的服务（如代理或 Web 服务器），应使用 **主机的实际 IP 地址** 而不是 `127.0.0.1`。

- 获取主机的 IP 地址：
  ```bash
  ifconfig
  ```
  或在 Windows 上：
  ```bash
  ipconfig
  ```

- 知道主机 IP 后，您可以在虚拟机中配置服务指向主机 IP 和相应的端口。

### 使用 SSH 端口转发
另一种从虚拟机访问主机服务的方法是使用 SSH 端口转发。以下是将虚拟机的 8080 端口转发到主机的命令示例：

```bash
ssh -L 8080:localhost:8080 multipass@<vm-ip-address>
```

现在，您可以在虚拟机中的 `127.0.0.1:8080` 上访问主机服务。

## 7. 桥接网络（可选）

如果您希望虚拟机出现在与主机相同的网络中（即具有自己的本地 IP 地址），您可以通过桥接网络启动虚拟机：

```bash
multipass launch --network en0 --name my-vm
```

将 `en0` 替换为适当的网络接口（使用 `ifconfig` 或 `networksetup -listallhardwareports` 查找正确的接口）。

## 结论

Multipass 是一个强大且易于使用的工具，可以快速创建和管理虚拟机。无论您是在本地开发还是在隔离环境中测试软件，它都能简化这一过程，并且可以很好地跨操作系统集成。通过本指南，您现在掌握了安装、管理和网络配置 Multipass 虚拟机的基础知识。

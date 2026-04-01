---
title: "Mac M1 上安装 MicroK8s 指南"
date: 2024-09-13T15:30:00+08:00
lastmod: 2024-09-13T15:30:00+08:00
draft: false
categories: ["容器与云原生"]
tags: ["MicroK8s", "Mac", "K8s", "M1"]
---

# Mac M1 上安装 MicroK8s 指南

此指南提供了在 Mac M1 芯片上安装 MicroK8s 的步骤。

## 1. 安装 Multipass

MicroK8s 在 macOS 上运行时需要虚拟化技术，它依赖 **Multipass** 来创建虚拟机。首先，你需要安装 Multipass。

- 打开终端，运行以下命令安装 Multipass：
  ```bash
  brew install --cask multipass
  ```

## 2. 安装 MicroK8s

Multipass 安装完毕后，可以使用它来运行 MicroK8s。现在可以在 macOS 上安装 MicroK8s。

新增文件cloud-init.yml
```yaml
#cloud-config

# 设置 apt 的国内镜像源
apt:
  preserve_sources_list: true
  sources:
    tsinghua:
      source: "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ $(lsb_release -cs) main restricted universe multiverse"
      keyid: "3B4FE6ACC0B21F32"

# 配置 snap 的中国镜像源
write_files:
  - path: /etc/systemd/system/snapd.service.d/override.conf
    content: |
      [Service]
      Environment="SNAPD_STORE_URL=https://mirrors.tuna.tsinghua.edu.cn/snap/"
    permissions: '0644'

runcmd:
  # 重新加载 systemd 并重启 snapd
  - sudo systemctl daemon-reload
  - sudo systemctl restart snapd
  - sudo snap install microk8s --classic
  
  # 启动 MicroK8s 并启用必要的插件
  - /snap/bin/microk8s.start
  - /snap/bin/microk8s.status --wait-ready
  - /snap/bin/microk8s.enable dns storage ingress
  - /snap/bin/microk8s.status --wait-ready

```

- 运行以下命令从 Snapcraft 安装 MicroK8s：
  ```bash
  multipass launch --name microk8s-vm --memory 4G --disk 40G --cpus 2 --cloud-init cloud-init.yml
  ```

这条命令会启动一个虚拟机，并在其中自动安装 MicroK8s。

## 3. 连接到 MicroK8s 虚拟机

安装完成后，你可以连接到运行 MicroK8s 的虚拟机来管理 Kubernetes 集群。

- 使用以下命令进入虚拟机：
  ```bash
  multipass shell microk8s-vm
  ```

进入虚拟机后，你可以通过 `microk8s` 命令来管理 Kubernetes。

## 4. 验证 MicroK8s 安装

进入虚拟机后，你可以使用以下命令检查 MicroK8s 是否正确安装并运行：
```bash
microk8s status --wait-ready
```

如果输出显示 "MicroK8s is running"，则表示安装成功。

## 5. 安装 kubectl 命令行工具

安装完 MicroK8s 后，你可以通过 `kubectl` 命令来管理集群。MicroK8s 自带 `kubectl`，你可以直接在虚拟机中使用：

```bash
microk8s kubectl get nodes
```

这将显示 MicroK8s 集群中运行的节点信息。

## 6. 启用 MicroK8s 功能（如 Dashboard）

你可以启用不同的附加组件，如 Dashboard 或 DNS 服务：

```bash
microk8s enable dashboard dns
```

这将启用 Kubernetes Dashboard 和 DNS 插件。

## 7. 退出虚拟机

当你完成工作后，可以使用以下命令退出虚拟机：
```bash
exit
```

## 注意事项

- M1 芯片由于架构的原因，在运行虚拟机时，可能需要确保 Multipass 使用 ARM 架构的虚拟机镜像。
- 安装成功后，你就可以在 macOS 上运行 Kubernetes 集群了！

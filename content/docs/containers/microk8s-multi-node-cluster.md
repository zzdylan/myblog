---
title: "MicroK8s 多节点集群部署与测试指南"
date: 2024-10-14T08:48:00+08:00
lastmod: 2025-01-20T11:07:23+08:00
draft: false
categories: ["容器与云原生"]
tags: ["MicroK8s", "K8s", "集群", "Kubernetes"]
---

# MicroK8s 多节点集群部署与测试指南

本文记录了从创建虚拟机、安装 MicroK8s、替换镜像文件、创建 Kubernetes 集群、部署 Nginx 副本到一系列测试操作的完整步骤。

## 1. 创建虚拟机

使用 Multipass 创建虚拟机，用于安装 MicroK8s。

```bash
multipass launch --name microk8s-vm --cpus 2 --mem 4G --disk 20G
multipass launch --name microk8s-vm2 --cpus 2 --mem 4G --disk 20G
```

- `microk8s-vm` 和 `microk8s-vm2` 是虚拟机的名称。
- 为每台虚拟机指定了 2 个 CPU，4GB 内存，20GB 磁盘。

## 2. 安装 MicroK8s

进入虚拟机并安装 MicroK8s。

```bash
multipass shell microk8s-vm
sudo apt update && sudo apt upgrade -y
sudo snap install microk8s --classic --channel=1.31/stable
```

- 更新系统并安装 MicroK8s。
- 安装完成后，MicroK8s 会自动启动。

## 3. 替换镜像源

为了提高下载速度，可以将 MicroK8s 的镜像源替换为国内镜像源。

```bash
sudo nano /var/snap/microk8s/current/args/containerd-template.toml
```

修改完成后，重启 MicroK8s 使更改生效：

```bash
sudo microk8s stop
sudo microk8s start
```

启动不起来的话，用一下命令查看下报错:
```bash
sudo journalctl -u snap.microk8s.daemon-containerd -e
```

有些镜像可以预先pull指定一下
```bash
sudo microk8s ctr image pull registry.cn-hangzhou.aliyuncs.com/xxx/cni:v3.25.1 && sudo microk8s ctr image tag registry.cn-hangzhou.aliyuncs.com/xxx/cni:v3.25.1 docker.io/calico/cni:v3.25.1
sudo microk8s ctr image pull registry.cn-hangzhou.aliyuncs.com/xxx/coredns:1.10.1 && sudo microk8s ctr image tag registry.cn-hangzhou.aliyuncs.com/xxx/coredns:1.10.1 docker.io/coredns/coredns:1.10.1
sudo microk8s ctr image pull registry.cn-hangzhou.aliyuncs.com/xxx/node:v3.25.1 && sudo microk8s ctr image tag registry.cn-hangzhou.aliyuncs.com/xxx/node:v3.25.1 docker.io/calico/node:v3.25.1
sudo microk8s ctr image pull registry.cn-hangzhou.aliyuncs.com/xxx/kube-controllers:v3.25.1 && sudo microk8s ctr image tag registry.cn-hangzhou.aliyuncs.com/xxx/kube-controllers:v3.25.1 docker.io/calico/kube-controllers:v3.25.1
```

## 4. 创建多节点集群

在主节点（`microk8s-vm`）上生成加入令牌：

```bash
microk8s add-node
```

进入辅助节点（`microk8s-vm2`）并加入集群：

```bash
microk8s join <主节点IP>:25000/<token>
```

例如：

```bash
microk8s join 192.168.64.13:25000/33e6aed76f87f6f536dc764b94d6401f/29023eed9b34
```

完成后可以在主节点上查看集群状态：

```bash
microk8s kubectl get nodes
```

## 5. 部署 Nginx 副本

在主节点上创建 Nginx 部署：

```bash
microk8s kubectl create deployment nginx --image=nginx
```

扩展 Nginx 副本数量到 5：

```bash
microk8s kubectl scale deployment nginx --replicas=5
```

查看 Pod 状态：

```bash
microk8s kubectl get pods -o wide
```

可以看到 5 个 Nginx 副本分布在两个节点上。

## 6. 暴露 Nginx 服务

通过 NodePort 暴露 Nginx 服务：

```bash
microk8s kubectl expose deployment nginx --port=80 --type=NodePort
```

查看服务的 NodePort：

```bash
microk8s kubectl get svc nginx
```

例如：`80:31499/TCP` 表示可以通过集群节点的 IP 和端口 `31499` 访问 Nginx 服务。

## 7. 测试负载均衡

在宿主机上通过 NodePort 访问 Nginx 服务：

```bash
curl http://<主节点IP>:<NodePort>
```

例如：

```bash
curl http://192.168.64.13:31499
```

通过多次访问，可以观察到请求被分发到不同的 Pod，验证负载均衡效果。

## 8. 模拟节点故障

停止辅助节点：

```bash
multipass stop microk8s-vm2
```

查看 Pod 的重新调度情况：

```bash
microk8s kubectl get pods -o wide
```

可以看到原来在辅助节点上的 Pod 被重新调度到主节点上，验证 Kubernetes 的高可用性和故障自愈能力。

## 9. 清理环境

删除 Nginx 部署和服务：

```bash
microk8s kubectl delete deployment nginx
microk8s kubectl delete service nginx
```

## 总结

通过以上步骤，我们实现了在 Multipass 虚拟机中部署 MicroK8s、创建多节点 Kubernetes 集群、部署 Nginx 副本以及测试负载均衡和故障自愈的完整过程。这些操作可以帮助我们更好地理解 Kubernetes 的高可用性和集群管理功能。

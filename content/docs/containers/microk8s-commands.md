---
title: "MicroK8s 常用命令"
date: 2024-09-30T10:00:00+08:00
lastmod: 2024-09-30T10:00:00+08:00
draft: false
categories: ["容器与云原生"]
tags: ["MicroK8s", "K8s", "Kubernetes", "命令"]
---

# MicroK8s 常用命令

## 基本命令

- **查看节点状态**：
  ```bash
  microk8s status
  ```

- **启用/禁用插件（如 DNS、Dashboard 等）**：
  - 启用插件：
    ```bash
    microk8s enable <plugin-name>
    ```
    例如：
    ```bash
    microk8s enable dns dashboard
    ```
  
  - 禁用插件：
    ```bash
    microk8s disable <plugin-name>
    ```

- **启动和停止 MicroK8s**：
  - 启动：
    ```bash
    microk8s start
    ```
  
  - 停止：
    ```bash
    microk8s stop
    ```

## 镜像
- **镜像列表**：
  ```bash
  microk8s ctr images list
  ```

- **拉取镜像**：
  ```bash
   microk8s ctr image pull <镜像名称>
  ```
  
- **删除镜像**：
  ```bash
  microk8s ctr image rm <镜像名称>
  ```

## 管理 Pod 和服务

- **查看所有命名空间下的 Pod**：
  ```bash
  microk8s kubectl get pods --all-namespaces
  ```

- **查看服务**：
  ```bash
  microk8s kubectl get svc
  ```

- **创建资源（如部署、服务等）**：
  ```bash
  microk8s kubectl apply -f <filename>.yaml
  ```

- **查看 Pod 的详细信息**：
  ```bash
  microk8s kubectl describe pod <pod-name>
  ```

- **删除资源**：
  ```bash
  microk8s kubectl delete -f <filename>.yaml
  ```
  或者
  ```bash
  microk8s kubectl delete pod <pod-name>
  ```

- **查看命名空间中的所有资源**：
  ```bash
  microk8s kubectl get all -n <namespace>
  ```

- **获取某个 Pod 的日志**：
  ```bash
  microk8s kubectl logs <pod-name>
  ```

- **进入正在运行的 Pod**（类似 `docker exec`）：
  ```bash
  microk8s kubectl exec -it <pod-name> -- /bin/bash
  ```

## 高级集群操作

- **查看节点信息**：
  ```bash
  microk8s kubectl get nodes
  ```

- **加入新节点**：
  在第一个节点上生成令牌：
  ```bash
  microk8s add-node
  ```
  然后使用生成的命令在新节点上加入：
  ```bash
  microk8s join <主节点IP>:25000/<token>
  ```

- **删除节点**：
  ```bash
  microk8s leave
  ```

- **查看集群节点**：
  ```bash
  microk8s kubectl get nodes
  ```

## 系统相关

- **检查 MicroK8s 服务是否运行**：
  ```bash
  sudo systemctl status snap.microk8s.daemon-kubelet
  ```

- **重启 MicroK8s**：
  ```bash
  sudo systemctl restart snap.microk8s.daemon-kubelet
  ```

- **查看所有插件**：
  ```bash
  microk8s status --wait-ready
  ```

## 5. Dashboard 和监控

- **启用 Kubernetes Dashboard**：
  ```bash
  microk8s enable dashboard
  ```

- **访问 Dashboard**：
  启动 Kubernetes Dashboard 后，通过以下命令访问 Dashboard：
  ```bash
  microk8s kubectl proxy
  ```
  然后在浏览器中访问：
  ```
  http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
  ```

## 6. 存储与网络

- **启用 Ingress**：
  ```bash
  microk8s enable ingress
  ```

- **启用存储卷**：
  ```bash
  microk8s enable storage
  ```

- **列出 PVC（持久化卷声明）**：
  ```bash
  microk8s kubectl get pvc
  ```

---
title: "k8s中的PV 和 PVC"
date: 2025-01-17T12:00:00+08:00
lastmod: 2025-01-17T12:00:00+08:00
draft: false
categories: ["容器与云原生"]
tags: ["K8s", "PV", "PVC", "存储"]
---

# Kubernetes PersistentVolumeClaim (PVC) 创建过程

在 Kubernetes 中，**PersistentVolumeClaim (PVC)** 是用于请求持久化存储资源的对象。PVC 请求一定数量的存储资源，并且会与现有的 **PersistentVolume (PV)** 绑定。在这篇文档中，我们将详细介绍 PVC 的创建过程，并讲解 PV 和 PVC 之间的关系。

## 1. 什么是 PV 和 PVC

### 1.1 PersistentVolume (PV)

**PersistentVolume (PV)** 是集群管理员创建的持久化存储资源，表示集群中的实际存储。它是 Kubernetes 的资源之一，表示一个可用的存储容量，可以是各种存储类型（如 `hostPath`、`NFS`、云存储等）。

### 1.2 PersistentVolumeClaim (PVC)

**PersistentVolumeClaim (PVC)** 是用户请求存储的声明，类似于一个存储的请求。当用户在 Kubernetes 中运行应用时，通常需要存储数据。PVC 允许用户在 Kubernetes 中请求持久化存储，并且由集群自动将其绑定到合适的 PV。

### 1.3 PV 和 PVC 之间的关系

- **PVC 是对 PV 的请求**：PVC 表示你需要一定的存储资源，指定了存储大小、访问模式等。PVC 会自动找到一个合适的 PV 来满足请求。如果 PVC 找不到合适的 PV，它将保持 `Pending` 状态。
- **PV 是实际的存储资源**：PV 是管理员创建的存储资源，它不依赖于 PVC。PV 可以通过不同的存储类型提供，且可以被多个 PVC 请求所绑定。
- **PVC 绑定到 PV**：当 PVC 发出请求时，它会选择一个符合条件的 PV 并与之绑定。PVC 和 PV 会通过 `storageClass`、`accessModes`、`capacity` 等进行匹配。

## 2. 创建 PVC 的基本步骤

### 步骤 1: 创建 PersistentVolumeClaim (PVC)

PVC 的基本配置包括请求的存储大小、访问模式以及存储类。以下是一个简单的 PVC 配置示例：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: ""
  volumeMode: Filesystem
```

#### 配置解释：
- `accessModes`: 存储访问模式，`ReadWriteOnce` 表示存储可以被单个节点以读写方式挂载。
- `resources.requests.storage`: PVC 请求的存储大小，在这里是 `1Gi`。
- `storageClassName`: 存储类名称，空值表示使用默认存储类。
- `volumeMode`: 存储模式，`Filesystem` 表示以文件系统的方式使用存储。

### 步骤 2: 创建 PersistentVolume (PV)

PVC 需要一个 PV 来进行绑定。以下是一个简单的 PV 配置，使用 `hostPath` 类型，存储数据在宿主机的某个路径下。

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mypv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: ""
  hostPath:
    path: /mnt/data
    type: DirectoryOrCreate
```

#### 配置解释：
- `capacity.storage`: 定义 PV 提供的存储容量，这里为 `1Gi`。
- `accessModes`: 存储访问模式，`ReadWriteOnce` 表示只能被一个节点以读写方式访问。
- `hostPath.path`: 存储路径，`/mnt/data` 表示宿主机上的路径，用于存储数据。
- `persistentVolumeReclaimPolicy`: `Retain` 表示当 PVC 被删除时，PV 保留其数据。

### 步骤 3: 应用配置

创建 PVC 和 PV 后，你可以使用 `microk8s kubectl` 命令将它们应用到 Kubernetes 集群中。

1. 创建 PV：
   ```bash
   microk8s kubectl apply -f mypv.yaml
   ```

2. 创建 PVC：
   ```bash
   microk8s kubectl apply -f mypvc.yaml
   ```

### 步骤 4: 验证 PVC 和 PV 是否绑定

在创建 PVC 和 PV 后，你可以检查它们的状态，确保它们已经成功绑定。

- 查看 PVC 状态：
   ```bash
   microk8s kubectl get pvc
   ```

- 查看 PV 状态：
   ```bash
   microk8s kubectl get pv
   ```

当 PVC 状态为 `Bound` 时，表示它已成功绑定到 PV。

## 3. 在 Pod 中使用 PVC

一旦 PVC 和 PV 成功绑定，你可以在 Pod 中使用 PVC。例如，在 Deployment 配置文件中，使用 PVC 作为存储卷：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp-container
          image: myimage
          volumeMounts:
            - mountPath: /data
              name: host-storage
      volumes:
        - name: host-storage
          persistentVolumeClaim:
            claimName: mypvc
```

#### 配置解释：
- `volumeMounts`: 将 PVC 挂载到容器的 `/data` 路径。
- `volumes`: 通过 `persistentVolumeClaim` 引用 PVC。

## 4. 总结

1. **PVC 是用于请求存储的声明**，Pod 会根据 PVC 的需求申请存储。
2. **PV 是实际的存储资源**，PVC 会尝试与现有的 PV 绑定。
3. **PVC 和 PV 配置需要匹配**，如存储大小、访问模式等。
4. 在 Pod 中使用 PVC 以挂载存储卷，确保数据持久化。

通过这些步骤，你应该能够成功创建 PVC 和 PV，并将它们应用到 Pod 中，以便持久化存储数据。

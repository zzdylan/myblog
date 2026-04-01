---
title: "使用 Docker 私有注册表（Registry）"
date: 2024-09-05T13:06:20+08:00
lastmod: 2024-09-09T17:31:26+08:00
draft: false
categories: ["容器与云原生"]
tags: ["Docker", "Registry", "私有仓库"]
---

# 运行私有注册表容器：
```bash
docker run -d -p 5000:5000 --name registry registry:2
```

# 将镜像推送到私有注册表：
```bash
docker tag <your-image> localhost:5000/<your-image>
docker push localhost:5000/<your-image>
```

# 从私有注册表中拉取镜像：
docker pull localhost:5000/<your-image>

这种方式可以减少反复从 Docker Hub 下载镜像的次数，提高效率。


# 注意事项
如果需要外网或者局域网访问，需要加白，里面的ip是你对应的外网ip或者局域网ip
```bash
sudo nano /etc/docker/daemon.json
```

```json
{
  "insecure-registries" : ["192.168.4.103:5000"]
}
```
```bash
sudo systemctl restart docker
```

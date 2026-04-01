---
title: "常用docker-compose"
date: 2024-09-09T10:00:00+08:00
lastmod: 2025-09-28T15:45:04+08:00
draft: false
categories: ["容器与云原生"]
tags: ["Docker", "docker-compose", "MySQL", "Redis", "RabbitMQ", "MinIO", "NSQ"]
---

# mysql、redis、rabbitmq
```yaml
version: '3.8'

services:

  mysql:
    image: mysql:8.0.33
    container_name: common-mysql8
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: my
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    #platform: linux/arm64
    platform: linux/amd64
    restart: always

  redis:
    image: redis:6.2.12
    container_name: common-redis6
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    environment:
      TZ: Asia/Shanghai
    #command: ["redis-server", "--requirepass", "yourpassword"]  # 直接在启动命令中写死密码
    #platform: linux/arm64
    platform: linux/amd64
    restart: always

  rabbitmq:
    image: rabbitmq:management
    container_name: rabbitmq-container
    environment:
      - RABBITMQ_DEFAULT_USER=YOURUSERNAME
      - RABBITMQ_DEFAULT_PASS=YOURPASSWORD
    ports:
      - "5672:5672"   # AMQP 端口
      - "15672:15672" # RabbitMQ 管理界面端口


volumes:
  mysql_data:
  redis_data:

```

# minio
```yaml
services:
  minio:
    image: minio/minio:RELEASE.2022-01-04T07-41-07Z
    container_name: minio
    environment:
      MINIO_ROOT_USER: "admin"  # 管理员用户名
      MINIO_ROOT_PASSWORD: "123456"  # 管理员密码
    ports:
      - "9001:9000"  # MinIO API 端口
      - "9002:9001"  # MinIO 控制台端口
    volumes:
      - ./minio_data:/data  # 持久化存储
    command: server /data --console-address ":9001"
    restart: unless-stopped
```

# nsq
```yaml
services:
  nsqlookupd:
    image: nsqio/nsq:v1.3.0
    command: /nsqlookupd
    ports:
      - "4160:4160"  # TCP 端口，供 nsqd 注册
      - "4161:4161"  # HTTP 端口，供 nsqadmin 和客户端查询
    networks:
      - nsq-network

  nsqd:
    image: nsqio/nsq:v1.3.0
    command: /nsqd 
      --lookupd-tcp-address=nsqlookupd:4160 
      --broadcast-address=nsqd  # 改为容器名或确保IP正确
    depends_on:
      - nsqlookupd
    ports:
      - "4150:4150"  # TCP 端口，供生产者连接
      - "4151:4151"  # HTTP 端口，供管理操作
    networks:
      - nsq-network

  nsqadmin:
    image: nsqio/nsq:v1.3.0
    command: /nsqadmin --lookupd-http-address=nsqlookupd:4161
    depends_on:
      - nsqlookupd
    ports:
      - "4171:4171"  # Web 管理界面
    networks:
      - nsq-network

networks:
  nsq-network:
    driver: bridge
```

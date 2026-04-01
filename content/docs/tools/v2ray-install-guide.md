---
title: "V2Ray 完整安装配置指南"
date: 2025-09-04T10:00:00+08:00
lastmod: 2025-09-04T10:00:00+08:00
draft: false
categories: ["工具"]
tags: ["V2Ray", "代理", "网络"]
---

# 🧭 Debian 系统 V2Ray 配置与 systemd 自启动指南

## 一、下载并部署 V2Ray

以 **Debian 系统** 为例：

```bash
# 进入临时目录
cd /tmp

# 下载 V2Ray 核心程序（选择最新版本）
curl -LO https://github.com/v2fly/v2ray-core/releases/latest/download/v2ray-linux-64.zip

# 解压到 /usr/local/v2ray
sudo mkdir -p /usr/local/v2ray
sudo unzip v2ray-linux-64.zip -d /usr/local/v2ray
```

> 💡 如果是其他架构（如 ARM64），请在 [V2Ray Releases 页面](https://github.com/v2fly/v2ray-core/releases) 选择对应包。

---

## 二、写入配置文件

编辑配置文件：

```bash
sudo nano /usr/local/v2ray/config.json
```

粘贴以下内容（按需修改 `address`、`port`、`user`、`pass` 等字段）：

```json
{
  "log": {
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "port": 10808,
      "protocol": "vmess",
      "tag": "vmess-in",
      "settings": {
        "clients": [
          {
            "id": "42876AD8-6202-41D8-B2FE-46E00C939BB7",
            "level": 1,
            "alterId": 0,
            "security": "auto"
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "none",
        "tcpSettings": {
          "header": {
            "type": "none"
          }
        }
      },
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "socks",
      "settings": {
        "servers": [
          {
            "address": "xx",
            "port": 49699,
            "users": [
              {
                "user": "xx",
                "pass": "xx"
              }
            ]
          }
        ]
      },
      "tag": "proxy"
    },
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    }
  ],
  "routing": {
    "domainStrategy": "AsIs",
    "rules": [
      {
        "type": "field",
        "inboundTag": ["vmess-in"],
        "outboundTag": "proxy"
      }
    ]
  },
  "dns": {
    "servers": [
      "8.8.8.8",
      "8.8.4.4",
      "1.1.1.1"
    ]
  }
}
```

> ⚠️ 注意：上游 socks5 代理在 V2Ray 中协议字段依旧写为 `"protocol": "socks"`。

---

## 三、创建 systemd 服务并设置开机自启

创建 systemd 服务文件：
/etc/systemd/system/v2ray.service
```bash
[Unit]
Description=V2Ray Service
After=network.target

[Service]
ExecStart=/usr/local/v2ray/v2ray run -config /usr/local/v2ray/config.json
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

启用并启动服务：

```bash
sudo systemctl daemon-reload
sudo systemctl enable v2ray
sudo systemctl start v2ray
sudo systemctl status v2ray
```

---

## ✅ 验证运行状态

```bash
sudo systemctl status v2ray
sudo journalctl -u v2ray -e
```

当输出中显示 "Active: active (running)" 时，即代表服务运行成功。

---

## 📚 参考

- [V2Fly 官方文档](https://www.v2fly.org)
- [V2Ray-Core GitHub](https://github.com/v2fly/v2ray-core)

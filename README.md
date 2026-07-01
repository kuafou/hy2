# hy2

基于 [Hysteria 2](https://v2.hysteria.network/) 的服务端与客户端 Docker 部署配置。

## 目录结构

```
hy2/
├── server/                 # 服务端
│   ├── hy-server.yaml      # 服务端配置
│   ├── compose.yaml        # Docker Compose
│   └── certs/              # TLS 自签证书
│       ├── server.crt
│       └── server.key
└── client/                 # 客户端
    ├── hy-client.yaml      # 客户端配置
    └── compose.yaml        # Docker Compose
```

## 服务端

### 前置条件

- 将 `pro.mlsys.fun` 解析到服务器 IP
- 防火墙/安全组放行 **UDP 13443**

### 生成自签证书

证书已生成时可跳过。在 `server` 目录下执行：

```bash
mkdir -p certs
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 \
  -keyout certs/server.key \
  -out certs/server.crt \
  -subj "/CN=pro.mlsys.fun"
```

### 启动

```bash
cd server
docker compose up -d
```

### 配置说明

| 项 | 值 |
|---|---|
| 监听端口 | UDP 13443 |
| TLS | 自签证书（`certs/server.crt`、`certs/server.key`） |
| 认证 | 密码（见 `hy-server.yaml`） |
| 带宽 | 上行/下行各 100 mbps |
| 伪装 | 代理至 bing.com |

## 客户端

### 启动

```bash
cd client
docker compose up -d
```

### 配置说明

| 项 | 值 |
|---|---|
| 服务端地址 | `pro.mlsys.fun:13443` |
| TLS | `insecure: true`（跳过自签证书校验） |
| SOCKS5 | `127.0.0.1:1088`（映射容器 1080） |
| HTTP 代理 | `127.0.0.1:7080`（映射容器 6080） |

### 使用代理

```bash
# SOCKS5
curl -x socks5h://127.0.0.1:1088 https://example.com

# HTTP
curl -x http://127.0.0.1:7080 https://example.com
```

## 日志

服务端与客户端均配置了 Docker 日志轮转，单文件最大 10MB，最多保留 3 个文件（合计约 30MB）。

修改配置后需重建容器生效：

```bash
docker compose up -d
```

## 注意事项

- 服务端与客户端的 **auth 密码**、**带宽** 需保持一致
- Hysteria 基于 QUIC，端口映射必须使用 **UDP**
- 生产环境建议更换默认密码，并妥善保管 `certs/server.key`

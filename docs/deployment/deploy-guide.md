---
id: DEPLOY-001
title: 部署指南
type: deployment
module: deployment
tags: [docker, deploy]
status: draft
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
---

# 部署指南

> 本文档描述智慧大棚AIoT系统的完整部署流程。
> 当前为框架文档，具体内容后续补充。

---

## 环境要求

### 硬件要求

| 资源 | 最低配置 | 推荐配置 |
|------|----------|----------|
| CPU | 4核 | 8核+ |
| 内存 | 8GB | 16GB+ |
| 磁盘 | 50GB SSD | 100GB+ SSD |
| 操作系统 | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04 LTS |

### 软件依赖

| 软件 | 版本 | 用途 |
|------|------|------|
| Docker | 24.x+ | 容器运行时 |
| Docker Compose | v2.x+ | 容器编排 |
| Git | 2.x+ | 代码拉取 |
| FFmpeg | 6.x | 视频截帧 |
| frp | latest | 内网穿透 |

### 网络要求

- 外网访问：AI API调用（百度AI、讯飞、DeepSeek、硅基流动、和风天气）
- 内网访问：ESP32设备通过WiFi连接MQTT Broker
- frp内网穿透：将本地服务暴露到公网（可选，用于APP远程访问）

### 端口规划

| 端口 | 服务 | 说明 |
|------|------|------|
| 8080 | Spring Boot | 后端API服务 |
| 3306 | MySQL | 关系数据库 |
| 8086 | InfluxDB | 时序数据库 |
| 8000 | Chroma | 向量数据库 |
| 1883 | Mosquitto MQTT | MQTT Broker |
| 9001 | Mosquitto WebSocket | MQTT WebSocket端口 |
| 80/443 | Nginx | Web管理端+反向代理 |

> [待补充] 详细端口说明和防火墙配置

---

## Docker Compose部署

### 服务清单

| 服务 | 镜像 | 说明 |
|------|------|------|
| mysql | mysql:8.0 | 业务数据库 |
| influxdb | influxdb:2.7 | 时序数据库 |
| chroma | chromadb/chroma:latest | 向量数据库 |
| mosquitto | eclipse-mosquitto:2 | MQTT Broker |
| nginx | nginx:alpine | Web服务器+反向代理 |
| spring-boot | 自构建镜像 | 后端API服务 |
| vue-web | 自构建镜像 | Vue Web管理端 |

> [待补充] 完整 docker-compose.yml 配置内容

### 启动步骤

```bash
# 1. 克隆项目
git clone <repository-url>
cd Smart_Greenhouse

# 2. 配置环境变量
cp .env.example .env
# 编辑 .env 文件，填入真实API Key和密码

# 3. 构建并启动所有服务
docker compose up -d

# 4. 查看服务状态
docker compose ps

# 5. 查看日志
docker compose logs -f
```

> [待补充] 详细启动步骤和常见问题处理

---

## 环境变量配置

### 必需环境变量

| 变量名 | 说明 | 示例 |
|--------|------|------|
| MYSQL_PASSWORD | MySQL root密码 | `your_secure_password` |
| INFLUX_TOKEN | InfluxDB访问Token | `your_influx_token` |
| DEEPSEEK_API_KEY | DeepSeek API密钥 | `sk-xxxxx` |
| SILICONFLOW_API_KEY | 硅基流动API密钥 | `sk-xxxxx` |
| BAIDU_API_KEY | 百度AI API Key | `your_baidu_key` |
| BAIDU_SECRET_KEY | 百度AI Secret Key | `your_baidu_secret` |
| XUNFEI_APP_ID | 讯飞应用ID | `your_xunfei_app_id` |
| XUNFEI_API_KEY | 讯飞API Key | `your_xunfei_key` |
| XUNFEI_API_SECRET | 讯飞API Secret | `your_xunfei_secret` |
| QWEATHER_API_KEY | 和风天气API Key | `your_qweather_key` |

### 可选环境变量

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| MQTT_USER | MQTT用户名 | `greenhouse` |
| MQTT_PASSWORD | MQTT密码 | `greenhouse_dev` |
| FRP_SERVER_ADDR | frp服务器地址 | - |
| FRP_SERVER_PORT | frp服务器端口 | `7000` |
| FRP_TOKEN | frp认证Token | - |
| JWT_SECRET | JWT签名密钥 | 随机生成 |

> [待补充] 完整 .env.example 模板内容

---

## 数据库初始化

### MySQL初始化

数据库初始化脚本位于 `sql/` 目录：

```
sql/
├── V1__init_schema.sql       # 建表脚本（25张表）
├── V2__init_data.sql         # 初始数据（管理员账号、默认预警规则等）
└── V3__seed_test_data.sql    # 测试数据（可选）
```

初始化步骤：

```bash
# 启动MySQL后，自动执行初始化脚本
docker compose exec mysql mysql -u root -p${MYSQL_PASSWORD} < sql/V1__init_schema.sql
docker compose exec mysql mysql -u root -p${MYSQL_PASSWORD} < sql/V2__init_data.sql
```

### InfluxDB初始化

```bash
# 创建Bucket
docker compose exec influxdb influx bucket create \
  --name sensor_data \
  --org greenhouse \
  --retention 90d
```

### Chroma初始化

Chroma集合在Spring Boot启动时通过 `ChromaConfig` 自动创建，无需手动初始化。

> [待补充] 数据库初始化详细步骤和Flyway迁移脚本配置

---

## frp内网穿透配置

### 适用场景

当服务器部署在本地局域网（无公网IP）时，通过frp将服务暴露到公网，使APP端可以远程访问。

### frp服务端配置 (frps.toml)

```toml
bindPort = 7000
auth.token = "your_frp_token"
```

### frp客户端配置 (frpc.toml)

```toml
serverAddr = "your_frp_server_address"
serverPort = 7000
auth.token = "your_frp_token"

[[proxies]]
name = "web"
type = "http"
localPort = 80
customDomains = ["greenhouse.yourdomain.com"]

[[proxies]]
name = "api"
type = "tcp"
localPort = 8080
remotePort = 8080

[[proxies]]
name = "mqtt"
type = "tcp"
localPort = 1883
remotePort = 1883
```

### 启动frp

```bash
# 服务端
./frps -c frps.toml

# 客户端（与Docker服务同一台机器）
./frpc -c frpc.toml
```

> [待补充] frp详细配置和HTTPS配置

---

## 验证部署

### 健康检查

```bash
# 检查所有容器状态
docker compose ps

# 检查Spring Boot健康状态
curl http://localhost:8080/actuator/health

# 检查API是否正常
curl http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"Admin@123"}'

# 检查Web管理端
curl http://localhost/

# 检查MQTT连接
mosquitto_sub -h localhost -t "greenhouse/+/device/+/heartbeat"
```

### 验收清单

| 检查项 | 验证方法 | 预期结果 |
|--------|----------|----------|
| MySQL可连接 | `docker compose exec mysql mysql -u root -p` | 进入MySQL命令行 |
| InfluxDB可连接 | 访问 `http://localhost:8086` | 显示InfluxDB UI |
| Chroma可连接 | `curl http://localhost:8000/api/v1/heartbeat` | 返回心跳信息 |
| MQTT可连接 | MQTT客户端连接 `tcp://localhost:1883` | 连接成功 |
| 后端API正常 | 访问 `/api/v1/auth/login` | 返回正常JSON响应 |
| Web管理端正常 | 访问 `http://localhost/` | 显示Vue登录页面 |
| frp穿透正常 | 通过公网域名访问 | 正常访问Web管理端和API |

> [待补充] 完整验收流程和自动化验收脚本

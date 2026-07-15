---
id: TASK-DOCKER
title: Docker部署环境搭建
module: Docker部署
type: task
priority: high
status: completed
created: 2026-07-12
completed: 2026-07-12
assignee: AI助手
related_docs:
  - DEVLOG-步骤2
tags: [docker, devops, mysql, influxdb, mosquitto, chroma]
---

# TASK-DOCKER: Docker 部署环境搭建

## 需求描述

搭建项目所需的全部基础设施 Docker 环境，实现一条 `docker-compose up -d` 命令即可启动所有中间件。

## 完成内容

### 步骤2 已完成（2026-07-12）

| 文件 | 说明 |
|------|------|
| `docker-compose.yml` | 4 个服务：MySQL 8.0 / InfluxDB 2.7 / Mosquitto 2.x / Chroma，含健康检查和数据卷持久化 |
| `mosquitto.conf` | MQTT Broker 配置，开发阶段允许匿名连接，含 WebSocket MQTT（端口 9001） |
| `.env.example` | 环境变量模板，含所有第三方 API 占位符 |

### 服务清单

| 服务 | 镜像 | 端口 | 说明 |
|------|------|------|------|
| MySQL | mysql:8.0 | 3306 | 业务数据存储（25 张表） |
| InfluxDB | influxdb:2.7 | 8086 | 时序数据存储（传感器读数） |
| Mosquitto | eclipse-mosquitto:2 | 1883/9001 | MQTT Broker（TCP + WebSocket） |
| Chroma | chromadb/chroma:latest | 8000 | 向量数据库（知识库 RAG） |

## 结果

一条 `docker-compose up -d` 命令即可启动全部基础设施。MQTT 匿名连接后期改认证只需改配置+重启，不影响代码。

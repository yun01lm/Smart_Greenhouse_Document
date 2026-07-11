---
id: ROOT-001
title: 智慧大棚AIoT系统 — 项目概述
type: overview
module: root
tags: [overview, quickstart]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - INDEX.md
  - docs/prd/
  - docs/tech/
  - docs/api/
  - docs/database/
  - docs/standards/development-standards.md
  - devlog/INDEX.md
---

# 智慧大棚AIoT系统

> **项目全称**：基于多模态融合的智慧大棚作物诊疗与智能管控APP
>
> **项目类型**：大学生创新创业训练计划项目（大创）— 创新训练
>
> **研究期限**：2026年05月 — 2027年05月（12个月）

---

## 项目简介

本项目研发一款 AIoT 智慧大棚管理 APP，融合环境数值、作物图像、方言语音多模态数据，实现环境趋势智能预警、生长状态评估、病害自主诊断与设备远程联动控制。

**核心闭环**：感知（传感器+摄像头）→ 分析（AI+知识库+多模态融合）→ 决策（预警+处方）→ 控制（设备联动）→ 兜底（专家咨询）

---

## 技术栈

| 层级 | 技术 | 说明 |
|------|------|------|
| Android APP | Java + MVVM + Jetpack | target SDK 34 |
| Web管理端 | Vue 3 + Element Plus | Composition API |
| 后端 | Spring Boot 3.3.x (Java 17+) | API Only |
| AI框架 | Spring AI 1.0.9 | RAG + LLM调用 |
| 关系数据库 | MySQL 8.0 | 业务数据 |
| 时序数据库 | InfluxDB 2.7 | 传感器数据 |
| 向量数据库 | Chroma | RAG知识库 |
| MQTT | Eclipse Mosquitto 2.x | 设备通信 |
| 实时通信 | WebSocket (STOMP) | 数据推送 + 聊天 |
| 硬件主控 | ESP32 (3-5组/大棚) | 传感采集 + 设备控制 |
| 部署 | Docker Compose + frp | 本地Linux |

---

## 文档导航

| 入口 | 路径 | 说明 |
|------|------|------|
| 📇 **文档总索引** | [INDEX.md](./INDEX.md) | 所有文档的索引和分类导航 |
| 📋 **功能PRD** | [docs/prd/](./docs/prd/) | 各功能模块需求文档 |
| 🏗️ **技术方案** | [docs/tech/](./docs/tech/) | 各模块技术设计文档 |
| 🔌 **API文档** | [docs/api/](./docs/api/) | 接口文档（按模块分组） |
| 🗄️ **数据库设计** | [docs/database/](./docs/database/) | 表结构 + 表-模块映射 |
| 📏 **开发规范** | [docs/standards/](./docs/standards/) | 编码/Git/文档规范 |
| 📝 **开发日志** | [devlog/](./devlog/) | 任务跟踪与开发记录 |
| 🚀 **部署运维** | [docs/deployment/](./docs/deployment/) | 部署指南 + 运维手册 |
| 🧪 **测试** | [docs/test/](./docs/test/) | 测试用例 |
| 📦 **变更日志** | [CHANGELOG.md](./CHANGELOG.md) | 版本变更记录 |

---

## 本地运行

### 环境要求

- JDK 17+
- Maven 3.9+
- Node.js 18+
- Docker 24+ & Docker Compose 2.x
- Android Studio Hedgehog+

### 启动步骤

```bash
# 1. 启动基础设施
cd Smart_Greenhouse_Project
docker-compose up -d mysql influxdb chroma mosquitto

# 2. 启动后端
cd backend
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# 3. 启动Web管理端
cd web-admin
pnpm install
pnpm dev

# 4. Android APP
# 用 Android Studio 打开 app/ 目录，运行到模拟器或真机
```

---

## 角色与权限

| 角色 | 可用端 | 权限范围 |
|------|--------|----------|
| 棚主(OWNER) | APP + Web | 自己的大棚、全部功能、管理员工 |
| 员工(WORKER) | 仅APP | 被授权大棚 + 被授权功能（单归属） |
| 专家(EXPERT) | 仅Web | 被授权大棚（7天）+ 查看数据 + 对话 |
| 管理员(ADMIN) | 仅Web | 全部大棚、全部功能、平台管理 |

---

## 开发计划

| Phase | 时间 | 核心任务 |
|-------|------|----------|
| Phase 1 | 2026.07-08 | 后端骨架 + 硬件原型 |
| Phase 2 | 2026.09-10 | APP核心 + AI API接入 + 知识库 |
| Phase 3 | 2026.11-12 | 多组传感器 + 专家系统 + 权限系统 |
| Phase 4 | 2027.01-02 | ResNet训练 + Whisper微调 + LSTM |
| Phase 5 | 2027.03-05 | 部署调试 + 数据收集 + 论文结题 |

# 智慧大棚AIoT系统 — 项目总文档

> 基于多模态融合的智慧大棚作物诊疗与智能管控系统  
> 大学创新创业项目 | 版本 v1.0

---

## 1. 项目概述

智慧大棚AIoT系统是一个**软硬结合**的农业智能化平台，融合物联网（IoT）、人工智能（AI）和多模态数据分析技术，实现大棚环境的实时监控、智能预警、AI 病虫害诊断、RAG 知识问答和专家远程协作。

### 核心能力

| 能力 | 说明 |
|------|------|
| 🌡️ **环境监控** | 11 种传感器实时采集（温度/湿度/CO₂/光照/土壤等），InfluxDB 时序存储 |
| 🚨 **智能预警** | 阈值/趋势/复合/天气 4 种规则引擎，三级告警（提示/警告/严重） |
| 🔬 **AI 诊断** | 百度 AI 植物识别 + ResNet 本地模型（预留），病虫害图片诊断 |
| 💬 **RAG 问答** | 知识库向量化（SiliconFlow bge-m3）+ Chroma 检索 + DeepSeek 生成 |
| 🎤 **语音交互** | 讯飞 ASR 方言识别（河北话）+ TTS 语音播报 |
| 📊 **多模态融合** | 环境数据(60%) + 视觉诊断(40%) × 天气修正 → 0-100 健康评分 |
| 👨‍⚕️ **专家协作** | 实时聊天 + 环境快照 + 数据授权（7 天有效期） |
| 🎛️ **设备控制** | MQTT 远程控制风机/卷帘/遮阳/灌溉/补光灯 + 场景联动 |

---

## 2. 技术架构

```
┌─────────────────────────────────────────────────────┐
│                    前端展示层                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │ Web 管理端│  │ Android  │  │   ESP32 固件     │  │
│  │ Vue3+Ele  │  │ 原生 APP │  │ (延后，硬件到货)  │  │
│  │ ment Plus │  │  Java    │  │                  │  │
│  └─────┬─────┘  └────┬─────┘  └────────┬─────────┘  │
│        │             │                │              │
├────────┼─────────────┼────────────────┼──────────────┤
│        ▼             ▼                ▼              │
│  ┌──────────────────────────────────────────────┐    │
│  │         Spring Boot 3.3.5 + Java 21         │    │
│  │  ┌────────┐ ┌────────┐ ┌──────────────────┐ │    │
│  │  │Security │ │  JWT   │ │WebSocket STOMP   │ │    │
│  │  │多角色   │ │ 认证   │ │实时推送           │ │    │
│  │  └────────┘ └────────┘ └──────────────────┘ │    │
│  │  ┌──────────────────────────────────────┐   │    │
│  │  │  22 个业务模块 (Controller→Service→Repo) │   │    │
│  │  └──────────────────────────────────────┘   │    │
│  └──────────────────┬───────────────────────────┘    │
│                     │                                │
├─────────────────────┼────────────────────────────────┤
│                     ▼             数据层              │
│  ┌────────┐  ┌──────────┐  ┌────────┐  ┌─────────┐  │
│  │ MySQL  │  │ InfluxDB │  │ Chroma │  │Mosquitto│  │
│  │ 8.0    │  │   2.7    │  │向量库  │  │  MQTT   │  │
│  │业务数据│  │ 时序数据 │  │RAG检索 │  │消息队列 │  │
│  └────────┘  └──────────┘  └────────┘  └─────────┘  │
│                                                      │
├──────────────────────────────────────────────────────┤
│                    AI 服务层                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌─────────┐ │
│  │ 百度 AI  │ │ 讯飞 ASR │ │DeepSeek  │ │Silicon  │ │
│  │植物识别  │ │ 语音识别 │ │  LLM    │ │Flow Emb │ │
│  └──────────┘ └──────────┘ └──────────┘ └─────────┘ │
└──────────────────────────────────────────────────────┘
```

### 技术栈

| 层级 | 技术 | 版本 |
|------|------|------|
| 后端框架 | Spring Boot | 3.3.5 |
| 语言 | Java | 21 |
| 安全 | Spring Security + JWT | jjwt 0.12.6 |
| ORM | Spring Data JPA | — |
| 时序数据库 | InfluxDB | 2.7 |
| 向量数据库 | Chroma | latest |
| 消息队列 | Eclipse Mosquitto (MQTT) | 2.x |
| Web 前端 | Vue 3 + Vite + Element Plus | 3.x |
| Android | 原生 Java + Retrofit + MPAndroidChart | SDK 34 |
| AI | Spring AI + DeepSeek + SiliconFlow + 百度 AI + 讯飞 | — |
| 容器化 | Docker Compose | — |
| 构建工具 | Maven + Gradle | — |

---

## 3. 项目结构

```
Smart_Greenhouse/
├── Smart_Greenhouse_Project/          # 代码仓库 (Smart-Greenhouse)
│   ├── pom.xml                        # 父 POM
│   ├── common/                        # 公共模块 (ApiResponse/PageResult/ErrorCode)
│   ├── backend/                       # Spring Boot 后端 (181 Java 文件)
│   │   └── src/main/java/com/greenhouse/
│   │       ├── config/                # Security/MQTT/InfluxDB/WebSocket 配置
│   │       ├── entity/                # JPA 实体 (25 张表)
│   │       ├── repository/            # 数据访问层
│   │       ├── security/              # JWT + AOP 权限切面
│   │       ├── ai/                    # AI 策略接口 (诊断/语音)
│   │       └── module/               # 22 个业务模块
│   ├── web/                           # Vue 3 管理端 (40 文件)
│   ├── app/                           # Android APP (116 Java 文件)
│   ├── simulator/                     # ESP32 设备模拟器
│   └── docker-compose.yml            # Docker 基础设施
│
└── document/                          # 文档仓库 (Smart_Greenhouse_Document)
    ├── README.md                      # 本文档
    ├── INDEX.md                       # 文档索引
    ├── AI开发规则文档_v2.0.md          # AI 开发规范
    ├── 架构审查报告_2026-07-14.md       # 架构审查
    ├── 架构整改规划_2026-07-14.md       # 整改规划
    ├── api/                           # 18 个 API 文档
    ├── database/                      # 数据库设计文档
    ├── devlog/                        # 开发日志 (45+ 任务日志)
    └── tech/                          # 技术设计文档 (8 篇)
```

---

## 4. 快速开始

### 4.1 环境要求

- JDK 21+
- Node.js 18+
- Docker & Docker Compose
- Android Studio (APP 端)

### 4.2 启动基础设施

```bash
cd Smart_Greenhouse_Project
docker-compose up -d
```

启动后验证：
- MySQL: `localhost:3306` (root/root123)
- InfluxDB: `http://localhost:8086`
- Chroma: `http://localhost:8000`
- MQTT: `localhost:1883` (TCP) / `localhost:9001` (WebSocket)

### 4.3 配置 API 密钥

复制 `.env.example` 为 `.env`，填入实际密钥：

```bash
cp .env.example .env
# 编辑 .env 填入:
# - SILICONFLOW_API_KEY    (向量化 + LLM)
# - DEEPSEEK_API_KEY       (RAG 问答)
# - BAIDU_AI_API_KEY       (图片诊断)
# - XUNFEI_APP_ID          (语音识别)
# - QWEATHER_KEY           (天气 API)
```

### 4.4 启动后端

```bash
cd backend
# 配置 application-dev.yml 中的数据库连接和 API 密钥
mvn spring-boot:run -Dspring-boot.run.profiles=dev
```

### 4.5 启动 Web 前端

```bash
cd web
npm install
npm run dev
# 访问 http://localhost:5173
# 默认管理员账号: admin / admin123
```

### 4.6 启动数据模拟器

```bash
cd simulator
pip install paho-mqtt
python device_simulator.py --mode normal
```

---

## 5. API 概览

所有 API 路径前缀：`/api/v1/`，认证方式：JWT Bearer Token。

| 模块 | 路径 | 端点数 | 认证 |
|------|------|:--:|------|
| 用户认证 | `/api/v1/auth/*` | 3 | 白名单 |
| 大棚管理 | `/api/v1/greenhouses/*` | 7 | 已认证 |
| 设备管理 | `/api/v1/greenhouses/{id}/devices/*` | 5 | 已认证 |
| 传感器数据 | `/api/v1/sensors/*` | 5 | 已认证 |
| 预警管理 | `/api/v1/alerts/*` | 9 | 已认证 |
| 设备控制 | `/api/v1/control/*` | 7 | 已认证 |
| 病虫害诊断 | `/api/v1/diagnosis/*` | 2 | 已认证 |
| RAG 问答 | `/api/v1/qa/*` | 3 | 已认证 |
| 天气服务 | `/api/v1/weather/*` | 2 | 已认证 |
| 作物周期 | `/api/v1/crop-cycles/*` | 7 | 已认证 |
| 实时聊天 | `/api/v1/chat/*` | 7 | 已认证 |
| 专家授权 | `/api/v1/expert/*` | 8 | 已认证 |
| 健康评分 | `/api/v1/health/*` | 3 | 已认证 |
| **管理员** | `/api/v1/admin/*` | **25** | ADMIN |
| WebSocket | `/ws/connect` | — | STOMP |

---

## 6. 角色权限

| 角色 | 权限范围 |
|------|------|
| **ADMIN** | 全系统管理：用户管理、知识库、预警配置、数据导出、系统监控、语料管理、专家工作台、棚主管理 |
| **OWNER** | 自有大棚管理：设备、预警规则、员工权限分配、场景联动、控制设备 |
| **WORKER** | 受限操作：按棚主分配的 6 项功能权限（查看数据/控制设备/诊断/问答/查看预警/查看历史） |
| **EXPERT** | 专家服务：在线接诊、实时聊天、申请数据授权、查看授权大棚数据 |

---

## 7. 数据库设计

### MySQL（25 张表）

| 表名 | 说明 |
|------|------|
| users | 用户表（ADMIN/OWNER/WORKER/EXPERT 四角色） |
| greenhouses | 大棚表（五级地址） |
| devices | 设备表（传感器/控制器，8 种子类型） |
| device_groups | 设备分组表 |
| employee_permissions | 员工权限表（6 项功能权限） |
| alert_rules | 预警规则表（4 种规则类型） |
| alert_records | 预警记录表 |
| user_alert_thresholds | 用户自定义阈值表 |
| scenes | 场景联动表 |
| control_logs | 设备控制日志表 |
| diagnostic_records | 病虫害诊断记录表 |
| qa_records | AI 问答记录表 |
| chat_conversations | 聊天会话表 |
| chat_messages | 聊天消息表 |
| data_authorizations | 数据授权表（5 种状态） |
| expert_availability | 专家在线状态表 |
| weather_cache | 天气缓存表 |
| crop_cycles | 作物生长周期表 |
| health_assessments | 多模态健康评估表 |
| knowledge_documents | 知识库文档表 |
| dialect_corpus | 方言语料表 |

### InfluxDB

- Bucket: `greenhouse` — 传感器时序数据

### Chroma

- Collection: `greenhouse_knowledge` — 知识库向量

---

## 8. 开发进度

| Phase | 内容 | 任务数 | 状态 |
|-------|------|:--:|:--:|
| Phase 1 | 基础设施 | 4 | ✅ |
| Phase 2 | 核心功能 | 16 | ✅ |
| Phase 3 | AI 能力 + 专家系统 | 13 | ✅ |
| Phase 4 | 管理端 + 训练 | 10 | ✅ |
| Phase 5 | 集成验收 | 4 | 🔄 |

| 端 | 任务数 | 状态 |
|------|:--:|:--:|
| 后端模块 (C01-C22) | 22 | ✅ 全部完成 |
| Android APP (F01-F11) | 11 | ✅ 全部完成 |
| Web 管理端 (G01-G10) | 10 | ✅ 全部完成 |
| Docker 部署 | 1 | ✅ 已完成 |
| ESP32 固件 | 2 | ⏸️ 延后 |
| 项目文档 | 1 | ✅ 本次完成 |

**总计**: 46/47 已完成，2 个 ESP32 任务延后（等硬件到货）。

---

## 9. Git 仓库

| 仓库 | 地址 | 说明 |
|------|------|------|
| 代码仓库 | [yun01lm/Smart-Greenhouse](https://github.com/yun01lm/Smart-Greenhouse) | Java + Vue + Android |
| 文档仓库 | [yun01lm/Smart_Greenhouse_Document](https://github.com/yun01lm/Smart_Greenhouse_Document) | 设计文档 + 开发日志 |

---

## 10. 许可证

本项目为大学创新创业项目，仅供学习和研究使用。

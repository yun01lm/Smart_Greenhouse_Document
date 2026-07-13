---
id: INDEX-ROOT
title: 智慧大棚AIoT系统 — 项目文档总索引
type: index
module: root
tags: [index, navigation, entry-point]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs: []
---

# 项目文档总索引 (INDEX)

> **这是AI进入项目后读取的第一个文件。** 本文档提供所有项目文档的索引和分类导航。
>
> 项目全称：基于多模态融合的智慧大棚作物诊疗与智能管控APP
>
> 项目根目录：`F:\Smart_Greenhouse\`

---

## 快速导航

| 文档类型 | 目录 | 数量 | 说明 |
|----------|------|------|------|
| 📇 项目概述 | [README.md](./README.md) | 1 | 项目简介、技术栈、快速开始 |
| 📋 PRD需求文档 | [docs/prd/](./docs/prd/) | 12 | 各功能模块需求文档 |
| 🏗️ 技术方案 | [docs/tech/](./docs/tech/) | 8 | 各模块技术设计文档 |
| 🔌 API文档 | [docs/api/](./docs/api/) | 18 | 接口文档（按模块分组） |
| 🗄️ 数据库设计 | [docs/database/](./docs/database/) | 2 | 表结构 + 表-模块映射 |
| 📏 开发规范 | [docs/standards/](./docs/standards/) | 1 | 编码/Git/文档规范 |
| 🚀 部署运维 | [docs/deployment/](./docs/deployment/) | 2 | 部署指南 + 运维手册 |
| 🧪 测试 | [docs/test/](./docs/test/) | 1 | 测试用例模板 |
| 📝 开发日志 | [devlog/](./devlog/) | 5 | 任务跟踪 + 开发日志 + 模板 |
| 📦 变更日志 | [CHANGELOG.md](./CHANGELOG.md) | 1 | 版本变更记录 |
| 📐 AI开发规则 | 项目根目录 | 1 | AI开发约束v2.2（最高优先级） |

---

## 一、完整文档清单

### 1.1 根目录文档

| 文件 | 类型 | 状态 | 说明 |
|------|------|------|------|
| [README.md](./README.md) | overview | final | 项目简介、技术栈、快速开始、角色权限 |
| [CHANGELOG.md](./CHANGELOG.md) | changelog | active | 版本变更记录 |
| [INDEX.md](./INDEX.md) | index | final | 📍 本文档，文档总索引 |

### 1.2 PRD需求文档 (docs/prd/)

| ID | 文件 | 模块 | 标签 | 状态 | 说明 |
|----|------|------|------|------|------|
| PRD-001 | [dashboard-realtime.md](./docs/prd/dashboard-realtime.md) | 数据看板 | realtime, dashboard, sensor | final | 实时数据看板：仪表盘/趋势曲线/多组对比/聚合视图 |
| PRD-002 | [alert-center.md](./docs/prd/alert-center.md) | 预警中心 | alert, threshold, notification | final | 环境预警中心：分级告警/自定义阈值/天气预判/异常模式 |
| PRD-003 | [disease-diagnosis.md](./docs/prd/disease-diagnosis.md) | 病虫害诊断 | diagnosis, ai, image-recognition | final | 病虫害拍照诊断：AI识别/防治方案/低置信度引导专家 |
| PRD-004 | [growth-assessment.md](./docs/prd/growth-assessment.md) | 长势评估 | growth, camera, lifecycle | final | 作物长势评估：截帧查看/株高叶面积叶色/生长阶段 |
| PRD-005 | [ai-qa.md](./docs/prd/ai-qa.md) | AI问答 | qa, rag, voice, dialect | final | AI智能问答：RAG文字问答/河北方言语音问答/来源追溯 |
| PRD-006 | [device-control.md](./docs/prd/device-control.md) | 设备控制 | control, actuator, scene | final | 设备控制：手动控制/场景联动/按区域控制/状态反馈 |
| PRD-007 | [expert-consultation.md](./docs/prd/expert-consultation.md) | 专家咨询 | expert, chat, authorization | final | 专家咨询：实时聊天/环境快照/7天授权/双轨制 |
| PRD-008 | [role-permission.md](./docs/prd/role-permission.md) | 权限管理 | role, permission, employee | final | 多角色权限：四种角色/大棚授权/功能授权/员工单归属 |
| PRD-009 | [multi-sensor.md](./docs/prd/multi-sensor.md) | 多组传感器 | sensor, group, device | final | 多组传感器管理：3-5组独立ESP32/区域标注/三视图 |
| PRD-010 | [crop-cycle.md](./docs/prd/crop-cycle.md) | 生长周期 | crop, lifecycle, growth | final | 作物生长周期：种植记录/阶段估算/生长时间线 |
| PRD-011 | [health-assessment.md](./docs/prd/health-assessment.md) | 健康评估 | health, multimodal, fusion | final | 多模态健康评估：环境+图像+天气→综合健康评分 |
| PRD-012 | [knowledge-corpus.md](./docs/prd/knowledge-corpus.md) | 知识库 | knowledge, corpus, dialect | final | 知识库与语料管理：RAG数据源/方言语料集/向量化 |

### 1.3 技术方案文档 (docs/tech/)

| ID | 文件 | 模块 | 标签 | 状态 | 说明 |
|----|------|------|------|------|------|
| TECH-001 | [perception-layer.md](./docs/tech/perception-layer.md) | 感知层 | esp32, sensor, mqtt, rtsp | final | 设备感知层：ESP32多组传感器采集+MQTT上报+RTSP推流 |
| TECH-002 | [data-storage.md](./docs/tech/data-storage.md) | 数据存储 | mysql, influxdb, chroma | final | 数据存储层：MySQL(25表)+InfluxDB(时序)+Chroma(向量) |
| TECH-003 | [auth-permission.md](./docs/tech/auth-permission.md) | 认证权限 | jwt, rbac, security | final | 认证与权限：JWT+RBAC+大棚授权+功能授权+员工单归属 |
| TECH-004 | [alert-engine.md](./docs/tech/alert-engine.md) | 预警引擎 | alert, threshold, lstm | final | 环境预警引擎：三级检测+自定义阈值+LSTM+场景联动 |
| TECH-005 | [ai-layer.md](./docs/tech/ai-layer.md) | AI能力层 | ai, strategy-pattern, rag, knowledge-graph | final | AI能力层：策略模式抽象层+RAG+知识图谱+API切换 |
| TECH-006 | [realtime-comm.md](./docs/tech/realtime-comm.md) | 实时通信 | mqtt, websocket, stomp | final | 实时通信层：MQTT设备通信+WebSocket STOMP数据推送+聊天 |
| TECH-007 | [expert-system.md](./docs/tech/expert-system.md) | 专家系统 | expert, chat, authorization | final | 专家咨询系统：实时聊天+环境快照+7天授权双轨制 |
| TECH-008 | [multimodal-fusion.md](./docs/tech/multimodal-fusion.md) | 多模态融合 | fusion, multimodal, health-score | final | 多模态融合分析：环境60%+视觉40%+天气修正因子 |

### 1.4 API文档 (docs/api/)

| ID | 文件 | 模块 | 标签 | 状态 | 端点数 |
|----|------|------|------|------|--------|
| API-001 | [auth/auth-api.md](./docs/api/auth/auth-api.md) | 认证 | auth, jwt, login | final | 3 |
| API-002 | [greenhouse/greenhouse-api.md](./docs/api/greenhouse/greenhouse-api.md) | 大棚管理 | greenhouse, region | final | 5 |
| API-003 | [device/device-api.md](./docs/api/device/device-api.md) | 设备管理 | device, sensor-config, group | final | 9 |
| API-004 | [sensor/sensor-api.md](./docs/api/sensor/sensor-api.md) | 传感器数据 | sensor, timeseries, export | final | 5 |
| API-005 | [alert/alert-api.md](./docs/api/alert/alert-api.md) | 预警管理 | alert, threshold, rules | final | 9 |
| API-006 | [control/control-api.md](./docs/api/control/control-api.md) | 设备控制 | control, scene, actuator | final | 4 |
| API-007 | [diagnosis/diagnosis-api.md](./docs/api/diagnosis/diagnosis-api.md) | 病虫害诊断 | diagnosis, image, ai | final | 2 |
| API-008 | [qa/qa-api.md](./docs/api/qa/qa-api.md) | AI问答 | qa, rag, voice | final | 3 |
| API-009 | [growth/growth-api.md](./docs/api/growth/growth-api.md) | 长势评估 | growth, assessment, image | final | 3 |
| API-010 | [health/health-api.md](./docs/api/health/health-api.md) | 健康评估 | health, score, multimodal | final | 3 |
| API-011 | [weather/weather-api.md](./docs/api/weather/weather-api.md) | 天气 | weather, forecast | final | 2 |
| API-012 | [chat/chat-api.md](./docs/api/chat/chat-api.md) | 专家聊天 | chat, conversation, snapshot | final | 8 |
| API-013 | [expert/expert-api.md](./docs/api/expert/expert-api.md) | 专家授权 | expert, authorization, data-access | final | 8 |
| API-014 | [employee/employee-api.md](./docs/api/employee/employee-api.md) | 员工管理 | employee, permission, owner | final | 6 |
| API-015 | [admin/admin-api.md](./docs/api/admin/admin-api.md) | 系统管理 | admin, user, ai-config | final | 5 |
| API-016 | [knowledge/knowledge-api.md](./docs/api/knowledge/knowledge-api.md) | 知识库 | knowledge, document, index | final | 5 |
| API-017 | [corpus/corpus-api.md](./docs/api/corpus/corpus-api.md) | 语料管理 | corpus, dialect, audio | final | 4 |
| API-018 | [crop/crop-api.md](./docs/api/crop/crop-api.md) | 生长周期 | crop, cycle, timeline | final | 6 |

> 总计约 **90+ API端点**，另有 **WebSocket STOMP端点** 在 `chat-api.md` 和 `realtime-comm.md` 中定义。

### 1.5 数据库设计文档 (docs/database/)

| ID | 文件 | 类型 | 状态 | 说明 |
|----|------|------|------|------|
| DB-001 | [database-design.md](./docs/database/database-design.md) | db | final | 25张MySQL表完整字段+索引+外键+InfluxDB+Chroma设计 |
| DB-002 | [table-module-mapping.md](./docs/database/table-module-mapping.md) | db | final | 表-模块映射清单：模块→表反向索引（AI速查用） |

### 1.6 规范文档 (docs/standards/)

| ID | 文件 | 类型 | 状态 | 说明 |
|----|------|------|------|------|
| STD-001 | [development-standards.md](./docs/standards/development-standards.md) | standards | final | 编码规范/Git规范/文档规范/代码审查清单 |

### 1.7 部署运维文档 (docs/deployment/)

| ID | 文件 | 类型 | 状态 | 说明 |
|----|------|------|------|------|
| DEPLOY-001 | [deploy-guide.md](./docs/deployment/deploy-guide.md) | deployment | draft | 部署指南：Docker Compose/环境变量/数据库初始化/frp |
| DEPLOY-002 | [runbook.md](./docs/deployment/runbook.md) | deployment | draft | 运维手册：巡检/日志/备份/故障恢复/性能监控 |

### 1.8 测试文档 (docs/test/)

| ID | 文件 | 类型 | 状态 | 说明 |
|----|------|------|------|------|
| TEST-000 | [test-case-template.md](./docs/test/test-case-template.md) | test | final | 测试用例模板 + 3个示例用例 |

### 1.9 开发日志 (devlog/)

| ID | 文件 | 类型 | 状态 | 说明 |
|----|------|------|------|------|
| DEVLOG-000 | [INDEX.md](./devlog/INDEX.md) | devlog | active | 47个任务总表（28已完成 + 19已计划）+ 5个Phase开发顺序 |
| DEVLOG-001 | [DEVLOG.md](./devlog/DEVLOG.md) | devlog | active | 项目总开发日志，按步骤记录开发过程 |
| — | [TASK-TEMPLATE.md](./devlog/TASK-TEMPLATE.md) | devlog | final | 任务日志模板（YAML头部+固定段落） |

---

## 二、按功能模块分类导航

> **AI使用指南**：当用户说"实现XX功能"时，按以下路径快速定位：
> 1. 先读对应 PRD（需求）
> 2. 再读对应 TECH（技术方案）
> 3. 再读对应 API（接口）
> 4. 用 DB-002 表-模块映射找相关表
> 5. 参考 STD-001 开发规范

### 模块1：实时数据看板

| 层级 | 文档 | 路径 |
|------|------|------|
| 📋 需求 | PRD-001 | [docs/prd/dashboard-realtime.md](./docs/prd/dashboard-realtime.md) |
| 🏗️ 技术 | TECH-001, TECH-002, TECH-006 | [感知层](./docs/tech/perception-layer.md) / [数据存储](./docs/tech/data-storage.md) / [实时通信](./docs/tech/realtime-comm.md) |
| 🔌 API | API-003, API-004 | [设备](./docs/api/device/device-api.md) / [传感器数据](./docs/api/sensor/sensor-api.md) |
| 🗄️ 数据库 | DB-001, DB-002 | [数据库设计](./docs/database/database-design.md) / [映射](./docs/database/table-module-mapping.md) |

### 模块2：环境预警中心

| 层级 | 文档 | 路径 |
|------|------|------|
| 📋 需求 | PRD-002 | [docs/prd/alert-center.md](./docs/prd/alert-center.md) |
| 🏗️ 技术 | TECH-004 | [预警引擎](./docs/tech/alert-engine.md) |
| 🔌 API | API-005 | [预警管理](./docs/api/alert/alert-api.md) |

### 模块3：病虫害拍照诊断

| 层级 | 文档 | 路径 |
|------|------|------|
| 📋 需求 | PRD-003 | [docs/prd/disease-diagnosis.md](./docs/prd/disease-diagnosis.md) |
| 🏗️ 技术 | TECH-005 | [AI能力层](./docs/tech/ai-layer.md) |
| 🔌 API | API-007 | [病虫害诊断](./docs/api/diagnosis/diagnosis-api.md) |

### 模块4：作物长势评估

| 层级 | 文档 | 路径 |
|------|------|------|
| 📋 需求 | PRD-004, PRD-010 | [长势评估](./docs/prd/growth-assessment.md) / [生长周期](./docs/prd/crop-cycle.md) |
| 🏗️ 技术 | TECH-001, TECH-008 | [感知层](./docs/tech/perception-layer.md) / [多模态融合](./docs/tech/multimodal-fusion.md) |
| 🔌 API | API-009, API-018 | [长势评估](./docs/api/growth/growth-api.md) / [生长周期](./docs/api/crop/crop-api.md) |

### 模块5：AI智能问答

| 层级 | 文档 | 路径 |
|------|------|------|
| 📋 需求 | PRD-005 | [docs/prd/ai-qa.md](./docs/prd/ai-qa.md) |
| 🏗️ 技术 | TECH-005 | [AI能力层](./docs/tech/ai-layer.md) |
| 🔌 API | API-008 | [AI问答](./docs/api/qa/qa-api.md) |

### 模块6：设备控制

| 层级 | 文档 | 路径 |
|------|------|------|
| 📋 需求 | PRD-006 | [docs/prd/device-control.md](./docs/prd/device-control.md) |
| 🏗️ 技术 | TECH-001, TECH-004 | [感知层](./docs/tech/perception-layer.md) / [预警引擎](./docs/tech/alert-engine.md) |
| 🔌 API | API-006 | [设备控制](./docs/api/control/control-api.md) |

### 模块7：专家咨询

| 层级 | 文档 | 路径 |
|------|------|------|
| 📋 需求 | PRD-007 | [docs/prd/expert-consultation.md](./docs/prd/expert-consultation.md) |
| 🏗️ 技术 | TECH-006, TECH-007 | [实时通信](./docs/tech/realtime-comm.md) / [专家系统](./docs/tech/expert-system.md) |
| 🔌 API | API-012, API-013 | [聊天](./docs/api/chat/chat-api.md) / [专家授权](./docs/api/expert/expert-api.md) |

### 模块8：多角色权限

| 层级 | 文档 | 路径 |
|------|------|------|
| 📋 需求 | PRD-008 | [docs/prd/role-permission.md](./docs/prd/role-permission.md) |
| 🏗️ 技术 | TECH-003 | [认证权限](./docs/tech/auth-permission.md) |
| 🔌 API | API-001, API-014 | [认证](./docs/api/auth/auth-api.md) / [员工管理](./docs/api/employee/employee-api.md) |

### 模块9：知识库与语料

| 层级 | 文档 | 路径 |
|------|------|------|
| 📋 需求 | PRD-012 | [docs/prd/knowledge-corpus.md](./docs/prd/knowledge-corpus.md) |
| 🏗️ 技术 | TECH-005 | [AI能力层](./docs/tech/ai-layer.md) |
| 🔌 API | API-016, API-017 | [知识库](./docs/api/knowledge/knowledge-api.md) / [语料](./docs/api/corpus/corpus-api.md) |

---

## 三、按文档类型分类

### 📋 PRD需求文档（12个）

| PRD-001 | [实时数据看板](./docs/prd/dashboard-realtime.md) | 📋申报书 |
| PRD-002 | [环境预警中心](./docs/prd/alert-center.md) | 📋申报书 + 🔧自定义阈值 |
| PRD-003 | [病虫害拍照诊断](./docs/prd/disease-diagnosis.md) | ⚠️API先行→ResNet替换 |
| PRD-004 | [作物长势评估](./docs/prd/growth-assessment.md) | 📋申报书 + 🆕生长周期 |
| PRD-005 | [AI智能问答](./docs/prd/ai-qa.md) | 📋申报书 + ⚠️方言 |
| PRD-006 | [设备控制](./docs/prd/device-control.md) | 📋申报书 |
| PRD-007 | [专家咨询](./docs/prd/expert-consultation.md) | 🆕v3.0新增 |
| PRD-008 | [多角色权限管理](./docs/prd/role-permission.md) | 🆕v3.0新增 + 🔧v3.1修正 |
| PRD-009 | [多组传感器管理](./docs/prd/multi-sensor.md) | 🆕v3.0新增 |
| PRD-010 | [作物生长周期管理](./docs/prd/crop-cycle.md) | 🆕v3.2新增 |
| PRD-011 | [多模态健康评估](./docs/prd/health-assessment.md) | ⚠️申报书创新点三 |
| PRD-012 | [知识库与语料管理](./docs/prd/knowledge-corpus.md) | ➕架构新增 |

### 🏗️ 技术方案（8个）

| TECH-001 | [设备感知层](./docs/tech/perception-layer.md) | ESP32 + MQTT + RTSP |
| TECH-002 | [数据存储层](./docs/tech/data-storage.md) | MySQL + InfluxDB + Chroma |
| TECH-003 | [认证与权限](./docs/tech/auth-permission.md) | JWT + RBAC + 三维权限 |
| TECH-004 | [环境预警引擎](./docs/tech/alert-engine.md) | 三级检测 + LSTM |
| TECH-005 | [AI能力层](./docs/tech/ai-layer.md) | 策略模式 + RAG + 知识图谱 |
| TECH-006 | [实时通信层](./docs/tech/realtime-comm.md) | MQTT + WebSocket STOMP |
| TECH-007 | [专家咨询系统](./docs/tech/expert-system.md) | 聊天 + 快照 + 7天授权 |
| TECH-008 | [多模态融合分析](./docs/tech/multimodal-fusion.md) | 环境+图像+天气融合 |

### 🔌 API文档（18个，按模块分组）

| 分组 | 文件 | 端点数 | 认证 |
|------|------|--------|------|
| 认证 | [auth-api.md](./docs/api/auth/auth-api.md) | 3 | 注册无认证，其余JWT |
| 大棚 | [greenhouse-api.md](./docs/api/greenhouse/greenhouse-api.md) | 5 | JWT |
| 设备 | [device-api.md](./docs/api/device/device-api.md) | 9 | JWT |
| 传感器 | [sensor-api.md](./docs/api/sensor/sensor-api.md) | 5 | JWT |
| 预警 | [alert-api.md](./docs/api/alert/alert-api.md) | 9 | JWT |
| 控制 | [control-api.md](./docs/api/control/control-api.md) | 4 | JWT |
| 诊断 | [diagnosis-api.md](./docs/api/diagnosis/diagnosis-api.md) | 2 | JWT |
| 问答 | [qa-api.md](./docs/api/qa/qa-api.md) | 3 | JWT |
| 长势 | [growth-api.md](./docs/api/growth/growth-api.md) | 3 | JWT |
| 健康 | [health-api.md](./docs/api/health/health-api.md) | 3 | JWT |
| 天气 | [weather-api.md](./docs/api/weather/weather-api.md) | 2 | JWT |
| 聊天 | [chat-api.md](./docs/api/chat/chat-api.md) | 8 | JWT |
| 专家 | [expert-api.md](./docs/api/expert/expert-api.md) | 8 | JWT |
| 员工 | [employee-api.md](./docs/api/employee/employee-api.md) | 6 | JWT (OWNER) |
| 管理 | [admin-api.md](./docs/api/admin/admin-api.md) | 5 | JWT (ADMIN) |
| 知识库 | [knowledge-api.md](./docs/api/knowledge/knowledge-api.md) | 5 | JWT (ADMIN) |
| 语料 | [corpus-api.md](./docs/api/corpus/corpus-api.md) | 4 | JWT (ADMIN) |
| 生长周期 | [crop-api.md](./docs/api/crop/crop-api.md) | 6 | JWT |

---

## 四、后端模块 ↔ 文档映射（C1-C22）

> 按后端模块编号快速查找所有相关文档。

| 模块 | 名称 | PRD | 技术方案 | API |
|------|------|-----|----------|-----|
| C1 | 用户认证 | PRD-008 | TECH-003 | API-001 |
| C2 | 设备管理 | PRD-009 | TECH-001 | API-003 |
| C3 | 大棚管理 | PRD-008 | TECH-002 | API-002 |
| C4 | MQTT消费 | PRD-001 | TECH-001, TECH-006 | — |
| C5 | 时序数据 | PRD-001, PRD-009 | TECH-002 | API-004 |
| C6 | 环境预警 | PRD-002 | TECH-004 | API-005 |
| C7 | 设备控制 | PRD-006 | TECH-001, TECH-004 | API-006 |
| C8 | 图片诊断 | PRD-003 | TECH-005 | API-007 |
| C9 | RAG问答 | PRD-005 | TECH-005 | API-008 |
| C10 | 语音识别 | PRD-005 | TECH-005 | API-008 |
| C11 | WebSocket推送 | PRD-001, PRD-002 | TECH-006 | — |
| C12 | 场景联动 | PRD-006 | TECH-004 | API-006 |
| C13 | 文件存储 | — | TECH-002 | — |
| C14 | 天气API | PRD-002 | TECH-008 | API-011 |
| C15 | 多模态融合 | PRD-011 | TECH-008 | API-010 |
| C16 | 实时聊天 | PRD-007 | TECH-006, TECH-007 | API-012 |
| C17 | 专家授权 | PRD-007 | TECH-007 | API-013 |
| C18 | 多角色权限 | PRD-008 | TECH-003 | API-001, API-014 |
| C19 | 设备分组 | PRD-009 | TECH-001 | API-003 |
| C20 | 地区管理 | PRD-008 | TECH-002 | API-002 |
| C21 | 自定义阈值 | PRD-002 | TECH-004 | API-005 |
| C22 | 生长周期 | PRD-010 | TECH-002 | API-018 |

---

## 五、关键文档入口（AI开发必读）

| 优先级 | 文档 | 路径 | 何时读 |
|--------|------|------|--------|
| 🔴 必读 | **AI开发规则** | `AI开发规则文档_v2.0.md` | 每次开始开发前 |
| 🔴 必读 | **INDEX.md** | `document/INDEX.md` | 📍 当前文档，首次进入项目 |
| 🔴 必读 | **表-模块映射** | [DB-002](./docs/database/table-module-mapping.md) | 需要操作数据库时 |
| 🟡 按需 | **PRD** | [docs/prd/](./docs/prd/) | 开发某个功能前读对应PRD |
| 🟡 按需 | **技术方案** | [docs/tech/](./docs/tech/) | 需要了解架构细节时 |
| 🟡 按需 | **API文档** | [docs/api/](./docs/api/) | 开发接口或调用接口时 |
| 🟢 参考 | **开发规范** | [STD-001](./docs/standards/development-standards.md) | 提交代码前自检 |
| 🟢 参考 | **开发日志** | [DEVLOG-000](./devlog/INDEX.md) | 开始新任务/查看进度 |

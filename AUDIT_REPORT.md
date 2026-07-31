# Smart-Greenhouse 文档-代码一致性审计报告

> 审计日期: 2026-07-31 | 审计人: AI 代码审计
> 原则: 仅以代码实际存在为准，不采信文档描述

---

## 审计概要

| 统计项 | 数量 |
|--------|------|
| 文档 API 端点总数 | ~100+ |
| 代码匹配 (存在) | ~85 |
| 代码缺失 (不存在) | ~13 |
| 方法/参数不一致 (偏差) | ~5 |
| PRD 功能点核验 | 9 个 PRD 模块 |
| 技术设计核验 | 8 个 TECH 文档 |

---

## 一、API 文档端点对照

### 1.1 admin-api.md — 系统管理 API

| 文档声明 | 代码状态 | 说明 |
|----------|----------|------|
| GET /api/v1/admin/users | 存在 | AdminController |
| GET /api/v1/admin/users/{id} | 存在 | AdminController |
| PUT /api/v1/admin/users/{id} | 存在 | AdminController |
| DELETE /api/v1/admin/users/{id} | 存在 | AdminController |
| GET /api/v1/admin/roles | 存在 | AdminController |
| GET /api/v1/admin/users/regions | **不存在** | 代码无此端点 |
| GET /api/v1/admin/ai/config | **不存在** | 代码无 AI 引擎管理模块 |
| PUT /api/v1/admin/ai/config | **不存在** | 同上 |
| GET /api/v1/admin/ai/status | **不存在** | 同上 |

### 1.2 sensor-api.md — 传感器数据 API

| 文档声明 | 代码状态 | 说明 |
|----------|----------|------|
| GET /api/v1/sensors/realtime | 存在 | |
| GET /api/v1/sensors/history | **偏差** | 文档写 GET，代码为 POST |
| GET /api/v1/sensors/compare | **偏差** | 文档写 GET，代码为 POST |
| GET /api/v1/sensors/aggregate | 存在 | 但参数要求不同 |
| GET /api/v1/sensors/export | 存在 | 文档多 format=csv 参数 |

### 1.3 growth-api.md — 长势评估 API

| 文档声明 | 代码状态 | 说明 |
|----------|----------|------|
| GET /api/v1/growth/latest | **不存在** | 整个 growth 模块不存在 |
| GET /api/v1/growth/history | **不存在** | 同上 |
| GET /api/v1/growth/images | **不存在** | 同上 |

### 1.4 employee-api.md — 员工管理 API

| 文档声明 | 代码状态 | 说明 |
|----------|----------|------|
| POST /api/v1/owner/employees | 存在 | PermissionController |
| GET /api/v1/owner/employees | 存在 | |
| PUT /api/v1/owner/employees/{id} | **不存在** | 代码仅有 POST/GET/DELETE |
| DELETE /api/v1/owner/employees/{id} | 存在 | |
| PUT /api/v1/owner/employees/{id}/permissions | 存在 | |
| GET /api/v1/owner/employees/{id}/permissions | 存在 | |
| GET /api/v1/worker/permissions | 存在 | WorkerPermissionController |
| GET /api/v1/worker/greenhouses | 存在 | |

### 1.5 全部匹配的 API 文档

以下文档声明的所有端点均在代码中存在:
auth-api.md, greenhouse-api.md, alert-api.md, chat-api.md, control-api.md,
corpus-api.md, crop-api.md, device-api.md, diagnosis-api.md, expert-api.md,
health-api.md, knowledge-api.md, qa-api.md, weather-api.md

---

## 二、PRD 功能需求对照

### PRD-005: AI 智能问答 (ai-qa.md)

| 功能点 | 代码状态 | 说明 |
|--------|----------|------|
| 文字输入问答 (RAG) | 存在 | RagQaService + ChromaRetrievalService |
| 语音输入问答 (方言识别) | 存在 | VoiceQaService |
| 答案引用来源展示 | **不存在** | |
| TTS 语音播报 | **不存在** | |
| 上下文连续对话 | 部分 | 未确认多轮对话 |

### PRD-002: 环境预警中心 (alert-center.md)

| 功能点 | 代码状态 | 说明 |
|--------|----------|------|
| 三级预警展示 | 存在 | |
| 预警推送通知 | 部分 | WebSocket 有，系统通知栏无 |
| 预警历史记录 | 存在 | |
| 自定义预警阈值 | 存在 | |
| LSTM 时序预测 | **不存在** | |
| 天气预报结合预警 | **不存在** | |

### PRD-004: 作物长势评估 (growth-assessment.md)

| 功能点 | 代码状态 | 说明 |
|--------|----------|------|
| 长势评估 | **不存在** | 无后端 controller/service |
| 摄像头定时截帧 | **不存在** | 无 FFmpeg 集成 |
| 生长阶段识别 | **不存在** | |
| 长势历史对比 | **不存在** | |

### PRD-003: 病虫害诊断 (disease-diagnosis.md)

| 功能点 | 代码状态 | 说明 |
|--------|----------|------|
| 拍照上传诊断 | 存在 | |
| 诊断结果展示 | 存在 | |
| AI Provider 策略切换 | **不存在** | |

### PRD-006: 健康评估 (health-assessment.md)

| 功能点 | 代码状态 | 说明 |
|--------|----------|------|
| 健康评分 | 存在 | |
| 评分历史 | 存在 | |
| 环境+长势融合评分 | 部分 | 引用 GrowthAssessment 但 growth 无后端 |

### PRD-008: 多角色权限管理 (role-permission.md)

| 功能点 | 代码状态 | 说明 |
|--------|----------|------|
| 4 角色体系 | 存在 | |
| 棚主管理员工 | 存在 | |
| 员工权限矩阵 | 存在 | |
| 五级地址 | **偏差** | 代码仅到三级 (province/city/district) |

### PRD-007 / PRD-009 / PRD-010: 全部功能点存在

知识库与语料管理、多组传感器、专家在线咨询 — 全部匹配

---

## 三、技术设计文档对照

| 技术文档 | 核心声明 | 代码状态 |
|----------|----------|----------|
| ai-layer.md | AI Provider 策略模式 | **不存在** |
| ai-layer.md | Spring AI 集成 | **不存在** |
| alert-engine.md | LSTM 时序预测 | **不存在** |
| multimodal-fusion.md | 多模态融合分析 | **不存在** |
| perception-layer.md | 多传感器 MQTT 接入 | 部分 |
| data-storage.md | Chroma 向量数据库 | 存在 |
| realtime-comm.md | STOMP WebSocket + MQTT | 存在 |
| auth-permission.md | JWT + RBAC + AOP | 存在 |
| expert-system.md | 专家授权流程 | 存在 |

---

## 四、代码缺失清单 (按优先级)

| 优先级 | 缺失项 | 影响范围 |
|--------|--------|----------|
| P0 | growth 模块 (/api/v1/growth/*) | 文档声明完整 API + PRD，代码完全不存在 |
| P0 | AI 引擎配置 (/api/v1/admin/ai/*) | admin-api.md 3 个端点不存在 |
| P1 | AI Provider 策略模式 | tech/ai-layer.md 核心架构未实现 |
| P1 | LSTM 时序预测 | alert-center PRD 核心功能 |
| P2 | PUT /api/v1/owner/employees/{id} | 多声明一个更新接口 |
| P2 | 五级地址 (乡镇/村字段) | 代码只到三级 |
| P2 | TTS 语音播报 | ai-qa PRD 功能 |
| P2 | 答案引用来源展示 | ai-qa PRD 功能 |
| P3 | sensor HTTP 方法不一致 | 文档 GET vs 代码 POST |

### 统计汇总

| 类型 | 数量 |
|------|------|
| 代码完全不存在 (端点/模块) | 13 |
| HTTP 方法不一致 | 2 |
| 参数差异 | 2 |
| 功能缺失 | 6 |

---

> **审计结论**: 文档覆盖范围超出实际代码实现。后端 19 个模块中 growth 模块完全缺失。
> AI 抽象层、LSTM 预测等核心架构在代码中不存在。建议优先补齐 growth 模块。

> 详细代码索引: [CODE_INDEX.md](../Smart-Greenhouse/CODE_INDEX.md)
---

## 补充说明（2026-08-01 复核）

### 关于 AI Provider 策略模式的审计结果修正

原审计报告标注 `tech/ai-layer.md` 描述的 AI Provider 策略模式"不存在"——此判断**有误**。

经复核，代码已在 `backend/.../ai/` 包下完整实现：

- 4 个策略接口：`DiseaseRecognitionProvider`, `SpeechRecognitionProvider`, `EmbeddingProvider`, `LlmProvider`
- 真实实现：`BaiduRecognitionProvider`, `XunfeiSpeechProvider`, `SiliconFlowEmbeddingProvider`, `DeepSeekLlmProvider`
- 预留实现：`ResNetRecognitionProvider`, `WhisperSpeechProvider`
- Mock 实现：4 个对应 Mock Provider
- 条件注入：通过 `@ConditionalOnProperty` + `application-dev.yml` 配置切换

原报告中标注为"不存在"的核心架构**实际已存在且完整**。

### 关于 growth 模块

原报告标注 growth 模块"不存在"——已在本次迭代中补齐：
- 新建 `GrowthController`（3 个端点）
- 新建 `GrowthService`（latest / history / images 查询）
- `GrowthAssessmentRepository` 新增查询方法

### AI 引擎配置

原报告标注 `/api/v1/admin/ai/*` 不存在——已在本次迭代中补齐：
- 新建 `AdminAiController`（config / status 端点）
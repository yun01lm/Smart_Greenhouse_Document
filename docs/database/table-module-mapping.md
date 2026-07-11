---
id: DB-002
title: 表-模块映射清单
type: db
module: database
tags: [mapping, module]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs: [docs/database/database-design.md]
---

## 概述

本文档为 AI 开发助手速查表，根据后端模块编号（C1-C22）或 API 分组快速定位需要操作的 MySQL 表。所有 25 张表已完整覆盖。

## 表-模块映射

| 表名 | 后端模块 | API 分组 | 说明 |
|------|----------|----------|------|
| `users` | C1 认证、C18 权限 | `/auth`、`/admin/users` | 用户注册/登录/角色管理 |
| `user_addresses` | C20 地区管理 | `/auth`（注册时填写） | 用户五级地区地址 |
| `greenhouses` | C3 大棚管理 | `/greenhouses` | 大棚CRUD、地区分布统计 |
| `device_groups` | C19 设备分组 | `/device-groups` | 传感器组CRUD、区域标注 |
| `devices` | C2 设备管理 | `/devices` | ESP32注册/绑定/心跳检测 |
| `sensors` | C2 设备管理 | `/devices/{id}/sensors` | 传感器配置、系统默认阈值 |
| `actuators` | C2 设备管理、C7 控制 | `/devices/{id}/actuators`、`/control` | 执行器配置、控制指令 |
| `employee_permissions` | C18 权限 | `/owner/employees/{id}/permissions` | 员工功能权限矩阵 |
| `alert_rules` | C6 预警引擎 | `/alerts/rules` | 预警规则定义 |
| `alerts` | C6 预警引擎、C11 WebSocket | `/alerts` | 预警记录读写、推送 |
| `scenes` | C12 场景联动、C7 控制 | `/control/scenes` | 场景联动规则 |
| `diagnostic_records` | C8 图片诊断 | `/diagnosis` | 诊断记录、转专家标记 |
| `qa_records` | C9 RAG问答 | `/qa` | 问答历史记录 |
| `control_logs` | C7 设备控制 | `/control`（写入日志） | 控制操作审计日志 |
| `knowledge_documents` | 知识库管理 | `/knowledge/documents` | 文档元数据、向量化状态 |
| `dialect_corpus` | 语料管理 | `/corpus` | 方言语料管理 |
| `growth_assessments` | C8 诊断、C22 生长周期 | `/growth` | 长势评估记录 |
| `health_assessments` | C15 多模态融合 | `/health` | 综合健康评分 |
| `weather_cache` | C14 天气API | `/weather` | 天气数据缓存 |
| `crop_cycles` | C22 作物生长周期 | `/crop-cycles` | 种植记录、阶段管理 |
| `chat_conversations` | C16 实时聊天 | `/chat/conversations` | 专家咨询对话 |
| `chat_messages` | C16 实时聊天、C11 WebSocket | `/chat/messages` | 聊天消息持久化 |
| `data_authorizations` | C17 专家授权 | `/expert/authorize` | 环境数据授权管理 |
| `expert_availability` | C16 聊天、C17 授权 | `/expert/status` | 专家在线状态 |
| `user_alert_thresholds` | C21 自定义阈值 | `/alerts/thresholds` | 用户自定义预警值 |

## 模块 → 表（反向索引）

| 后端模块 | 需要操作的表 |
|----------|-------------|
| C1 用户认证 | `users` |
| C2 设备管理 | `devices`, `sensors`, `actuators` |
| C3 大棚管理 | `greenhouses` |
| C4 MQTT消费 | → InfluxDB `sensor_data`（不写MySQL） |
| C5 时序数据 | → InfluxDB `sensor_data`（只读） |
| C6 预警引擎 | `alert_rules`, `alerts`, `user_alert_thresholds`, `sensors` |
| C7 设备控制 | `actuators`, `control_logs`, `scenes` |
| C8 图片诊断 | `diagnostic_records`, `growth_assessments` |
| C9 RAG问答 | `qa_records`, → Chroma `agricultural_knowledge` |
| C10 语音识别 | `qa_records`（写入问答记录） |
| C11 WebSocket推送 | `alerts`（读预警推APP） |
| C12 场景联动 | `scenes`, `control_logs` |
| C13 文件存储 | 本地文件系统（不操作表） |
| C14 天气API | `weather_cache` |
| C15 多模态融合 | `health_assessments`, `growth_assessments`, `alerts` |
| C16 实时聊天 | `chat_conversations`, `chat_messages`, `expert_availability` |
| C17 专家授权 | `data_authorizations`, `chat_conversations` |
| C18 多角色权限 | `users`, `employee_permissions` |
| C19 设备分组 | `device_groups`, `devices`（更新 group_id） |
| C20 地区管理 | `user_addresses`, `greenhouses`, `users` |
| C21 自定义阈值 | `user_alert_thresholds` |
| C22 作物生长周期 | `crop_cycles`, `growth_assessments` |

## 跨库读写速查

| 数据库 | 写操作模块 | 读操作模块 |
|--------|-----------|-----------|
| MySQL | C1, C2, C3, C6, C7, C8, C9, C10, C12, C14, C15, C16, C17, C18, C19, C20, C21, C22 | 所有模块 |
| InfluxDB | C4（MQTT消费写入） | C5（时序查询）, C6（预警检测）, C15（融合分析）, C17（授权数据查看） |
| Chroma | 知识库管理（向量化写入） | C9（RAG检索） |
| 本地文件系统 | C8（诊断图片）, C10（语音文件）, C13（通用文件） | C8, C10, C13 |

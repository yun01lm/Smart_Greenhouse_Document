---
id: TASK-C17
title: 专家授权模块开发
module: C17 专家授权
type: task
priority: high
status: completed
created: 2026-07-13
started: 2026-07-13
completed: 2026-07-13
assignee: AI助手
related_docs:
  - DEVLOG-步骤16
tags: [expert, authorization, 7-day, online-status]
---

# 专家授权模块开发

---

## 任务描述

### 背景

C17 专家授权是专家咨询系统的权限管理模块。通过"双轨制"设计（环境快照+数据授权）平衡专家辅助诊断的便利性和用户隐私保护。授权有效期7天，用户可随时撤销。

### 目标

完成专家列表查询、数据授权管理（请求/同意/拒绝/撤销）和专家在线状态管理。

### 范围

仅后端代码，不涉及前端APP/Web界面。

---

## 验收标准

1. [x] DataAuthorization 实体创建完成，与 DB 表 23 一致
2. [x] ExpertAvailability 实体创建完成，与 DB 表 24 一致
3. [x] 授权请求：防重复有效授权检查
4. [x] 同意授权：自动设置7天过期时间
5. [x] 撤销授权：记录撤销人和撤销时间
6. [x] 专家列表：含在线状态，支持筛选
7. [x] 9个 API 端点与接口文档一致

---

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `entity/DataAuthorization.java` | 新建 | DB表23，5种状态，7天有效期 |
| `entity/ExpertAvailability.java` | 新建 | DB表24，在线状态+并发控制 |
| `repository/DataAuthorizationRepository.java` | 新建 | 多维度查询 |
| `repository/ExpertAvailabilityRepository.java` | 新建 | 按专家ID查询 |
| `module/expert/service/ExpertService.java` | 新建 | 专家列表+授权管理+在线状态 |
| `module/expert/controller/AuthorizationController.java` | 新建 | 8个授权端点 |
| `module/expert/controller/ExpertController.java` | 新建 | 1个专家列表端点 |
| `module/expert/dto/AuthorizationResponse.java` | 新建 | 授权响应（含remainingDays） |

---

## 完成结果

专家授权模块支持完整的授权生命周期：专家发起请求→用户同意（7天有效期）→自动过期或用户主动撤销。双轨制：环境快照一次性使用 + 数据授权7天持续查看。专家在线状态管理支持上线/离线切换。

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-13
- 关联DEVLOG：步骤16
- 授权过期定时扫描任务 Phase 5 实现

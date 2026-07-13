---
id: TASK-C21
title: 自定义预警阈值开发
module: C21 自定义阈值
type: task
priority: high
status: completed
created: 2026-07-13
started: 2026-07-13
completed: 2026-07-13
assignee: AI助手
related_docs:
  - DEVLOG-步骤12
tags: [c21-自定义阈值]
---

# 自定义预警阈值开发

---

## 任务描述

### 背景

C21 自定义阈值是智慧大棚AIoT系统的核心后端模块，属于Phase 2/3开发计划。

### 目标

完成C21 自定义阈值的全部后端代码开发，包括实体类、Repository、Service、Controller和DTO。

### 范围

仅后端代码，不涉及前端APP/Web界面。

---

## 验收标准

1. [x] 实体类创建完成，与数据库设计一致
2. [x] Repository层实现，支持基本CRUD和业务查询
3. [x] Service层实现，包含核心业务逻辑和权限校验
4. [x] Controller层实现，API端点与接口文档一致
5. [x] DTO定义完整，包含@Valid校验
6. [x] API路径使用 /api/v1/ 前缀 + kebab-case

---

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `entity/UserAlertThreshold.java` | 新建 | DB第25号表，minThreshold/maxThreshold
| `module/alert/service/AlertThresholdService.java` | 新建 | 设置（Upsert）/列表/删除
- 2个DTO：ThresholdRequest、ThresholdResponse
- 集成到AlertEngine：检查系统规则的同时检查用户自定义阈值

---

## 完成结果

用户可为每个大棚的每种传感器设置自定义告警阈值，与系统规则并行生效

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-13
- 关联DEVLOG：步骤12

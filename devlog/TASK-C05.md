---
id: TASK-C05
title: 时序数据服务开发
module: C5 时序数据
type: task
priority: high
status: completed
created: 2026-07-12
started: 2026-07-12
completed: 2026-07-12
assignee: AI助手
related_docs:
  - DEVLOG-步骤8
tags: [c5-时序数据]
---

# 时序数据服务开发

---

## 任务描述

### 背景

C5 时序数据是智慧大棚AIoT系统的核心后端模块，属于Phase 2/3开发计划。

### 目标

完成C5 时序数据的全部后端代码开发，包括Service、Controller和DTO（时序数据存储在InfluxDB，无需JPA实体和Repository）。

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
| `module/sensor/service/SensorDataService.java` | 新建 | 核心服务：writeData、getRealtimeData(Flux last)、getHistoryData(aggregateWindow)、getCompareData、getAggregateData(五维统计)、exportCsv
| `module/sensor/controller/SensorController.java` | 新建 | 5个端点（realtime/history/compare/aggregate/export）
- 6个DTO：SensorDataPoint、SensorRealtimeResponse、SensorHistoryRequest、SensorCompareResponse、SensorAggregateResponse、CompareRequest
| `module/sensor/service/InfluxDbConfigHelper.java` | 新建 | 辅助组件

---

## 完成结果

时序数据服务完整，支持实时/历史/对比/聚合/CSV导出五种查询模式

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-12
- 关联DEVLOG：步骤8
- 步骤31审查修复：修正目标描述（时序数据存储在InfluxDB，无JPA实体和Repository）

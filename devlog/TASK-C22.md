---
id: TASK-C22
title: 作物生长周期管理开发
module: C22 作物生长周期
type: task
priority: high
status: completed
created: 2026-07-13
started: 2026-07-13
completed: 2026-07-13
assignee: AI助手
related_docs:
  - DEVLOG-步骤15
tags: [crop, lifecycle, growth-stage, timeline]
---

# 作物生长周期管理开发

---

## 任务描述

### 背景

C22 作物生长周期管理是智慧大棚AIoT系统的种植管理模块。为每个大棚的每轮种植建立完整的生长周期记录，从种植日期到收获日期的全过程管理。

### 目标

完成种植记录的增删改查、生长阶段自动估算与手动修正、生长时间线查看功能。

### 范围

仅后端代码，不涉及前端APP/Web界面。

---

## 验收标准

1. [x] CropCycle 实体创建完成，与 DB 表 20 一致
2. [x] 支持创建种植记录（防重复：同一大棚不能有多个ACTIVE周期）
3. [x] 阶段自动估算（育苗期0-20天→生长期→开花期→结果期→收获期81天+）
4. [x] 支持手动覆盖阶段（MANUAL模式）
5. [x] 支持标记完成收获
6. [x] 生长时间线查看
7. [x] 7个API端点与接口文档一致

---

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `entity/CropCycle.java` | 新建 | DB表20，5阶段+自动估算+手动修正 |
| `repository/CropCycleRepository.java` | 新建 | 按大棚+状态查询，防重复检查 |
| `module/crop/service/CropCycleService.java` | 新建 | CRUD+阶段管理+时间线生成 |
| `module/crop/controller/CropCycleController.java` | 新建 | 7个端点（含PATCH /stage和/complete） |
| `module/crop/dto/CropCycleRequest.java` | 新建 | 创建/更新请求DTO |
| `module/crop/dto/CropCycleResponse.java` | 新建 | 响应DTO（含daysSincePlanting） |
| `module/crop/dto/CropTimelineResponse.java` | 新建 | 时间线DTO（含TimelineEvent列表） |

---

## 完成结果

作物生长周期模块支持完整的种植→生长→收获生命周期管理。阶段自动估算根据种植天数推算当前阶段（育苗期→生长期→开花期→结果期→收获期），用户可随时手动覆盖。时间线汇总种植、阶段变更、收获等关键事件。

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-13
- 关联DEVLOG：步骤15
- Phase 4 将关联长势评估记录(growth_assessments)和预警记录(alerts)丰富时间线内容

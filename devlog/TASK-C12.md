---
id: TASK-C12
title: 场景联动引擎开发
module: C12 场景联动
type: task
priority: high
status: completed
created: 2026-07-12
started: 2026-07-12
completed: 2026-07-12
assignee: AI助手
related_docs:
  - DEVLOG-步骤10
tags: [c12-场景联动]
---

# 场景联动引擎开发

---

## 任务描述

### 背景

C12 场景联动是智慧大棚AIoT系统的核心后端模块，属于Phase 2/3开发计划。

### 目标

完成C12 场景联动的全部后端代码开发，包括实体类、Repository、Service、Controller和DTO。

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
| `entity/Scene.java` | 新建 | DB第11号表，actionsJson字段，触发条件JSON（Phase 2用）
| `repository/SceneRepository.java` | 新建 | 按大棚/启用状态查询
| `module/control/controller/SceneController.java` | 新建 | 5个端点（列表/创建/更新/删除/执行）
| `module/control/service/SceneService.java` | 新建 | CRUD+手动执行（逐一调用ControlService，标记source=SCENE）
- 2个DTO：SceneRequest（含SceneAction子类）、SceneResponse
- 修改：ControlLogRepository（按场景查询日志）

---

## 完成结果

场景联动支持一键批量控制多设备，Phase 1手动执行，Phase 2自动触发

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-12
- 关联DEVLOG：步骤10

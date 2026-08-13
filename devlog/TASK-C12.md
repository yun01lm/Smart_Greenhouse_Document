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

---

## 后续变更（R36：预警联动场景自动执行 + 设备控制记录，2026-08-14）

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `AlertEngine.java` | 修改 | 预警触发后联动执行规则绑定的场景；10 分钟冷却防抖；异步执行不阻塞 MQTT 回调 |
| `SceneService.java` | 修改 | 新增 `executeSceneByAlert`（系统身份执行场景，日志 source=ALERT） |
| `ControlService.java` | 修改 | 新增 `controlDeviceBySystem`（系统控制、日志 userId=null 显示"系统"）；新增 `getGreenhouseLogs` 分页查询（权限收口+场景名）；`ControlLogResponse` 新增 sceneName |
| `ControlController.java` | 修改 | `/control/logs` 扩展 greenhouseId 分页查询（兼容原 deviceId） |
| `ControlLogRepository.java` | 修改 | 新增按设备ID集+来源分页查询 |
| `web/src/views/alerts/AlertRulePage.vue` | 修改 | 预警规则弹窗新增"联动场景"下拉；表格新增"联动场景"列 |
| `web/src/api/control.js` | 新增 | 场景列表/执行场景 API 封装 |

> 验证：后端 mvn compile、APP gradle 离线编译、Web npm build 均通过。

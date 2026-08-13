---
id: TASK-C06
title: 环境预警引擎开发
module: C6 环境预警
type: task
priority: high
status: completed
created: 2026-07-13
started: 2026-07-13
completed: 2026-07-13
assignee: AI助手
related_docs:
  - DEVLOG-步骤12
tags: [c6-环境预警]
---

# 环境预警引擎开发

---

## 任务描述

### 背景

C6 环境预警是智慧大棚AIoT系统的核心后端模块，属于Phase 2/3开发计划。

### 目标

完成C6 环境预警的全部后端代码开发，包括实体类、Repository、Service、Controller和DTO。

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
| `entity/AlertRule.java` | 新建 | DB第9号表，4种规则类型(THRESHOLD/TREND/COMPOSITE/WEATHER)、3级告警
| `entity/Alert.java` | 新建 | DB第10号表，level/title/content/sensorValue/readStatus
- 3个Repository：AlertRuleRepository（按大棚+传感器查询）、AlertRepository（分页+未读统计）、UserAlertThresholdRepository
| `module/alert/service/AlertEngine.java` | 新建 | 核心引擎：MQTT数据→阈值比对→生成Alert→WebSocket推送
| `module/alert/service/AlertRuleService.java` | 新建 | 规则CRUD，50条上限
| `module/alert/controller/AlertController.java` | 新建 | 9个端点（告警列表/已读/规则CRUD/阈值CRUD）
- 5个DTO：AlertRuleRequest、AlertRuleResponse、AlertResponse、ThresholdRequest、ThresholdResponse
- 修改：MqttSubscriber（加alertEngine.check()调用）

---

## 完成结果

预警检测闭环：MQTT数据→AlertEngine比对→超限生成Alert→WebSocket实时推送

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-13
- 关联DEVLOG：步骤12
- **步骤35 编译修复（2026-07-14）**：AlertEngine.java `level` 变量类型 `Alert.AlertLevel` → `AlertRule.AlertLevel`
- **步骤37 架构整改（2026-07-14）**：
  - 新建 `AlertService.java`：从 AlertController 提取告警记录管理逻辑（listAlerts / markAsRead / getGreenhouseName）
  - 修改 `AlertController.java`：移除 AlertRepository/GreenhouseRepository 直接注入，改为注入 AlertService
  - 修改原因：Controller 直接操作 Repository 违反分层约定，规则和阈值已走 Service，告警记录管理也应收敛

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

---
id: TASK-C07
title: 设备控制模块开发
module: C7 设备控制
type: task
priority: high
status: completed
created: 2026-07-12
started: 2026-07-12
completed: 2026-07-12
assignee: AI助手
related_docs:
  - DEVLOG-步骤10
tags: [c7-设备控制]
---

# 设备控制模块开发

---

## 任务描述

### 背景

C7 设备控制是智慧大棚AIoT系统的核心后端模块，属于Phase 2/3开发计划。

### 目标

完成C7 设备控制的全部后端代码开发，包括实体类、Repository、Service、Controller和DTO。

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
| `module/control/service/ControlService.java` | 新建 | 校验CONTROLLER类型/在线状态/权限、MQTT下发指令、自动记录日志
| `module/control/controller/ControlController.java` | 新建 | 2个端点（POST控制、GET日志）
| `entity/ControlLog.java` | 新建 | DB第14号表，action/source/success/failReason
| `repository/ControlLogRepository.java` | 新建 | JPA Repository，按greenhouseId+时间范围查询
- 2个DTO：ControlRequest、ControlLogResponse

---

## 完成结果

设备控制链路完整：API→MQTT下发→ESP32执行→日志记录

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-12
- 关联DEVLOG：步骤10
- 步骤31审查修复：关键修改表中补充遗漏的 ControlLogRepository.java
- **步骤37 架构整改（2026-07-14）**：
  - 修改 `ControlService.java`：sendMqttCommand() 中 Topic 字符串拼接替换为 MqttTopicConstants.deviceControlTopic()
  - 修改 `Device.java`：@PrePersist 中 mqttTopic 生成替换为 MqttTopicConstants.deviceDataTopic()
  - 修改原因：MQTT Topic 统一管理，消除分散在各处的字符串拼接

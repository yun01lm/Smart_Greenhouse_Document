---
id: TASK-C04
title: MQTT消费模块开发
module: C4 MQTT消费
type: task
priority: high
status: completed
created: 2026-07-12
started: 2026-07-12
completed: 2026-07-12
assignee: AI助手
related_docs:
  - DEVLOG-步骤8
tags: [c4-mqtt消费]
---

# MQTT消费模块开发

---

## 任务描述

### 背景

C4 MQTT消费是智慧大棚AIoT系统的核心后端模块，属于Phase 2/3开发计划。

### 目标

完成C4 MQTT消费的全部后端代码开发，包括实体类、Repository、Service、Controller和DTO。

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
| `config/MqttConfig.java` | 新建 | MQTT客户端配置，支持认证/自动重连
| `module/mqtt/MqttSubscriber.java` | 新建 | @PostConstruct订阅 greenhouse/+/device/+，解析JSON写入InfluxDB+更新设备状态
| `config/InfluxDbConfig.java` | 新建 | InfluxDB 2.x客户端配置

---

## 完成结果

MQTT消费链路：ESP32上报→MQTT Broker→MqttSubscriber解析→InfluxDB存储+设备状态更新

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-12
- 关联DEVLOG：步骤8
- **步骤35 编译修复（2026-07-14）**：MqttSubscriber.java 补充 `MqttMessage` import
- **步骤37 架构整改（2026-07-14）**：
  - 新建 `MqttTopicConstants.java`：统一管理所有 MQTT Topic（DEVICE_DATA_WILDCARD 常量 + deviceDataTopic/deviceControlTopic 工厂方法 + DEFAULT_QOS）
  - 修改 `MqttConfig.java`：移除本地 SUBSCRIBE_TOPIC 常量
  - 修改 `MqttSubscriber.java`：使用 MqttTopicConstants.DEVICE_DATA_WILDCARD 和 DEFAULT_QOS
  - 修改原因：MQTT Topic 在 MqttConfig/MqttSubscriber/ControlService/Device 四处分散定义，字符串拼接重复

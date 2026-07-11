---
id: API-006
title: 设备控制 API
type: api
module: control
tags: [设备控制, 场景联动, 执行器, 继电器]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

设备控制模块负责向大棚内的执行器（通风风机、卷帘/遮阳网、滴灌阀门、补光灯）下发开关指令。支持单设备控制和场景联动控制两种模式。所有控制指令通过 MQTT 下发到指定 ESP32 设备，经权限校验后方可执行。专家角色无设备控制权限。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| POST | `/api/v1/control/actuator` | 是 | OWNER/WORKER(需授权) | 控制单个设备 |
| GET | `/api/v1/control/scenes` | 是 | OWNER/WORKER | 场景列表 |
| POST | `/api/v1/control/scenes` | 是 | OWNER | 创建场景 |
| POST | `/api/v1/control/scenes/{id}/execute` | 是 | OWNER/WORKER(需授权) | 执行场景 |

## 请求/响应示例

### 1. 控制单个设备

**请求：**
```json
POST /api/v1/control/actuator
Authorization: Bearer <token>
Content-Type: application/json

{
  "actuatorId": 1,
  "action": "ON",
  "greenhouseId": 1
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "actuatorId": 1,
    "actuatorName": "通风风机",
    "action": "ON",
    "previousState": "OFF",
    "currentState": "ON",
    "deviceId": 1,
    "groupId": 1,
    "zoneLabel": "东侧",
    "executedAt": "2026-07-11T10:30:00"
  }
}
```

### 2. 获取场景列表

**请求：**
```
GET /api/v1/control/scenes?greenhouseId=1
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "list": [
      {
        "id": 1,
        "name": "高温通风",
        "triggerCondition": {"sensorType": "TEMP", "operator": ">", "value": 35.0},
        "actionsJson": [
          {"actuatorId": 1, "action": "ON", "description": "开启通风风机"},
          {"actuatorId": 2, "action": "ON", "description": "展开遮阳网"}
        ],
        "greenhouseId": 1,
        "enabled": true
      },
      {
        "id": 2,
        "name": "定时灌溉",
        "triggerCondition": {"cron": "0 0 6,18 * * ?"},
        "actionsJson": [
          {"actuatorId": 3, "action": "ON", "duration": 600, "description": "开启滴灌10分钟"}
        ],
        "greenhouseId": 1,
        "enabled": true
      }
    ]
  }
}
```

### 3. 创建场景

**请求：**
```json
POST /api/v1/control/scenes
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "夜间补光",
  "triggerCondition": {"sensorType": "LIGHT", "operator": "<", "value": 5000},
  "actionsJson": [
    {"actuatorId": 4, "action": "ON", "description": "开启补光灯"}
  ],
  "greenhouseId": 1,
  "enabled": true
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 3,
    "name": "夜间补光",
    "greenhouseId": 1,
    "enabled": true,
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

### 4. 执行场景

**请求：**
```json
POST /api/v1/control/scenes/1/execute
Authorization: Bearer <token>
Content-Type: application/json

{
  "greenhouseId": 1
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "sceneId": 1,
    "sceneName": "高温通风",
    "executedAt": "2026-07-11T10:30:00",
    "results": [
      {"actuatorId": 1, "actuatorName": "通风风机", "action": "ON", "status": "SUCCESS"},
      {"actuatorId": 2, "actuatorName": "卷帘/遮阳网", "action": "ON", "status": "SUCCESS"}
    ]
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 4003 | 404 | 设备不存在 |
| 4004 | 400 | 设备已离线 |
| 3002 | 403 | 无该大棚访问权限 |
| 3003 | 403 | 无该功能使用权限 |
| 1001 | 400 | 参数错误 |

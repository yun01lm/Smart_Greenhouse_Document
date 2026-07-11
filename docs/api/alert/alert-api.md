---
id: API-005
title: 预警管理 API
type: api
module: alert
tags: [预警, 阈值, 告警, 规则, 自定义阈值]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

预警管理模块提供环境预警的完整功能，包括预警记录的查询与已读标记、预警规则的CRUD管理，以及用户自定义预警阈值的管理。预警引擎优先使用用户自定义阈值，无自定义时回退到系统默认值。预警可精确定位到传感器组级别。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

### 预警记录

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/alerts` | 是 | OWNER/WORKER/ADMIN | 预警列表（分页/筛选/含组定位） |
| PUT | `/api/v1/alerts/{id}/read` | 是 | OWNER/WORKER | 标记已读 |

### 预警规则

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/alerts/rules` | 是 | OWNER/WORKER | 预警规则列表 |
| POST | `/api/v1/alerts/rules` | 是 | OWNER/ADMIN | 创建预警规则 |
| PUT | `/api/v1/alerts/rules/{id}` | 是 | OWNER/ADMIN | 更新规则 |

### 自定义预警阈值

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/alerts/thresholds` | 是 | OWNER/WORKER | 用户自定义预警阈值列表 |
| POST | `/api/v1/alerts/thresholds` | 是 | OWNER | 设置自定义预警阈值 |
| PUT | `/api/v1/alerts/thresholds/{id}` | 是 | OWNER | 更新自定义预警阈值 |
| DELETE | `/api/v1/alerts/thresholds/{id}` | 是 | OWNER | 删除自定义预警阈值（回退系统默认） |

## 请求/响应示例

### 1. 获取预警列表

**请求：**
```
GET /api/v1/alerts?greenhouseId=1&level=WARNING&readStatus=0&page=1&size=20
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
        "id": 56,
        "greenhouseId": 1,
        "groupId": 2,
        "zoneLabel": "西侧",
        "level": "WARNING",
        "title": "温度过高预警",
        "content": "西侧区域温度达到35.2°C，超过预警上限35°C",
        "sensorType": "TEMP",
        "sensorValue": 35.2,
        "thresholdValue": 35.0,
        "readStatus": 0,
        "createdAt": "2026-07-11T10:30:00"
      }
    ],
    "total": 1,
    "page": 1,
    "size": 20
  }
}
```

### 2. 创建预警规则

**请求：**
```json
POST /api/v1/alerts/rules
Authorization: Bearer <token>
Content-Type: application/json

{
  "greenhouseId": 1,
  "groupId": null,
  "sensorType": "TEMP",
  "ruleType": "THRESHOLD",
  "conditionJson": {"max": 35.0, "min": 10.0, "duration": "5m"},
  "alertLevel": "WARNING",
  "sceneId": 1,
  "enabled": true
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 10,
    "greenhouseId": 1,
    "sensorType": "TEMP",
    "ruleType": "THRESHOLD",
    "alertLevel": "WARNING",
    "enabled": true,
    "createdAt": "2026-07-11T10:00:00"
  }
}
```

### 3. 设置自定义预警阈值

**请求：**
```json
POST /api/v1/alerts/thresholds
Authorization: Bearer <token>
Content-Type: application/json

{
  "greenhouseId": 1,
  "groupId": null,
  "sensorType": "TEMP",
  "minThreshold": 12.0,
  "maxThreshold": 32.0,
  "enabled": true
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 5,
    "userId": 1,
    "greenhouseId": 1,
    "sensorType": "TEMP",
    "minThreshold": 12.0,
    "maxThreshold": 32.0,
    "enabled": true,
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 4001 | 404 | 大棚不存在 |
| 4005 | 404 | 传感器组不存在 |
| 3002 | 403 | 无该大棚访问权限 |
| 3004 | 403 | 仅棚主可执行此操作 |
| 1001 | 400 | 参数错误 |

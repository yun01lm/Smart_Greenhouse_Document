---
id: API-004
title: 传感器数据 API
type: api
module: sensor
tags: [传感器, 时序数据, InfluxDB, 多组对比, 聚合, 导出]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

传感器数据模块提供时序传感器数据的查询、对比、聚合和导出功能。数据存储在 InfluxDB 中，支持按大棚、传感器组（group_id）、传感器类型和时间范围进行多维度查询。支持三种数据视图：单组视图、多组对比视图和全棚聚合视图。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/sensors/realtime` | 是 | OWNER/WORKER | 实时数据（可按组过滤） |
| GET | `/api/v1/sensors/history` | 是 | OWNER/WORKER | 历史数据（时间范围/聚合/按组） |
| GET | `/api/v1/sensors/export` | 是 | OWNER/ADMIN | 导出CSV |
| GET | `/api/v1/sensors/compare` | 是 | OWNER/WORKER | 多组数据对比 |
| GET | `/api/v1/sensors/aggregate` | 是 | OWNER/WORKER | 大棚聚合数据（平均/最高最低） |

## 请求/响应示例

### 1. 实时数据

**请求：**
```
GET /api/v1/sensors/realtime?greenhouseId=1&groupId=1
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "greenhouseId": 1,
    "groupId": 1,
    "zoneLabel": "东侧",
    "timestamp": "2026-07-11T10:30:00",
    "sensors": {
      "TEMP": 28.5,
      "HUMIDITY": 65.2,
      "LIGHT": 32000,
      "CO2": 420,
      "O2": 20.8,
      "SOIL_TEMP": 24.3,
      "SOIL_HUMIDITY": 45.6,
      "EC": 1.2,
      "N": 120,
      "P": 45,
      "K": 180
    }
  }
}
```

### 2. 历史数据

**请求：**
```
GET /api/v1/sensors/history?greenhouseId=1&groupId=1&sensorType=TEMP&startTime=2026-07-10T00:00:00&endTime=2026-07-11T00:00:00&aggregation=30m
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "greenhouseId": 1,
    "groupId": 1,
    "sensorType": "TEMP",
    "aggregation": "30m",
    "unit": "°C",
    "dataPoints": [
      {"time": "2026-07-10T00:00:00", "avg": 22.5, "min": 21.8, "max": 23.1},
      {"time": "2026-07-10T00:30:00", "avg": 22.3, "min": 21.5, "max": 23.0},
      {"time": "2026-07-10T01:00:00", "avg": 21.9, "min": 21.2, "max": 22.5}
    ]
  }
}
```

### 3. 多组数据对比

**请求：**
```
GET /api/v1/sensors/compare?greenhouseId=1&sensorType=TEMP&startTime=2026-07-11T08:00:00&endTime=2026-07-11T10:00:00
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "greenhouseId": 1,
    "sensorType": "TEMP",
    "groups": [
      {
        "groupId": 1,
        "zoneLabel": "东侧",
        "dataPoints": [
          {"time": "2026-07-11T08:00:00", "value": 26.5},
          {"time": "2026-07-11T09:00:00", "value": 28.1}
        ]
      },
      {
        "groupId": 2,
        "zoneLabel": "西侧",
        "dataPoints": [
          {"time": "2026-07-11T08:00:00", "value": 25.8},
          {"time": "2026-07-11T09:00:00", "value": 27.5}
        ]
      }
    ]
  }
}
```

### 4. 大棚聚合数据

**请求：**
```
GET /api/v1/sensors/aggregate?greenhouseId=1
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "greenhouseId": 1,
    "timestamp": "2026-07-11T10:30:00",
    "summary": {
      "TEMP": {"avg": 27.8, "min": 26.5, "max": 29.2, "stdDev": 1.1},
      "HUMIDITY": {"avg": 64.5, "min": 60.1, "max": 68.3, "stdDev": 3.2},
      "CO2": {"avg": 415, "min": 400, "max": 435, "stdDev": 12.5}
    },
    "groups": [
      {"groupId": 1, "zoneLabel": "东侧", "online": true},
      {"groupId": 2, "zoneLabel": "西侧", "online": true},
      {"groupId": 3, "zoneLabel": "入口", "online": false}
    ]
  }
}
```

### 5. 导出CSV

**请求：**
```
GET /api/v1/sensors/export?greenhouseId=1&sensorType=TEMP&startTime=2026-07-01&endTime=2026-07-11&format=csv
Authorization: Bearer <token>
```

**响应：** 返回 CSV 文件下载流，Content-Type: `text/csv`

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 4001 | 404 | 大棚不存在 |
| 4005 | 404 | 传感器组不存在 |
| 3002 | 403 | 无该大棚访问权限 |
| 1001 | 400 | 参数错误（时间范围/传感器类型无效） |
| 1003 | 500 | 数据查询服务异常 |

---
id: API-003
title: 设备与传感器管理 API
type: api
module: device
tags: [设备, ESP32, 传感器, 执行器, 传感器组, 区域标注]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

设备管理模块负责 ESP32 设备的注册、状态监控，以及传感器/执行器的配置管理。支持多组传感器独立部署（3-5组/大棚），每组通过 `group_id` 标识，可进行区域标注（如"东侧"、"西侧"、"入口"）。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

### 设备管理

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/devices` | 是 | OWNER/WORKER/ADMIN | 设备列表 |
| POST | `/api/v1/devices` | 是 | OWNER/ADMIN | 注册设备 |
| GET | `/api/v1/devices/{id}/status` | 是 | OWNER/WORKER/ADMIN | 设备状态详情 |
| POST | `/api/v1/devices/{id}/sensors` | 是 | OWNER/ADMIN | 配置传感器 |
| POST | `/api/v1/devices/{id}/actuators` | 是 | OWNER/ADMIN | 配置执行器 |

### 传感器组管理

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/device-groups` | 是 | OWNER/WORKER/ADMIN | 传感器组列表 |
| POST | `/api/v1/device-groups` | 是 | OWNER/ADMIN | 创建传感器组 |
| PUT | `/api/v1/device-groups/{id}` | 是 | OWNER/ADMIN | 更新传感器组（区域标注） |
| DELETE | `/api/v1/device-groups/{id}` | 是 | OWNER/ADMIN | 删除传感器组 |

## 请求/响应示例

### 1. 注册设备

**请求：**
```json
POST /api/v1/devices
Authorization: Bearer <token>
Content-Type: application/json

{
  "deviceCode": "ESP32-GH1-001",
  "greenhouseId": 1,
  "groupId": 1,
  "name": "东侧传感器节点",
  "wifiSsid": "Greenhouse-WiFi"
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "deviceCode": "ESP32-GH1-001",
    "greenhouseId": 1,
    "groupId": 1,
    "name": "东侧传感器节点",
    "status": 0,
    "createdAt": "2026-07-11T10:00:00"
  }
}
```

### 2. 获取设备状态

**请求：**
```
GET /api/v1/devices/1/status
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "deviceCode": "ESP32-GH1-001",
    "name": "东侧传感器节点",
    "status": 1,
    "lastHeartbeat": "2026-07-11T10:29:45",
    "wifiSsid": "Greenhouse-WiFi",
    "sensors": [
      {"id": 1, "sensorType": "TEMP", "minThreshold": 15.0, "maxThreshold": 35.0, "status": 1},
      {"id": 2, "sensorType": "HUMIDITY", "minThreshold": 30.0, "maxThreshold": 90.0, "status": 1}
    ],
    "actuators": [
      {"id": 1, "name": "通风风机", "actuatorType": "FAN", "gpioPin": 25, "currentState": 0}
    ]
  }
}
```

### 3. 配置传感器

**请求：**
```json
POST /api/v1/devices/1/sensors
Authorization: Bearer <token>
Content-Type: application/json

{
  "sensorType": "TEMP",
  "i2cAddress": "0x44",
  "minThreshold": 15.0,
  "maxThreshold": 35.0
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 3,
    "deviceId": 1,
    "sensorType": "TEMP",
    "i2cAddress": "0x44",
    "minThreshold": 15.0,
    "maxThreshold": 35.0,
    "status": 1
  }
}
```

### 4. 创建传感器组

**请求：**
```json
POST /api/v1/device-groups
Authorization: Bearer <token>
Content-Type: application/json

{
  "greenhouseId": 1,
  "groupName": "东侧传感器组",
  "zoneLabel": "东侧",
  "description": "覆盖大棚东侧区域，监测番茄种植区",
  "sortOrder": 1
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "greenhouseId": 1,
    "groupName": "东侧传感器组",
    "zoneLabel": "东侧",
    "description": "覆盖大棚东侧区域，监测番茄种植区",
    "sortOrder": 1,
    "deviceCount": 1,
    "status": 1,
    "createdAt": "2026-07-11T10:00:00"
  }
}
```

### 5. 配置执行器

**请求：**
```json
POST /api/v1/devices/1/actuators
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "滴灌阀门",
  "actuatorType": "IRRIGATION",
  "gpioPin": 27
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 2,
    "deviceId": 1,
    "name": "滴灌阀门",
    "actuatorType": "IRRIGATION",
    "gpioPin": 27,
    "currentState": 0,
    "status": 1
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 4003 | 404 | 设备不存在 |
| 4004 | 400 | 设备已离线 |
| 4005 | 404 | 传感器组不存在 |
| 4001 | 404 | 大棚不存在 |
| 3002 | 403 | 无该大棚访问权限 |
| 1001 | 400 | 参数错误 |

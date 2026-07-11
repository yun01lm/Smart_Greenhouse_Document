---
id: API-009b
title: 专家授权与工作台 API
type: api
module: expert
tags: [专家, 授权, 7天有效期, 专家列表, 工作台]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

专家模块提供专家列表查询、数据授权管理和专家工作台功能。专家可通过授权机制申请查看用户大棚的传感器数据，授权有效期为7天，用户可随时撤销。专家工作台（Web端）提供求助列表、咨询统计和在线状态管理功能。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

### 专家列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/experts` | 是 | OWNER/WORKER | 专家列表（含在线状态） |

### 专家授权

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| POST | `/api/v1/expert/authorize/request` | 是 | EXPERT | 专家发起授权请求 |
| GET | `/api/v1/expert/authorize/pending` | 是 | OWNER/WORKER | 用户查看待处理请求 |
| PUT | `/api/v1/expert/authorize/{id}/approve` | 是 | OWNER | 用户同意授权 |
| PUT | `/api/v1/expert/authorize/{id}/reject` | 是 | OWNER | 用户拒绝授权 |
| PUT | `/api/v1/expert/authorize/{id}/revoke` | 是 | OWNER | 用户撤销授权 |
| GET | `/api/v1/expert/authorize/active` | 是 | OWNER/EXPERT | 查看有效授权列表 |
| GET | `/api/v1/expert/authorize/history` | 是 | OWNER/EXPERT | 授权历史 |
| GET | `/api/v1/expert/greenhouses` | 是 | EXPERT | 专家查看已授权的大棚数据 |

### 专家工作台（Web端）

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/expert/consultations` | 是 | EXPERT | 求助列表（待处理/处理中/已结束） |
| GET | `/api/v1/expert/statistics` | 是 | EXPERT | 咨询统计 |
| PUT | `/api/v1/expert/status` | 是 | EXPERT | 更新在线状态 |
| GET | `/api/v1/expert/greenhouses/{id}/data` | 是 | EXPERT(已授权) | 查看已授权大棚传感器数据 |

## 请求/响应示例

### 1. 专家列表

**请求：**
```
GET /api/v1/experts?specialty=蔬菜植保&onlineOnly=true
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
        "id": 5,
        "realName": "李明",
        "expertSpecialty": "蔬菜植保",
        "expertStatus": 1,
        "rating": 4.8,
        "consultCount": 156,
        "currentCount": 2
      },
      {
        "id": 6,
        "realName": "王芳",
        "expertSpecialty": "蔬菜植保",
        "expertStatus": 1,
        "rating": 4.6,
        "consultCount": 98,
        "currentCount": 1
      }
    ]
  }
}
```

### 2. 专家发起授权请求

**请求：**
```json
POST /api/v1/expert/authorize/request
Authorization: Bearer <token>
Content-Type: application/json

{
  "greenhouseId": 1,
  "reason": "需要查看环境数据辅助病害诊断"
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 45,
    "expertId": 5,
    "userId": 1,
    "greenhouseId": 1,
    "status": "PENDING",
    "reason": "需要查看环境数据辅助病害诊断",
    "requestedAt": "2026-07-11T10:30:00"
  }
}
```

### 3. 用户同意授权

**请求：**
```
PUT /api/v1/expert/authorize/45/approve
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 45,
    "status": "APPROVED",
    "approvedAt": "2026-07-11T10:35:00",
    "expiresAt": "2026-07-18T10:35:00",
    "expiresIn": "7天"
  }
}
```

### 4. 专家查看已授权大棚数据

**请求：**
```
GET /api/v1/expert/greenhouses/1/data
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "greenhouseId": 1,
    "greenhouseName": "1号番茄大棚",
    "cropType": "番茄",
    "realtimeData": {
      "timestamp": "2026-07-11T10:35:00",
      "groups": [
        {
          "groupId": 1,
          "zoneLabel": "东侧",
          "sensors": {
            "TEMP": 28.5,
            "HUMIDITY": 65.2,
            "CO2": 420,
            "SOIL_HUMIDITY": 45.6,
            "EC": 1.2
          }
        }
      ]
    },
    "authorizationExpiresAt": "2026-07-18T10:35:00"
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 6001 | 404 | 专家不存在 |
| 6005 | 404 | 授权记录不存在 |
| 6006 | 403 | 授权已过期 |
| 6007 | 409 | 已有有效授权，无需重复申请 |
| 3002 | 403 | 无该大棚访问权限 |
| 3004 | 403 | 仅棚主可执行此操作 |
| 1001 | 400 | 参数错误 |

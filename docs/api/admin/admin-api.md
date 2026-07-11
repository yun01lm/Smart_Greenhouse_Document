---
id: API-010b
title: 系统管理 API
type: api
module: admin
tags: [管理员, 用户管理, 角色管理, AI引擎, 地区统计]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

系统管理模块仅限管理员（ADMIN）角色在 Web 管理端使用，提供用户管理、角色管理、地区统计和 AI 引擎配置管理功能。管理员可查看平台下所有用户和大棚，按角色和地区进行筛选，并管理 AI 引擎的切换和状态监控。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

### 用户管理

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/admin/users` | 是 | ADMIN | 用户列表（含角色/地区筛选） |
| PUT | `/api/v1/admin/users/{id}` | 是 | ADMIN | 更新用户信息 |
| GET | `/api/v1/admin/users/regions` | 是 | ADMIN | 用户地区分布统计 |

### 角色管理

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/admin/roles` | 是 | ADMIN | 角色列表 |

### AI引擎管理

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/admin/ai/config` | 是 | ADMIN | 当前AI引擎配置 |
| PUT | `/api/v1/admin/ai/config` | 是 | ADMIN | 切换AI引擎 |
| GET | `/api/v1/admin/ai/status` | 是 | ADMIN | 各引擎状态和调用量 |

## 请求/响应示例

### 1. 用户列表

**请求：**
```
GET /api/v1/admin/users?role=OWNER&province=河北省&page=1&size=20
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
        "username": "zhangfarmer",
        "phone": "13800138001",
        "realName": "张三",
        "role": "OWNER",
        "province": "河北省",
        "city": "石家庄市",
        "district": "藁城区",
        "greenhouseCount": 3,
        "employeeCount": 2,
        "status": 1,
        "createdAt": "2026-07-01T08:00:00"
      }
    ],
    "total": 12,
    "page": 1,
    "size": 20
  }
}
```

### 2. 用户地区分布统计

**请求：**
```
GET /api/v1/admin/users/regions
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "totalUsers": 50,
    "byProvince": [
      {"province": "河北省", "userCount": 30, "greenhouseCount": 85},
      {"province": "山东省", "userCount": 12, "greenhouseCount": 30},
      {"province": "河南省", "userCount": 8, "greenhouseCount": 15}
    ],
    "byRole": {
      "OWNER": 25,
      "WORKER": 18,
      "EXPERT": 5,
      "ADMIN": 2
    }
  }
}
```

### 3. 角色列表

**请求：**
```
GET /api/v1/admin/roles
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "roles": [
      {
        "code": "ADMIN",
        "name": "管理员",
        "description": "系统管理员，Web端使用，管理全部用户和大棚",
        "availableEndpoints": ["Web端"],
        "userCount": 2
      },
      {
        "code": "OWNER",
        "name": "棚主",
        "description": "大棚所有者，APP端+Web端使用，管理自己名下大棚和员工",
        "availableEndpoints": ["APP端", "Web端"],
        "userCount": 25
      },
      {
        "code": "WORKER",
        "name": "员工",
        "description": "大棚员工，仅APP端使用，权限由棚主分配，单归属",
        "availableEndpoints": ["APP端"],
        "userCount": 18
      },
      {
        "code": "EXPERT",
        "name": "专家",
        "description": "农业专家，仅Web端使用，提供诊断咨询",
        "availableEndpoints": ["Web端"],
        "userCount": 5
      }
    ]
  }
}
```

### 4. 当前AI引擎配置

**请求：**
```
GET /api/v1/admin/ai/config
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "imageProvider": "baidu",
    "voiceProvider": "xunfei",
    "llmProvider": "deepseek-chat",
    "embeddingProvider": "siliconflow-bge-m3",
    "availableImageProviders": ["baidu", "resnet"],
    "availableVoiceProviders": ["xunfei", "whisper"],
    "updatedAt": "2026-07-01T08:00:00"
  }
}
```

### 5. 切换AI引擎

**请求：**
```json
PUT /api/v1/admin/ai/config
Authorization: Bearer <token>
Content-Type: application/json

{
  "imageProvider": "resnet",
  "voiceProvider": "whisper"
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "imageProvider": "resnet",
    "voiceProvider": "whisper",
    "llmProvider": "deepseek-chat",
    "embeddingProvider": "siliconflow-bge-m3",
    "updatedAt": "2026-07-11T11:00:00"
  }
}
```

### 6. AI引擎状态

**请求：**
```
GET /api/v1/admin/ai/status
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "engines": [
      {
        "name": "baidu-image",
        "type": "图像识别",
        "status": "ACTIVE",
        "todayCalls": 45,
        "totalCalls": 1230,
        "avgResponseTime": 1200,
        "lastCallAt": "2026-07-11T10:30:00"
      },
      {
        "name": "deepseek-chat",
        "type": "LLM问答",
        "status": "ACTIVE",
        "todayCalls": 28,
        "totalCalls": 850,
        "avgResponseTime": 3500,
        "lastCallAt": "2026-07-11T10:28:00"
      },
      {
        "name": "siliconflow-bge-m3",
        "type": "向量化",
        "status": "ACTIVE",
        "todayCalls": 120,
        "totalCalls": 3500,
        "avgResponseTime": 500,
        "lastCallAt": "2026-07-11T10:30:00"
      }
    ]
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 3001 | 403 | 无访问权限（非管理员） |
| 1002 | 404 | 用户不存在 |
| 1001 | 400 | 参数错误 |
| 1003 | 500 | AI引擎切换失败 |

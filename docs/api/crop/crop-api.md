---
id: API-010e
title: 作物生长周期 API
type: api
module: crop
tags: [生长周期, 种植记录, 阶段管理, 时间线]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

作物生长周期管理模块提供种植记录的增删改查、生长阶段自动估算与手动修正、以及生长时间线查看功能。系统根据种植日期和作物类型的标准周期自动估算当前生长阶段（育苗期→生长期→开花期→结果期→收获期），用户可根据实际情况手动覆盖。生长时间线关联长势评估记录和预警记录，形成完整的种植追溯。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/crop-cycles` | 是 | OWNER/WORKER | 生长周期列表（按大棚过滤） |
| POST | `/api/v1/crop-cycles` | 是 | OWNER | 创建种植记录 |
| GET | `/api/v1/crop-cycles/{id}` | 是 | OWNER/WORKER | 周期详情 |
| PUT | `/api/v1/crop-cycles/{id}` | 是 | OWNER | 更新周期（推进阶段等） |
| PATCH | `/api/v1/crop-cycles/{id}/complete` | 是 | OWNER | 标记完成（收获） |
| GET | `/api/v1/crop-cycles/{id}/timeline` | 是 | OWNER/WORKER | 生长时间线 |

## 请求/响应示例

### 1. 创建种植记录

**请求：**
```json
POST /api/v1/crop-cycles
Authorization: Bearer <token>
Content-Type: application/json

{
  "greenhouseId": 1,
  "cropType": "番茄",
  "variety": "金棚一号",
  "plantingDate": "2026-06-01",
  "expectedHarvestDate": "2026-08-15",
  "notes": "春季茬，使用滴灌施肥"
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
    "cropType": "番茄",
    "variety": "金棚一号",
    "plantingDate": "2026-06-01",
    "expectedHarvestDate": "2026-08-15",
    "currentStage": "开花期",
    "stageSource": "AUTO",
    "daysSincePlanting": 40,
    "status": "ACTIVE",
    "notes": "春季茬，使用滴灌施肥",
    "createdAt": "2026-07-11T10:00:00"
  }
}
```

### 2. 生长周期列表

**请求：**
```
GET /api/v1/crop-cycles?greenhouseId=1&status=ACTIVE
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
        "greenhouseId": 1,
        "cropType": "番茄",
        "variety": "金棚一号",
        "plantingDate": "2026-06-01",
        "currentStage": "开花期",
        "stageSource": "AUTO",
        "daysSincePlanting": 40,
        "status": "ACTIVE",
        "latestHealthScore": 85.2,
        "createdAt": "2026-07-11T10:00:00"
      }
    ],
    "total": 1
  }
}
```

### 3. 周期详情

**请求：**
```
GET /api/v1/crop-cycles/1
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "greenhouseId": 1,
    "greenhouseName": "1号番茄大棚",
    "cropType": "番茄",
    "variety": "金棚一号",
    "plantingDate": "2026-06-01",
    "expectedHarvestDate": "2026-08-15",
    "currentStage": "开花期",
    "stageSource": "AUTO",
    "daysSincePlanting": 40,
    "estimatedDaysToHarvest": 35,
    "status": "ACTIVE",
    "stageHistory": [
      {"stage": "育苗期", "startDate": "2026-06-01", "endDate": "2026-06-15", "source": "AUTO"},
      {"stage": "生长期", "startDate": "2026-06-15", "endDate": "2026-06-30", "source": "AUTO"},
      {"stage": "开花期", "startDate": "2026-06-30", "endDate": null, "source": "AUTO"}
    ],
    "latestAssessment": {
      "id": 15,
      "plantHeight": 85.5,
      "leafArea": 320.5,
      "healthScore": 88.0,
      "createdAt": "2026-07-11T10:30:00"
    },
    "notes": "春季茬，使用滴灌施肥",
    "createdAt": "2026-07-11T10:00:00"
  }
}
```

### 4. 更新生长周期（手动修正阶段）

**请求：**
```json
PUT /api/v1/crop-cycles/1
Authorization: Bearer <token>
Content-Type: application/json

{
  "currentStage": "结果期",
  "stageSource": "MANUAL",
  "notes": "春季茬，实际生长较快，手动推进到结果期"
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "currentStage": "结果期",
    "stageSource": "MANUAL",
    "notes": "春季茬，实际生长较快，手动推进到结果期",
    "updatedAt": "2026-07-11T11:00:00"
  }
}
```

### 5. 标记完成（收获）

**请求：**
```
PATCH /api/v1/crop-cycles/1/complete
Authorization: Bearer <token>
Content-Type: application/json

{
  "actualHarvestDate": "2026-08-10",
  "notes": "产量良好，果实品质佳"
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "status": "COMPLETED",
    "actualHarvestDate": "2026-08-10",
    "totalDays": 70,
    "notes": "产量良好，果实品质佳",
    "completedAt": "2026-08-10T18:00:00"
  }
}
```

### 6. 生长时间线

**请求：**
```
GET /api/v1/crop-cycles/1/timeline
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "cycleId": 1,
    "cropType": "番茄",
    "plantingDate": "2026-06-01",
    "events": [
      {
        "date": "2026-06-01",
        "type": "PLANTING",
        "description": "种植番茄（金棚一号）"
      },
      {
        "date": "2026-06-15",
        "type": "STAGE_CHANGE",
        "description": "进入生长期（自动估算）"
      },
      {
        "date": "2026-06-30",
        "type": "STAGE_CHANGE",
        "description": "进入开花期（自动估算）"
      },
      {
        "date": "2026-07-05",
        "type": "ASSESSMENT",
        "description": "长势评估：株高75cm，健康评分86"
      },
      {
        "date": "2026-07-08",
        "type": "ALERT",
        "description": "温度预警：西侧温度达35.2°C"
      },
      {
        "date": "2026-07-11",
        "type": "ASSESSMENT",
        "description": "长势评估：株高85.5cm，健康评分88"
      }
    ]
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 7001 | 404 | 生长周期记录不存在 |
| 7002 | 400 | 该生长周期已结束 |
| 7003 | 409 | 该大棚已有进行中的生长周期 |
| 4001 | 404 | 大棚不存在 |
| 3002 | 403 | 无该大棚访问权限 |
| 3004 | 403 | 仅棚主可执行此操作 |
| 1001 | 400 | 参数错误 |

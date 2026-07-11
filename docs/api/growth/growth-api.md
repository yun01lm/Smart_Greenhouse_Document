---
id: API-008
title: 长势评估 API
type: api
module: growth
tags: [长势, 截帧, 生长阶段, 株高, 叶面积, 叶色]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

长势评估模块提供作物生长状态的评估数据，包括最新长势评估结果、历史记录和 RTSP 截帧图片列表。截帧由 FFmpeg 从 ESP32-CAM RTSP 推流中定时获取（每30分钟），长势评估关联到作物生长周期，记录株高、叶面积、叶色等生长指标。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/growth/latest` | 是 | OWNER/WORKER | 最新长势评估 |
| GET | `/api/v1/growth/history` | 是 | OWNER/WORKER | 长势历史（分页） |
| GET | `/api/v1/growth/images` | 是 | OWNER/WORKER | 截帧图片列表（分页） |

## 请求/响应示例

### 1. 最新长势评估

**请求：**
```
GET /api/v1/growth/latest?greenhouseId=1
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 15,
    "greenhouseId": 1,
    "cropCycleId": 1,
    "imagePath": "/uploads/frames/2026/07/11/frame_103000.jpg",
    "growthStage": "开花期",
    "plantHeight": 85.5,
    "leafArea": 320.5,
    "leafColor": "深绿色",
    "healthScore": 88.0,
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

### 2. 长势历史

**请求：**
```
GET /api/v1/growth/history?greenhouseId=1&page=1&size=10
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
        "id": 15,
        "imagePath": "/uploads/frames/2026/07/11/frame_103000.jpg",
        "growthStage": "开花期",
        "plantHeight": 85.5,
        "leafArea": 320.5,
        "leafColor": "深绿色",
        "healthScore": 88.0,
        "createdAt": "2026-07-11T10:30:00"
      },
      {
        "id": 14,
        "imagePath": "/uploads/frames/2026/07/11/frame_100000.jpg",
        "growthStage": "开花期",
        "plantHeight": 85.0,
        "leafArea": 318.2,
        "leafColor": "深绿色",
        "healthScore": 87.5,
        "createdAt": "2026-07-11T10:00:00"
      }
    ],
    "total": 48,
    "page": 1,
    "size": 10
  }
}
```

### 3. 截帧图片列表

**请求：**
```
GET /api/v1/growth/images?greenhouseId=1&date=2026-07-11&page=1&size=20
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
        "imagePath": "/uploads/frames/2026/07/11/frame_103000.jpg",
        "capturedAt": "2026-07-11T10:30:00",
        "resolution": "640x480",
        "fileSize": 52480
      },
      {
        "id": 2,
        "imagePath": "/uploads/frames/2026/07/11/frame_100000.jpg",
        "capturedAt": "2026-07-11T10:00:00",
        "resolution": "640x480",
        "fileSize": 51320
      }
    ],
    "total": 48,
    "page": 1,
    "size": 20
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 4001 | 404 | 大棚不存在 |
| 3002 | 403 | 无该大棚访问权限 |
| 1001 | 400 | 参数错误 |

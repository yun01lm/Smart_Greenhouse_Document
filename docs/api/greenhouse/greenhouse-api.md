---
id: API-002
title: 大棚管理 API
type: api
module: greenhouse
tags: [大棚, CRUD, 地区统计, 五级地址]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

大棚管理模块提供多大棚的增删改查功能，支持五级地区地址（省-市-区/县-乡镇-村）。数据按角色权限过滤：棚主查看自己名下大棚，员工查看被授权大棚，管理员查看全部大棚。地区分布统计仅管理员可用。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/greenhouses` | 是 | OWNER/WORKER/ADMIN | 大棚列表（按角色过滤） |
| POST | `/api/v1/greenhouses` | 是 | OWNER | 创建大棚 |
| PUT | `/api/v1/greenhouses/{id}` | 是 | OWNER | 更新大棚信息 |
| DELETE | `/api/v1/greenhouses/{id}` | 是 | OWNER | 删除大棚 |
| GET | `/api/v1/greenhouses/regions` | 是 | ADMIN | 地区分布统计 |

## 请求/响应示例

### 1. 获取大棚列表

**请求：**
```
GET /api/v1/greenhouses?page=1&size=10
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
        "name": "1号番茄大棚",
        "location": "河北省石家庄市藁城区",
        "cropType": "番茄",
        "province": "河北省",
        "city": "石家庄市",
        "district": "藁城区",
        "town": "廉州镇",
        "village": "东刘村",
        "deviceGroupCount": 3,
        "deviceCount": 4,
        "status": 1,
        "createdAt": "2026-07-01T08:00:00"
      }
    ],
    "total": 1,
    "page": 1,
    "size": 10
  }
}
```

### 2. 创建大棚

**请求：**
```json
POST /api/v1/greenhouses
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "2号黄瓜大棚",
  "location": "河北省石家庄市藁城区",
  "cropType": "黄瓜",
  "province": "河北省",
  "city": "石家庄市",
  "district": "藁城区",
  "town": "廉州镇",
  "village": "东刘村"
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 2,
    "name": "2号黄瓜大棚",
    "location": "河北省石家庄市藁城区",
    "cropType": "黄瓜",
    "province": "河北省",
    "city": "石家庄市",
    "district": "藁城区",
    "town": "廉州镇",
    "village": "东刘村",
    "userId": 1,
    "status": 1,
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

### 3. 更新大棚

**请求：**
```json
PUT /api/v1/greenhouses/1
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "1号番茄大棚（改造升级）",
  "cropType": "番茄"
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "name": "1号番茄大棚（改造升级）",
    "cropType": "番茄",
    "updatedAt": "2026-07-11T11:00:00"
  }
}
```

### 4. 地区分布统计

**请求：**
```
GET /api/v1/greenhouses/regions
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "totalGreenhouses": 25,
    "totalUsers": 12,
    "byProvince": [
      {"province": "河北省", "greenhouseCount": 18, "userCount": 8},
      {"province": "山东省", "greenhouseCount": 7, "userCount": 4}
    ],
    "byCity": [
      {"city": "石家庄市", "province": "河北省", "greenhouseCount": 10},
      {"city": "保定市", "province": "河北省", "greenhouseCount": 8}
    ]
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 4001 | 404 | 大棚不存在 |
| 4002 | 400 | 大棚数量已达上限 |
| 3001 | 403 | 无访问权限 |
| 3002 | 403 | 无该大棚访问权限 |
| 1001 | 400 | 参数错误 |

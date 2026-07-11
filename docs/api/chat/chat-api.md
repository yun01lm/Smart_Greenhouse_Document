---
id: API-009
title: 专家咨询对话 API
type: api
module: chat
tags: [专家咨询, 实时聊天, WebSocket, 环境快照, 对话管理]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

专家咨询模块提供用户与专家之间的实时对话功能。支持文字、图片、视频和环境快照四种消息类型。对话消息通过 WebSocket (STOMP) 进行实时推送，同时持久化到 MySQL。REST API 作为备用通道，用于对话管理、消息历史查询等操作。环境快照可在对话中一键发送，展示当前大棚各传感器组的实时数据。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

**WebSocket 端点**：`/ws/connect`（STOMP协议）

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| POST | `/api/v1/chat/conversations` | 是 | OWNER/WORKER | 创建对话（发起求助） |
| GET | `/api/v1/chat/conversations` | 是 | OWNER/WORKER/EXPERT | 对话列表（分页） |
| GET | `/api/v1/chat/conversations/{id}/messages` | 是 | 参与者 | 对话消息历史（分页） |
| POST | `/api/v1/chat/messages` | 是 | 参与者 | 发送消息（REST备用） |
| POST | `/api/v1/chat/snapshot` | 是 | OWNER/WORKER | 发送环境快照 |
| PUT | `/api/v1/chat/conversations/{id}/close` | 是 | 参与者 | 关闭对话 |
| GET | `/api/v1/chat/unread` | 是 | 已登录 | 未读消息数 |

## 请求/响应示例

### 1. 创建对话（发起求助）

**请求：**
```json
POST /api/v1/chat/conversations
Authorization: Bearer <token>
Content-Type: application/json

{
  "expertId": 5,
  "greenhouseId": 1,
  "subject": "番茄叶片出现黄斑，求助诊断",
  "diagnosticId": 42
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 10,
    "userId": 1,
    "expertId": 5,
    "expertName": "李明",
    "expertSpecialty": "蔬菜植保",
    "greenhouseId": 1,
    "subject": "番茄叶片出现黄斑，求助诊断",
    "status": "WAITING",
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

### 2. 获取对话列表

**请求：**
```
GET /api/v1/chat/conversations?status=ACTIVE&page=1&size=10
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
        "id": 10,
        "expertId": 5,
        "expertName": "李明",
        "expertSpecialty": "蔬菜植保",
        "subject": "番茄叶片出现黄斑，求助诊断",
        "status": "ACTIVE",
        "lastMessage": "请拍一张叶片背面的照片",
        "lastMessageTime": "2026-07-11T10:35:00",
        "unreadCount": 2,
        "createdAt": "2026-07-11T10:30:00"
      }
    ],
    "total": 3,
    "page": 1,
    "size": 10
  }
}
```

### 3. 获取对话消息历史

**请求：**
```
GET /api/v1/chat/conversations/10/messages?page=1&size=50
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
        "id": 201,
        "senderId": 1,
        "senderType": "USER",
        "senderName": "张农户",
        "messageType": "TEXT",
        "content": "李老师您好，我大棚里的番茄叶片出现黄斑，您能帮忙看看吗？",
        "readStatus": 1,
        "createdAt": "2026-07-11T10:30:00"
      },
      {
        "id": 202,
        "senderId": 5,
        "senderType": "EXPERT",
        "senderName": "李明",
        "messageType": "TEXT",
        "content": "请拍一张叶片背面的照片，我需要看清楚病斑的具体形态。",
        "readStatus": 1,
        "createdAt": "2026-07-11T10:32:00"
      },
      {
        "id": 203,
        "senderId": 1,
        "senderType": "USER",
        "senderName": "张农户",
        "messageType": "IMAGE",
        "content": "[图片]",
        "filePath": "/uploads/chat/2026/07/11/img_001.jpg",
        "readStatus": 0,
        "createdAt": "2026-07-11T10:34:00"
      }
    ],
    "total": 5,
    "page": 1,
    "size": 50
  }
}
```

### 4. 发送环境快照

**请求：**
```json
POST /api/v1/chat/snapshot
Authorization: Bearer <token>
Content-Type: application/json

{
  "conversationId": 10,
  "greenhouseId": 1,
  "groupIds": [1, 2, 3]
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 204,
    "messageType": "ENV_SNAPSHOT",
    "content": "[环境快照] 1号大棚 · 2026-07-11 10:30",
    "snapshotData": {
      "greenhouseId": 1,
      "greenhouseName": "1号番茄大棚",
      "capturedAt": "2026-07-11T10:30:00",
      "groups": [
        {
          "groupId": 1,
          "zoneLabel": "东侧",
          "sensors": {
            "TEMP": 28.5,
            "HUMIDITY": 65.2,
            "CO2": 420,
            "O2": 20.8,
            "SOIL_TEMP": 24.3,
            "SOIL_HUMIDITY": 45.6,
            "EC": 1.2
          }
        }
      ],
      "avgTemp": 28.1,
      "avgHumidity": 64.8
    },
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

### 5. 获取未读消息数

**请求：**
```
GET /api/v1/chat/unread
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "totalUnread": 5,
    "conversations": [
      {"conversationId": 10, "expertName": "李明", "unreadCount": 2},
      {"conversationId": 8, "expertName": "王芳", "unreadCount": 3}
    ]
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 6001 | 404 | 专家不存在 |
| 6002 | 400 | 专家当前不在线 |
| 6003 | 404 | 对话不存在 |
| 6004 | 400 | 对话已结束 |
| 3002 | 403 | 无该大棚访问权限 |
| 1001 | 400 | 参数错误 |

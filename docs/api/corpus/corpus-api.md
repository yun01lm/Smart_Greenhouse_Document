---
id: API-010d
title: 语料管理 API
type: api
module: corpus
tags: [语料, 方言, 语音, 标注, Whisper]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

语料管理模块供管理员在 Web 管理端维护河北方言语料集。支持语料上传、删除和用途标记。语料集既作为项目交付物之一，也为后期 Whisper 方言微调提供训练和测试数据。语料标注包含方言原文、普通话翻译、分类和用途（训练/测试/通用）。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/corpus` | 是 | ADMIN | 语料列表（分页/按分类/用途筛选） |
| POST | `/api/v1/corpus` | 是 | ADMIN | 上传语料 |
| DELETE | `/api/v1/corpus/{id}` | 是 | ADMIN | 删除语料 |
| PUT | `/api/v1/corpus/{id}/usage` | 是 | ADMIN | 更新语料用途 |

## 请求/响应示例

### 1. 语料列表

**请求：**
```
GET /api/v1/corpus?category=病害描述&usageType=TRAIN&page=1&size=20
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
        "audioPath": "/uploads/corpus/2026/07/01/hebei_001.wav",
        "dialectText": "这菜叶子咋黄了哩？",
        "mandarinText": "这菜叶子怎么黄了呢？",
        "category": "病害描述",
        "durationSec": 3.5,
        "usageType": "TRAIN",
        "createdAt": "2026-07-01T08:00:00"
      },
      {
        "id": 2,
        "audioPath": "/uploads/corpus/2026/07/01/hebei_002.wav",
        "dialectText": "大棚里头忒热了，开开风机呗",
        "mandarinText": "大棚里面太热了，打开风机吧",
        "category": "设备控制",
        "durationSec": 4.2,
        "usageType": "TRAIN",
        "createdAt": "2026-07-01T08:05:00"
      }
    ],
    "total": 120,
    "page": 1,
    "size": 20
  }
}
```

### 2. 上传语料

**请求：**
```
POST /api/v1/corpus
Authorization: Bearer <token>
Content-Type: multipart/form-data

audio: (binary file, .wav, 16kHz, 单声道, max 30MB)
dialectText: "这苗儿长得可不赖"
mandarinText: "这苗长得真不错"
category: "长势描述"
usageType: "BOTH"
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 121,
    "audioPath": "/uploads/corpus/2026/07/11/hebei_121.wav",
    "dialectText": "这苗儿长得可不赖",
    "mandarinText": "这苗长得真不错",
    "category": "长势描述",
    "durationSec": 2.8,
    "usageType": "BOTH",
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

### 3. 更新语料用途

**请求：**
```json
PUT /api/v1/corpus/121/usage
Authorization: Bearer <token>
Content-Type: application/json

{
  "usageType": "TRAIN"
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 121,
    "usageType": "TRAIN",
    "updatedAt": "2026-07-11T10:35:00"
  }
}
```

### 4. 删除语料

**请求：**
```
DELETE /api/v1/corpus/121
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": null
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 3001 | 403 | 无访问权限（非管理员） |
| 1002 | 404 | 语料不存在 |
| 8001 | 500 | 文件上传失败 |
| 8002 | 400 | 文件大小超过限制 |
| 8003 | 400 | 不支持的文件类型（仅支持.wav） |
| 1001 | 400 | 参数错误（方言文本/普通话文本为空） |

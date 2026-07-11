---
id: API-010c
title: 知识库管理 API
type: api
module: knowledge
tags: [知识库, 文档管理, 向量化, RAG, Chroma]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

知识库管理模块供管理员在 Web 管理端维护农业知识库。支持文档上传、删除、向量化入库和问答测试。知识库文档通过 Spring AI 自动切片（每片约500字），经 SiliconFlow bge-m3 模型生成1024维向量后存储到 Chroma 向量数据库，为 RAG 问答提供检索基础。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/knowledge/documents` | 是 | ADMIN | 文档列表（分页/按分类筛选） |
| POST | `/api/v1/knowledge/documents` | 是 | ADMIN | 上传文档 |
| DELETE | `/api/v1/knowledge/documents/{id}` | 是 | ADMIN | 删除文档 |
| POST | `/api/v1/knowledge/index` | 是 | ADMIN | 触发向量化 |
| POST | `/api/v1/knowledge/test` | 是 | ADMIN | 问答测试 |

## 请求/响应示例

### 1. 文档列表

**请求：**
```
GET /api/v1/knowledge/documents?category=病虫害防治&cropType=番茄&page=1&size=10
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
        "title": "番茄晚疫病综合防治技术",
        "category": "病虫害防治",
        "cropType": "番茄",
        "filePath": "/uploads/knowledge/tomato_late_blight.md",
        "chunkCount": 12,
        "vectorIndexed": true,
        "createdAt": "2026-07-01T08:00:00"
      },
      {
        "id": 2,
        "title": "大棚番茄水肥管理要点",
        "category": "栽培技术",
        "cropType": "番茄",
        "filePath": "/uploads/knowledge/tomato_fertilizer.md",
        "chunkCount": 8,
        "vectorIndexed": true,
        "createdAt": "2026-07-01T08:30:00"
      }
    ],
    "total": 25,
    "page": 1,
    "size": 10
  }
}
```

### 2. 上传文档

**请求：**
```
POST /api/v1/knowledge/documents
Authorization: Bearer <token>
Content-Type: multipart/form-data

file: (binary file, .md/.txt/.docx, max 20MB)
title: "黄瓜霜霉病防治手册"
category: "病虫害防治"
cropType: "黄瓜"
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 26,
    "title": "黄瓜霜霉病防治手册",
    "category": "病虫害防治",
    "cropType": "黄瓜",
    "filePath": "/uploads/knowledge/cucumber_downy_mildew.md",
    "chunkCount": 0,
    "vectorIndexed": false,
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

### 3. 触发向量化

**请求：**
```json
POST /api/v1/knowledge/index
Authorization: Bearer <token>
Content-Type: application/json

{
  "documentId": 26
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "documentId": 26,
    "title": "黄瓜霜霉病防治手册",
    "chunkCount": 10,
    "vectorIndexed": true,
    "indexedAt": "2026-07-11T10:31:00"
  }
}
```

### 4. 问答测试

**请求：**
```json
POST /api/v1/knowledge/test
Authorization: Bearer <token>
Content-Type: application/json

{
  "question": "黄瓜霜霉病怎么防治？",
  "topK": 5
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "question": "黄瓜霜霉病怎么防治？",
    "answer": "黄瓜霜霉病是由古巴假霜霉菌引起的真菌性病害...",
    "retrievedChunks": [
      {
        "documentId": 26,
        "documentTitle": "黄瓜霜霉病防治手册",
        "chunkIndex": 3,
        "content": "化学防治：发病初期可使用霜脲·锰锌800倍液...",
        "score": 0.92
      }
    ],
    "responseTime": 3200
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 3001 | 403 | 无访问权限（非管理员） |
| 1002 | 404 | 文档不存在 |
| 5003 | 500 | AI服务暂时不可用 |
| 5004 | 500 | 向量化服务异常 |
| 8001 | 500 | 文件上传失败 |
| 8002 | 400 | 文件大小超过限制 |
| 8003 | 400 | 不支持的文件类型 |
| 1001 | 400 | 参数错误 |

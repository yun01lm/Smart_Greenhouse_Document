---
id: API-007b
title: AI问答 API
type: api
module: qa
tags: [问答, RAG, DeepSeek, 语音识别, 知识库]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

AI 问答模块提供基于 RAG（检索增强生成）的智能问答服务。支持文字和语音（河北话方言）两种输入方式。系统通过 Chroma 向量库检索相关知识片段，结合 DeepSeek 大语言模型生成专业答案，并附上引用来源。语音问答经 AI 抽象层（初期讯飞ASR，后期 Whisper）转写为文字后再进行问答。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| POST | `/api/v1/qa/ask` | 是 | OWNER/WORKER | 文字问答 |
| POST | `/api/v1/qa/ask/voice` | 是 | OWNER/WORKER | 语音问答（multipart/form-data） |
| GET | `/api/v1/qa/records` | 是 | OWNER/WORKER | 问答历史（分页） |

## 请求/响应示例

### 1. 文字问答

**请求：**
```json
POST /api/v1/qa/ask
Authorization: Bearer <token>
Content-Type: application/json

{
  "question": "番茄叶子发黄是什么原因？怎么处理？",
  "greenhouseId": 1
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 128,
    "question": "番茄叶子发黄是什么原因？怎么处理？",
    "answer": "番茄叶子发黄可能由多种原因引起：\n\n1. **缺氮**：老叶先发黄，整体呈淡绿色。建议追施尿素或复合肥。\n2. **缺铁**：新叶发黄但叶脉保持绿色。建议喷施螯合铁叶面肥。\n3. **浇水过多**：根系缺氧导致黄叶。应控制浇水量，保持土壤见干见湿。\n4. **病害**：如黄化曲叶病毒病，需及时拔除病株并防治烟粉虱。\n\n建议先检查土壤湿度和近期施肥情况，结合大棚内环境参数综合判断。",
    "inputType": "TEXT",
    "sources": [
      {"title": "番茄常见缺素症诊断与防治", "category": "栽培技术"},
      {"title": "大棚番茄水肥管理要点", "category": "水肥管理"}
    ],
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

### 2. 语音问答

**请求：**
```
POST /api/v1/qa/ask/voice
Authorization: Bearer <token>
Content-Type: multipart/form-data

audio: (binary file, .wav/.mp3/.amr, max 30MB)
greenhouseId: 1
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 129,
    "question": "大棚里温度太高了咋整？",
    "rawDialectText": "大棚里头温度忒高了咋整？",
    "answer": "大棚温度过高时建议采取以下措施：\n\n1. **立即通风**：打开通风风机，促进空气对流\n2. **遮阳降温**：展开遮阳网，减少阳光直射\n3. **喷雾降温**：可适当进行叶面喷雾，利用蒸发降温\n4. **检查设备**：确认温度传感器读数是否准确\n\n当前您的1号大棚东侧温度为28.5°C，在正常范围内。如温度持续升高超过32°C，系统将自动触发预警。",
    "inputType": "VOICE",
    "asrEngine": "xunfei",
    "dialect": "hebei",
    "sources": [
      {"title": "大棚夏季温度管理技术", "category": "环境调控"}
    ],
    "createdAt": "2026-07-11T10:35:00"
  }
}
```

### 3. 获取问答历史

**请求：**
```
GET /api/v1/qa/records?page=1&size=10
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
        "id": 128,
        "question": "番茄叶子发黄是什么原因？怎么处理？",
        "inputType": "TEXT",
        "createdAt": "2026-07-11T10:30:00"
      },
      {
        "id": 129,
        "question": "大棚里温度太高了咋整？",
        "inputType": "VOICE",
        "asrEngine": "xunfei",
        "createdAt": "2026-07-11T10:35:00"
      }
    ],
    "total": 12,
    "page": 1,
    "size": 10
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 5002 | 500 | 语音识别失败，请尝试文字输入 |
| 5003 | 500 | AI服务暂时不可用 |
| 5004 | 500 | 向量化服务异常 |
| 8001 | 500 | 文件上传失败 |
| 8002 | 400 | 文件大小超过限制 |
| 8003 | 400 | 不支持的文件类型 |
| 1001 | 400 | 参数错误（问题内容为空） |

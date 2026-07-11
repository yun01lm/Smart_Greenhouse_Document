---
id: API-007
title: 病虫害诊断 API
type: api
module: diagnosis
tags: [病虫害, 图像识别, AI, 诊断, 知识图谱]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

病虫害诊断模块接收作物图片，通过 AI 图像识别服务（初期百度AI API，后期 ResNet）识别病害类型，并从 Chroma 向量知识库中检索关联的防治方案。当 AI 识别置信度低于 70% 时，系统会建议用户求助专家进行人工诊断。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| POST | `/api/v1/diagnosis/recognize` | 是 | OWNER/WORKER(需授权) | 上传图片→识别病害 (multipart/form-data) |
| GET | `/api/v1/diagnosis/records` | 是 | OWNER/WORKER | 诊断历史（分页） |

## 请求/响应示例

### 1. 上传图片进行诊断

**请求：**
```
POST /api/v1/diagnosis/recognize
Authorization: Bearer <token>
Content-Type: multipart/form-data

image: (binary file, .jpg/.png, max 10MB)
greenhouseId: 1
```

**响应（成功 - 高置信度）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 42,
    "imagePath": "/uploads/diagnosis/2026/07/11/abc123.jpg",
    "diseaseName": "番茄晚疫病",
    "diseaseCategory": "FUNGAL",
    "confidence": 0.92,
    "description": "由致病疫霉引起的真菌性病害，主要危害叶片和果实...",
    "treatment": "1. 农业防治：及时清除病残体... 2. 化学防治：使用霜脲·锰锌可湿性粉剂...",
    "treatmentDetail": {
      "agricultural": "合理密植，加强通风，及时清除病叶病果",
      "physical": "高温闷棚，覆盖地膜降低湿度",
      "biological": "使用木霉菌制剂预防",
      "chemical": "霜脲·锰锌800倍液或嘧菌酯1500倍液喷雾，间隔7-10天",
      "precautions": ["施药时注意安全防护", "采收前7天停止用药"]
    },
    "recognitionEngine": "baidu",
    "needExpertHelp": false,
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

**响应（成功 - 低置信度）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 43,
    "imagePath": "/uploads/diagnosis/2026/07/11/def456.jpg",
    "diseaseName": "疑似黄瓜霜霉病",
    "diseaseCategory": "FUNGAL",
    "confidence": 0.58,
    "description": "AI识别置信度较低，建议求助专家确认",
    "treatment": "暂提供通用防治建议...",
    "recognitionEngine": "baidu",
    "needExpertHelp": true,
    "suggestMessage": "AI识别置信度为58%，建议点击下方按钮求助专家进行人工诊断",
    "createdAt": "2026-07-11T10:35:00"
  }
}
```

### 2. 获取诊断历史

**请求：**
```
GET /api/v1/diagnosis/records?greenhouseId=1&page=1&size=10
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
        "id": 42,
        "imagePath": "/uploads/diagnosis/2026/07/11/abc123.jpg",
        "diseaseName": "番茄晚疫病",
        "confidence": 0.92,
        "recognitionEngine": "baidu",
        "expertConsulted": false,
        "createdAt": "2026-07-11T10:30:00"
      }
    ],
    "total": 5,
    "page": 1,
    "size": 10
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 5001 | 500 | 图像识别失败，请重试 |
| 4001 | 404 | 大棚不存在 |
| 3002 | 403 | 无该大棚访问权限 |
| 3003 | 403 | 无该功能使用权限 |
| 8001 | 500 | 文件上传失败 |
| 8002 | 400 | 文件大小超过限制（最大10MB） |
| 8003 | 400 | 不支持的文件类型（仅支持jpg/png） |

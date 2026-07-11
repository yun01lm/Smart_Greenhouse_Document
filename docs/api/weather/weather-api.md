---
id: API-008c
title: 天气信息 API
type: api
module: weather
tags: [天气, 预报, 和风天气, 预警辅助]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

天气信息模块对接和风天气 API，提供大棚所在地区的当前天气和未来天气预报。天气数据每3小时定时拉取并缓存到 MySQL weather_cache 表中，同时辅助预警引擎和多模态融合分析进行风险预判。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/weather/current` | 是 | OWNER/WORKER | 当前天气 |
| GET | `/api/v1/weather/forecast` | 是 | OWNER/WORKER | 天气预报（3天/7天） |

## 请求/响应示例

### 1. 当前天气

**请求：**
```
GET /api/v1/weather/current?greenhouseId=1
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "location": "石家庄市藁城区",
    "temperature": 28.5,
    "feelsLike": 30.1,
    "humidity": 65,
    "weatherCode": "100",
    "weatherText": "晴",
    "windSpeed": 12.5,
    "windDirection": "南风",
    "pressure": 1012,
    "visibility": 15,
    "updatedAt": "2026-07-11T10:00:00"
  }
}
```

### 2. 天气预报

**请求：**
```
GET /api/v1/weather/forecast?greenhouseId=1&days=3
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "location": "石家庄市藁城区",
    "forecasts": [
      {
        "date": "2026-07-12",
        "tempMax": 32.0,
        "tempMin": 22.0,
        "weatherCode": "101",
        "weatherText": "多云",
        "humidity": 60,
        "windSpeed": 15.0,
        "precipitation": 0.0
      },
      {
        "date": "2026-07-13",
        "tempMax": 34.0,
        "tempMin": 23.0,
        "weatherCode": "100",
        "weatherText": "晴",
        "humidity": 55,
        "windSpeed": 10.0,
        "precipitation": 0.0
      },
      {
        "date": "2026-07-14",
        "tempMax": 30.0,
        "tempMin": 21.0,
        "weatherCode": "305",
        "weatherText": "小雨",
        "humidity": 75,
        "windSpeed": 18.0,
        "precipitation": 5.2
      }
    ],
    "updatedAt": "2026-07-11T10:00:00"
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 4001 | 404 | 大棚不存在 |
| 3002 | 403 | 无该大棚访问权限 |
| 1003 | 500 | 天气服务暂时不可用 |
| 1001 | 400 | 参数错误 |

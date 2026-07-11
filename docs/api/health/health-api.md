---
id: API-008b
title: 健康综合评估 API
type: api
module: health
tags: [健康评分, 多模态融合, 环境健康, 视觉健康]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

健康综合评估模块通过多模态融合分析引擎，综合环境时序数据（60%权重）和视觉长势数据（40%权重）计算综合健康评分（0-100分）。融合引擎还结合天气预报数据进行未来风险预判。当评分低于阈值时，会触发预警流。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/health/score` | 是 | OWNER/WORKER | 当前综合健康评分 |
| GET | `/api/v1/health/history` | 是 | OWNER/WORKER | 健康评分历史（分页） |
| GET | `/api/v1/health/detail/{id}` | 是 | OWNER/WORKER | 详细评估报告 |

## 请求/响应示例

### 1. 当前综合健康评分

**请求：**
```
GET /api/v1/health/score?greenhouseId=1
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 30,
    "greenhouseId": 1,
    "overallScore": 85.2,
    "envScore": 88.0,
    "visualScore": 81.0,
    "weatherRisk": "低风险",
    "analysisJson": {
      "envDetail": {
        "tempScore": 90,
        "humidityScore": 85,
        "co2Score": 88,
        "soilScore": 89
      },
      "visualDetail": {
        "leafHealth": 82,
        "growthRate": 80,
        "diseaseRisk": 85
      }
    },
    "recommendations": "大棚整体健康状况良好。环境参数在适宜范围内，作物长势正常。建议保持当前管理策略，注意关注天气预报，防范突发降温。",
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

### 2. 健康评分历史

**请求：**
```
GET /api/v1/health/history?greenhouseId=1&startDate=2026-07-01&endDate=2026-07-11&page=1&size=10
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
        "id": 30,
        "overallScore": 85.2,
        "envScore": 88.0,
        "visualScore": 81.0,
        "weatherRisk": "低风险",
        "createdAt": "2026-07-11T10:30:00"
      },
      {
        "id": 29,
        "overallScore": 84.8,
        "envScore": 87.5,
        "visualScore": 80.5,
        "weatherRisk": "低风险",
        "createdAt": "2026-07-11T10:00:00"
      }
    ],
    "total": 22,
    "page": 1,
    "size": 10
  }
}
```

### 3. 详细评估报告

**请求：**
```
GET /api/v1/health/detail/30
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 30,
    "greenhouseId": 1,
    "greenhouseName": "1号番茄大棚",
    "overallScore": 85.2,
    "envScore": 88.0,
    "visualScore": 81.0,
    "weatherRisk": "低风险",
    "analysisJson": {
      "envDetail": {
        "tempScore": 90,
        "tempComment": "温度在25-30°C适宜范围内",
        "humidityScore": 85,
        "humidityComment": "湿度正常，略偏高但无病害风险",
        "co2Score": 88,
        "co2Comment": "CO2浓度正常",
        "soilScore": 89,
        "soilComment": "土壤温湿度及养分正常"
      },
      "visualDetail": {
        "leafHealth": 82,
        "leafHealthComment": "叶片颜色正常，无明显病斑",
        "growthRate": 80,
        "growthRateComment": "生长速度正常，处于开花期",
        "diseaseRisk": 85,
        "diseaseRiskComment": "无病虫害迹象"
      },
      "weatherImpact": {
        "currentWeather": "晴，28°C",
        "forecast": "未来24小时多云转晴",
        "riskAssessment": "无极端天气风险"
      }
    },
    "recommendations": "大棚整体健康状况良好。环境参数在适宜范围内，作物长势正常。建议保持当前管理策略。",
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 4001 | 404 | 大棚不存在 |
| 1002 | 404 | 评估报告不存在 |
| 3002 | 403 | 无该大棚访问权限 |
| 1001 | 400 | 参数错误 |

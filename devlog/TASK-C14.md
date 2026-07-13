---
id: TASK-C14
title: 天气API对接开发
module: C14 天气API
type: task
priority: high
status: completed
created: 2026-07-13
started: 2026-07-13
completed: 2026-07-13
assignee: AI助手
related_docs:
  - DEVLOG-步骤15
tags: [weather, qweather, cache]
---

# 天气API对接开发

---

## 任务描述

### 背景

C14 天气API对接是智慧大棚AIoT系统的环境感知辅助模块。通过和风天气 API 获取大棚所在地区的天气数据，为预警引擎和多模态融合分析提供天气参考。

### 目标

完成天气数据的获取、缓存和查询功能，支持当前天气和未来天气预报。

### 范围

仅后端代码，不涉及前端APP/Web界面。

---

## 验收标准

1. [x] WeatherCache 实体创建完成，与 DB 表 19 一致
2. [x] 和风天气 API 调用正常（当前天气 + 3天/7天预报）
3. [x] 缓存策略：3小时内返回缓存，过期自动刷新
4. [x] API 端点：GET /api/v1/weather/current、GET /api/v1/weather/forecast
5. [x] API 失败时返回友好错误提示

---

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `entity/WeatherCache.java` | 新建 | DB表19，3小时缓存策略 |
| `repository/WeatherCacheRepository.java` | 新建 | 按位置查询最新缓存 |
| `module/weather/service/QWeatherService.java` | 新建 | 和风天气API：当前天气+预报，缓存优先模式 |
| `module/weather/controller/WeatherController.java` | 新建 | 2个端点：/current、/forecast |
| `module/weather/dto/WeatherCurrentResponse.java` | 新建 | 当前天气DTO（含体感温度/气压/能见度） |
| `module/weather/dto/WeatherForecastResponse.java` | 新建 | 预报DTO（含DayForecast子类） |

---

## 完成结果

天气模块提供当前天气+预报查询，缓存策略减少API调用次数。3小时内重复查询不消耗API配额。

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-13
- 关联DEVLOG：步骤15
- 缓存时间可配置：修改 CACHE_DURATION_HOURS 常量

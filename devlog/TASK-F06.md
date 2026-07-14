---
id: TASK-F06
title: 历史数据模块开发
type: task
module: F6 APP历史
tags: [android, history, chart, mpandroidchart, timeseries]
status: completed
created: 2026-07-13
completed: 2026-07-13
author: AI助手
devlog_step: 26
dependencies: [TASK-C01, TASK-C05]
---

# TASK-F06: 历史数据模块开发

> APP端历史数据查询模块，支持传感器类型选择、时间范围切换、趋势曲线图展示。

---

## 任务概述

开发Android端历史数据查询功能，用户可从看板页进入历史数据页面，选择传感器类型和时间范围，查看趋势曲线图（平均值/最大值/最小值）。

## 功能列表

| 序号 | 功能 | 说明 | 状态 |
|------|------|------|------|
| 1 | 传感器类型选择 | Spinner选择11种传感器类型 | ✅ |
| 2 | 时间范围切换 | ChipGroup切换1h/24h/7d/30d | ✅ |
| 3 | 趋势曲线图 | MPAndroidChart LineChart（3条线） | ✅ |
| 4 | 聚合粒度自适应 | 时间范围越大聚合粒度越大 | ✅ |
| 5 | 看板入口 | 看板页并排卡片入口 | ✅ |

## 架构设计

```
HistoryActivity (UI层)
  ├── Toolbar (返回)
  ├── Spinner → 传感器类型选择
  ├── ChipGroup → 时间范围切换
  └── LineChart → MPAndroidChart趋势曲线
       ├── 平均值 (蓝色实线)
       ├── 最大值 (红色虚线)
       └── 最小值 (绿色虚线)

HistoryViewModel (业务逻辑层)
  ├── SensorTypeItem[] 内置11种传感器
  ├── selectSensorType(String) → loadHistory()
  ├── selectTimeRange(String) → loadHistory()
  └── loadHistory() → 计算时间参数 + 调用API

GreenhouseRepository (数据层)
  └── getHistory(greenhouseId, sensorType, start, end, agg)
       → GET /api/v1/sensors/history
```

## 时间范围与聚合粒度

| 时间范围 | 聚合粒度 | X轴标签格式 |
|----------|----------|-------------|
| 1小时 | 1分钟 | HH:mm |
| 24小时 | 30分钟 | HH:mm |
| 7天 | 6小时 | MM/dd |
| 30天 | 1天 | MM/dd |

## 开发规范检查

| 规范要求 | 状态 |
|----------|------|
| Activity不含业务逻辑 | ✅ 通过 |
| ViewModel不持有Context | ✅ 通过 |
| 网络请求在后台线程 | ✅ 通过（ExecutorService） |
| 图表库使用标准库 | ✅ MPAndroidChart v3.1.0 |

## 变更文件

### 新增文件 (6个)
- `data/model/HistoryDataPoint.java`
- `data/model/HistoryResponse.java`
- `viewmodel/HistoryViewModel.java`
- `ui/history/HistoryActivity.java`
- `res/layout/activity_history.xml`
- `res/drawable/ic_history.xml`

### 修改文件 (5个)
- `data/api/GreenhouseApiService.java`
- `data/repository/GreenhouseRepository.java`
- `ui/dashboard/DashboardFragment.java`
- `res/layout/fragment_dashboard.xml`
- `AndroidManifest.xml`

---

## API接口

```
GET /api/v1/sensors/history?greenhouseId=&sensorType=&startTime=&endTime=&aggregation=
```

---

## 关联任务

- **TASK-C01**（用户认证）：API鉴权依赖
- **TASK-C05**（时序数据服务）：后端InfluxDB查询

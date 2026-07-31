---
id: TASK-PERF01
title: 传感器数据Caffeine缓存
module: sensor
type: task
priority: medium
status: completed
created: 2026-08-01
started: 2026-08-01
completed: 2026-08-01
assignee: AI助手
related_docs: [CODE_REVIEW.md]
tags: [性能优化, 缓存, Caffeine]
---

# 传感器数据 Caffeine 缓存

## 任务描述
为 `SensorDataService.getRealtimeData()` 添加 Caffeine 本地缓存，减少 InfluxDB 重复查询。

## 验收标准
1. [x] 5秒内重复请求命中缓存
2. [x] 最大100条目，防止内存溢出
3. [x] 缓存 Key 为 greenhouseId

## 关键修改
| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `backend/pom.xml` | 修改 | 新增 caffeine 依赖 |
| `backend/.../config/CacheConfig.java` | 新建 | Caffeine CacheManager 配置 |
| `backend/.../sensor/service/SensorDataService.java` | 修改 | @Cacheable 注解 |

## 阻塞记录
无阻塞

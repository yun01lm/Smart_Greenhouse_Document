---
id: TASK-G07
title: 系统监控界面开发
module: G7 Web系统监控
type: task
priority: high
status: completed
created: 2026-07-15
started: 2026-07-15
completed: 2026-07-15
assignee: AI助手
related_docs:
  - DEVLOG-步骤41
tags: [web, vue3, monitor, admin, element-plus]
---

# TASK-G07: 系统监控界面开发

## 需求描述

Web 管理端系统监控界面。管理员（ADMIN 角色）可以查看系统运行状态，包括设备在线率、告警统计、服务连接状态（MQTT/数据库）、系统数据概览。

## 设计决策（重要）

### 方案选择：自建监控端点 vs Spring Boot Actuator

| 方案 | 说明 | 优点 | 缺点 |
|------|------|------|------|
| A. Spring Boot Actuator | 引入 `spring-boot-starter-actuator`，调用 /actuator 标准端点 | 信息全面（health/metrics/env 等几十种指标）、Spring 官方标准 | 需引入新依赖、需配置安全暴露端点、数据格式需前端转换、与业务监控需求不匹配 |
| B. 自建监控端点 | 新建 AdminMonitorController，手动聚合各数据源 | 零依赖、数据格式定制化、前端直接可用、只暴露必要信息 | 指标种类不如 Actuator 全面 |

**决策：方案 B（自建监控端点）**。理由：
1. **需求匹配度**：当前 4 项监控需求均为业务级别指标（设备在线率、告警统计、服务连接状态、系统数据概览），Actuator 擅长的是 JVM 技术指标（内存/CPU/GC/线程），两者互补但不重叠。引入 Actuator 不会直接满足当前的业务监控需求。
2. **零依赖**：不增加项目体积和复杂度。
3. **数据格式定制化**：前端直接消费，无需在浏览器端做数据转换。
4. **可扩展**：后续如需技术指标监控（如 JVM 内存使用率），再引入 Actuator 作为补充，两者互不影响。

### 监控内容范围

当前作为**实验版本**支持 4 项监控内容，后续按需扩展：
1. 设备在线率（总数/在线/离线/告警 + 百分比进度条）
2. 告警统计（最近 24h 按级别分组）
3. 服务连接状态（MQTT + 数据库实时检测）
4. 系统数据概览（大棚/设备/用户/规则总数）

## 技术方案

### 后端
- 新建 `MonitorOverviewResponse` DTO：4 个内嵌类（DeviceStats/AlertStats/ServiceStatus/SystemOverview）
- 新建 `AdminMonitorService`：聚合 4 类数据，从不同数据源查询
- 新建 `AdminMonitorController`：1 个 GET 端点 `/api/v1/admin/monitor/overview`
- 修改 `DeviceRepository`：新增 `countByStatus()` 方法
- 零依赖新增，路径复用 SecurityConfig `hasRole('ADMIN')` 保护

### 前端
- Vue 3 Composition API + Element Plus
- 服务状态指示灯（绿/红 + 呼吸光晕效果）
- 设备在线率：4 个大数字 + 百分比进度条
- 告警统计：24h 总数大字 + 三级细分
- 系统概览：2x2 网格 + 彩色图标
- 刷新按钮手动更新

## 后端 API

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/v1/admin/monitor/overview | 综合监控概览（一次性返回全部 4 类数据） |

## 关键修改

### 后端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `module/admin/dto/MonitorOverviewResponse.java` | 100 | 监控响应 DTO（含 4 个内嵌类） |
| `module/admin/service/AdminMonitorService.java` | 135 | 监控数据聚合服务 |
| `module/admin/controller/AdminMonitorController.java` | 40 | 监控 API 控制器 |

### 后端修改文件

| 文件 | 变更 |
|------|------|
| `repository/DeviceRepository.java` | 新增 `countByStatus()` 方法（ADMIN 全量统计用） |

### 前端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `web/src/api/monitor.js` | 10 | 监控 API 封装 |
| `web/src/views/monitor/MonitorPage.vue` | 325 | 系统监控页面 |

### 前端修改文件

| 文件 | 变更 |
|------|------|
| `web/src/router/index.js` | 注册 /monitor 路由 |
| `web/src/layouts/MainLayout.vue` | 启用"系统监控"菜单项（移除 disabled） |

## 数据源说明

| 监控项 | 数据来源 | 查询方式 |
|------|------|------|
| 设备在线率 | MySQL devices 表 | JPA count() + countByStatus() |
| 告警统计 | MySQL alert_records 表 | JPA findAll() + 内存过滤最近 24h + 按级别分组 |
| 服务连接状态 | MqttClient Bean + DataSource | MqttClient.isConnected() + DataSource.getConnection().isValid(3) |
| 系统数据概览 | MySQL 4 张表 | JPA count() 聚合 |

## 构建结果

### 后端
```
mvn compile -pl common,backend → BUILD SUCCESS
零错误零警告
```

### 前端
```
vite build → ✓ built in 993ms
新增产出物：
- dist/assets/MonitorPage-ByMRm9RP.js (6.73 kB / gzip: 2.10 kB)
- dist/assets/MonitorPage-BEvbvcV_.css (3.44 kB / gzip: 0.87 kB)
```

## 已知限制 & 后续扩展

| 项目 | 说明 |
|------|------|
| JVM 技术指标 | 当前不支持。后续可引入 Actuator 补充内存/CPU/GC/线程等指标 |
| 实时刷新 | 当前手动刷新，未使用 WebSocket 或定时轮询。后续可加 30s 自动刷新 |
| 告警趋势 | 当前仅展示 24h 总数，后续可加 7 天趋势折线图 |
| 历史数据 | 无历史快照存储，每次查询为实时数据 |

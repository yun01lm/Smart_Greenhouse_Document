---
id: TASK-G06
title: 数据导出报表开发
module: G6 Web数据导出
type: task
priority: high
status: completed
created: 2026-07-15
started: 2026-07-15
completed: 2026-07-15
assignee: AI助手
related_docs:
  - DEVLOG-步骤40
tags: [web, vue3, excel, poi, report, admin]
---

# TASK-G06: 数据导出报表开发

## 需求描述

Web 管理端数据导出报表。管理员（ADMIN 角色）可以按大棚和时间范围筛选，导出 4 种类型数据为 Excel（.xlsx）文件：
1. 传感器历史数据
2. 预警记录
3. 设备控制日志
4. 健康评分记录

## 设计决策（重要）

### 服务端报表 vs 前端导出

| 方案 | 优点 | 缺点 | 选择 |
|------|------|------|:--:|
| A. 前端导出 | 无需后端改动，开发快 | 大数据量时浏览器卡顿，格式不统一 | ❌ |
| B. 服务端报表 (Apache POI) | 支持大数据量，格式统一美观，多 Sheet 可扩展 | 需引入 POI 依赖，后端开发量稍大 | ✅ |

**决策：方案 B（服务端报表）**。理由：
- 报表是管理端核心功能，格式统一（加粗表头 + 灰底 + 自动列宽）提升专业性
- 传感器历史数据可能很大（7天5分钟粒度 = 2000+ 行），前端生成有性能风险
- Apache POI 是 Java 生态标准库，成熟稳定
- 后续可轻松扩展多 Sheet 报表（如"传感器+预警"合并导出）

### 数据类型范围

当前作为**实验版本**支持 4 种类型，后续按需扩展（如问答记录、诊断记录、场景执行日志等）。

## 技术方案

### 后端
- 引入 Apache POI 5.2.5（poi-ooxml），版本号在父 POM 统一管理
- 新建 `AdminReportService`：4 个导出方法，每个方法独立查询数据源 → 构建 Excel → 返回 byte[]
- 新建 `AdminReportController`：4 个 GET 端点，接收筛选参数，返回 Excel 文件流
- 路径 `/api/v1/admin/report/*`，复用 SecurityConfig `hasRole('ADMIN')` 保护
- 文件响应格式：`Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`

### 前端
- Vue 3 Composition API + Element Plus
- 卡片式布局：通用筛选区 + 4 张导出卡片（2x2 网格）+ 导出说明
- 每张卡片含独立筛选条件，导出按钮带 loading 状态和禁用逻辑
- Blob 下载：`responseType: 'blob'` → `URL.createObjectURL` → `<a>` 点击下载

## 后端 API

| 方法 | 路径 | 说明 | 默认时间范围 |
|------|------|------|:--:|
| GET | /api/v1/admin/report/sensors | 传感器历史数据（?greenhouseId=&sensorType=&startTime=&endTime=） | 7天 |
| GET | /api/v1/admin/report/alerts | 预警记录（?greenhouseId=&level=&startTime=&endTime=） | 30天 |
| GET | /api/v1/admin/report/controls | 控制日志（?greenhouseId=&startTime=&endTime=） | 30天 |
| GET | /api/v1/admin/report/health | 健康评分（?greenhouseId=&startTime=&endTime=） | 30天 |

## 关键修改

### 后端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `module/admin/service/AdminReportService.java` | 290 | 4 种报表 Excel 生成服务 |
| `module/admin/controller/AdminReportController.java` | 105 | 报表导出 API 控制器 |

### 后端修改文件

| 文件 | 变更 |
|------|------|
| `pom.xml`（父POM） | 新增 `poi.version` 属性 |
| `backend/pom.xml` | 新增 `poi-ooxml` 依赖 |

### 前端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `web/src/api/report.js` | 57 | 报表 API 封装 + Blob 下载工具函数 |
| `web/src/views/export/ReportPage.vue` | 285 | 数据导出页面（4 张卡片 + 筛选 + 说明） |

### 前端修改文件

| 文件 | 变更 |
|------|------|
| `web/src/router/index.js` | 注册 /export 路由 |
| `web/src/layouts/MainLayout.vue` | 启用"数据导出"菜单项（移除 disabled） |

## Excel 格式规范

- 表头：加粗 + 灰色背景（GREY_25_PERCENT）+ 底部边框
- 列宽：自动调整，上限 50 字符
- 数字类型：数值字段使用 `setCellValue(double)` 确保 Excel 中为数字格式
- 日期时间：统一 `yyyy-MM-dd HH:mm:ss` 格式
- 中文翻译：预警级别（INFO→提示/WARNING→警告/CRITICAL→严重）、控制来源（MANUAL→手动/SCENE→场景/ALERT→预警）、健康等级（fromScore 动态计算）

## 数据源说明

| 报表类型 | 数据源 | 查询方式 |
|------|------|------|
| 传感器历史 | InfluxDB | 通过 SensorDataService.getHistoryData() 查询 |
| 预警记录 | MySQL alert_records 表 | JPA findByGreenhouseId + 内存时间过滤 |
| 控制日志 | MySQL control_logs 表 | JPA findByDeviceIdIn（先查大棚设备ID列表） |
| 健康评分 | MySQL health_assessments 表 | JPA findByGreenhouseIdAndCreatedAtBetween |

## 构建结果

### 后端
```
mvn compile -pl common,backend → BUILD SUCCESS
零错误零警告
```

### 前端
```
vite build → ✓ built in 999ms
新增产出物：
- dist/assets/ReportPage-BZnABVV8.js (7.73 kB / gzip: 3.06 kB)
- dist/assets/ReportPage-AhzPH9xl.css (1.24 kB / gzip: 0.44 kB)
```

## 已知限制 & 后续扩展

| 项目 | 说明 |
|------|------|
| 传感器历史数据 | 当前仅支持单个传感器类型导出，不支持多类型合并到一个 Sheet |
| 时间范围 | 前端使用日期选择器（天级精度），后端接受 epoch 毫秒，小时级精度需后续支持 |
| 导出类型 | 当前 4 种，后续可添加：问答记录、诊断记录、场景执行日志、用户操作日志 |
| 多 Sheet | 当前每种类型独立导出，后续可支持"综合报表"（一个 xlsx 含多个 Sheet） |
| 异步导出 | 当前同步生成，数据量极大时可能超时。后续可改为异步任务 + 下载链接 |

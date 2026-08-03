---
id: API-REPORT
title: 数据导出 API（OWNER/WORKER）
type: api
module: report
tags: [导出, Excel, 报表, OWNER, WORKER]
status: active
created: 2026-08-04
last_modified: 2026-08-04
author: AI助手
related_docs:
  - PLAN-WEB-ADMIN
---

## 概述

数据导出提供棚主（OWNER）与其名下技术员（WORKER）将大棚数据导出为 Excel（.xlsx）的能力，覆盖传感器历史数据、预警记录、设备控制日志、健康评分四类。

**认证方式**：JWT Bearer Token
**角色**：仅 OWNER / WORKER（SecurityConfig 统一收口）
**细粒度权限**：OWNER 只能导出本人大棚；WORKER 只能导出被授权管理的大棚（employee_permissions 校验），否则返回 403（3002 无该大棚访问权限）。

> 注：系统管理员的系统级导出接口为 `/api/v1/admin/report/**`（仅 ADMIN，前端不暴露）。

## 端点列表

| 方法 | 路径 | 权限 | 说明 |
|------|------|------|------|
| GET | `/api/v1/report/sensors` | OWNER/WORKER | 导出传感器历史数据（Excel） |
| GET | `/api/v1/report/alerts` | OWNER/WORKER | 导出预警记录（Excel） |
| GET | `/api/v1/report/controls` | OWNER/WORKER | 导出设备控制日志（Excel） |
| GET | `/api/v1/report/health` | OWNER/WORKER | 导出健康评分记录（Excel） |

## 参数说明

### GET /api/v1/report/sensors
| 参数 | 必填 | 说明 |
|------|------|------|
| greenhouseId | 是 | 大棚 ID（须有权限） |
| sensorType | 是 | 传感器类型（TEMP/HUMIDITY/LIGHT/CO2 等） |
| startTime | 否 | 开始时间（epoch 毫秒），默认最近 7 天 |
| endTime | 否 | 结束时间（epoch 毫秒），默认当前 |

### GET /api/v1/report/alerts
| 参数 | 必填 | 说明 |
|------|------|------|
| greenhouseId | 是 | 大棚 ID（须有权限） |
| level | 否 | 预警级别（INFO/WARNING/CRITICAL），缺省全部 |
| startTime / endTime | 否 | 时间范围，默认最近 30 天 |

### GET /api/v1/report/controls / health
| 参数 | 必填 | 说明 |
|------|------|------|
| greenhouseId | 是 | 大棚 ID（须有权限） |
| startTime / endTime | 否 | 时间范围，默认最近 30 天 |

## 响应

- 成功：`200`，`application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`，`Content-Disposition: attachment; filename*=UTF-8''<文件名>`，body 为 xlsx 字节流
- 无权限：`403`（3001 无访问权限 / 3002 无该大棚访问权限）
- 大棚不存在：`400`（4001 大棚不存在）

## 实现位置
- 后端：`backend/src/main/java/com/greenhouse/module/report/controller/ReportController.java`、`module/report/service/ReportAccessService.java`（Excel 生成复用 `module/admin/service/AdminReportService.java`）
- 前端：`web/src/api/report.js`、`web/src/views/export/ReportPage.vue`

---
id: TASK-G02
title: 设备管理界面开发
module: G2 Web设备管理
type: task
priority: high
status: completed
created: 2026-07-14
started: 2026-07-14
completed: 2026-07-14
assignee: AI助手
related_docs:
  - DEVLOG-步骤33
tags: [web, vue3, device-management, element-plus]
---

# TASK-G02: 设备管理界面开发

## 需求描述

Web 端设备管理界面，提供设备列表的增删改查和设备分组管理功能。使用已有的后端 REST API。

## 技术方案

- Vue 3 Composition API + Element Plus
- Tab 切换：设备列表 + 设备分组两个面板
- 前端分页、筛选、搜索
- 穿梭框（Transfer）用于分组设备分配

## 后端API

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/v1/greenhouses/{id}/devices | 设备列表（支持 type/status 筛选） |
| POST | /api/v1/greenhouses/{id}/devices | 添加设备 |
| PUT | /api/v1/greenhouses/{id}/devices/{deviceId} | 更新设备 |
| DELETE | /api/v1/greenhouses/{id}/devices/{deviceId} | 删除设备 |
| GET | /api/v1/greenhouses/{id}/device-groups | 分组列表 |
| POST | /api/v1/greenhouses/{id}/device-groups | 创建分组 |
| PUT | /api/v1/greenhouses/{id}/device-groups/{groupId} | 更新分组 |
| DELETE | /api/v1/greenhouses/{id}/device-groups/{groupId} | 删除分组 |
| POST | /api/v1/greenhouses/{id}/device-groups/{groupId}/devices/{deviceId} | 添加设备到分组 |
| DELETE | /api/v1/greenhouses/{id}/device-groups/{groupId}/devices/{deviceId} | 从分组移除设备 |

## 关键修改

### 新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `web/src/api/device.js` | 60 | 设备 + 分组全部 12 个 API 封装 |
| `web/src/views/devices/DevicePage.vue` | 26 | Tab 容器页 |
| `web/src/views/devices/DeviceList.vue` | 240 | 设备列表（表格+筛选+CRUD对话框） |
| `web/src/views/devices/DeviceGroup.vue` | 340 | 分组管理（列表+详情+穿梭框分配） |

### 修改文件

| 文件 | 变更 |
|------|------|
| `web/src/router/index.js` | 注册 /devices 路由 |
| `web/src/layouts/MainLayout.vue` | 启用"设备管理"菜单项（移除 disabled） |

## 功能特性

### 设备列表
- 类型筛选（传感器/控制器）、状态筛选（在线/离线/告警）
- 关键词搜索（设备名称/编号）
- 前端分页（10/20/50 条/页）
- 添加/编辑对话框（表单校验：名称/编号/类型必填，传感器类型条件必填）
- 删除确认弹窗

### 设备分组
- 左侧分组列表：创建、编辑、删除分组
- 右侧分组详情：展示组内设备表格
- 穿梭框分配设备（仅传感器类型可选）
- 组内设备移除功能
- 批量分配（diff 计算增量/减量，并发请求）

## 构建结果

```
vite v8.1.4 building for production...
✓ 2270 modules transformed.
✓ built in 1.11s
```

新增产出物：
- `dist/assets/DevicePage-h3I29nL-.css` (2.42 kB)
- `dist/assets/DevicePage-Dp2oBaTA.js` (17.94 kB / gzip: 5.30 kB)

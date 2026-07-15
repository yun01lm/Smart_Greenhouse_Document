---
id: TASK-G05
title: 预警规则配置开发
module: G5 Web预警配置
type: task
priority: high
status: completed
created: 2026-07-15
started: 2026-07-15
completed: 2026-07-15
assignee: AI助手
related_docs:
  - DEVLOG-步骤39
tags: [web, vue3, alert, threshold, element-plus, admin]
---

# TASK-G05: 预警规则配置开发

## 需求描述

Web 管理端预警规则配置界面。管理员（ADMIN 角色）可以跨大棚管理全量预警规则和用户自定义阈值。不同于棚主端 OWNER 权限校验，管理员可以查看、创建、编辑、删除任意大棚的预警规则和任意用户的阈值。

## 技术方案

### 后端
- 复用 `AlertRuleService` 和 `AlertThresholdService`，新增 ADMIN 专用方法（绕过所有权校验）
- 新建 `AdminAlertController`，路径 `/api/v1/admin/alerts`，SecurityConfig 已有 `/api/v1/admin/**` → `hasRole('ADMIN')` 保护
- `AlertRuleService` 新增：`listAllRules()` / `createRuleAdmin()` / `updateRuleAdmin()` / `deleteRuleAdmin()`
- `AlertThresholdService` 新增：`listAllThresholds()` / `listThresholdsByGreenhouse()` / `deleteThresholdAdmin()`

### 前端
- Vue 3 Composition API + Element Plus
- Tab 切换：预警规则管理 + 自定义阈值管理
- 规则管理：大棚筛选 + 传感器类型筛选 + 表格 + 新建/编辑对话框（含 JSON 校验）+ 开关切换启用 + 删除确认
- 阈值管理：大棚筛选 + 表格 + 管理员删除

## 后端 API

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| GET | /api/v1/admin/alerts/rules | 规则列表（?greenhouseId=） | ADMIN |
| POST | /api/v1/admin/alerts/rules | 创建规则 | ADMIN |
| PUT | /api/v1/admin/alerts/rules/{id} | 更新规则 | ADMIN |
| DELETE | /api/v1/admin/alerts/rules/{id} | 删除规则 | ADMIN |
| GET | /api/v1/admin/alerts/thresholds | 阈值列表（?greenhouseId=） | ADMIN |
| DELETE | /api/v1/admin/alerts/thresholds/{id} | 删除阈值 | ADMIN |

## 关键修改

### 后端修改文件

| 文件 | 变更 |
|------|------|
| `module/alert/service/AlertRuleService.java` | 新增 4 个 ADMIN 专用方法（listAllRules/createRuleAdmin/updateRuleAdmin/deleteRuleAdmin），绕过 OWNER 所有权校验 |
| `module/alert/service/AlertThresholdService.java` | 新增 3 个 ADMIN 专用方法（listAllThresholds/listThresholdsByGreenhouse/deleteThresholdAdmin），绕过用户校验 |

### 后端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `module/admin/controller/AdminAlertController.java` | 70 | 管理员预警配置 API 控制器 |

### 前端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `web/src/api/alert-rule.js` | 48 | 预警规则 + 阈值 API 封装（6 个接口） |
| `web/src/views/alerts/AlertRulePage.vue` | 395 | 预警规则配置页面（规则管理 + 阈值管理） |

### 前端修改文件

| 文件 | 变更 |
|------|------|
| `web/src/router/index.js` | 注册 /alerts 路由 |
| `web/src/layouts/MainLayout.vue` | 启用"预警配置"菜单项（移除 disabled） |

## 设计决策

- **ADMIN 专用方法 vs 权限绕过**：选择在 Service 层新增 ADMIN 方法而非修改现有方法加角色判断。理由：保持现有 OWNER 权限逻辑不变，ADMIN 方法有明确的前缀标识，日志区分清晰
- **阈值管理只读**：管理员可查看和删除阈值，但不支持创建/编辑。理由：阈值是用户个性化设置，管理员强行修改不合适；删除场景为清理无效数据
- **规则条件 JSON 校验**：前端使用 `JSON.parse` 校验格式合法性，按规则类型显示格式提示（THRESHOLD/TREND/COMPOSITE/WEATHER）

## 构建结果

### 后端
```
mvn compile -pl common,backend → BUILD SUCCESS
零错误零警告
```

### 前端
```
vite build → ✓ built in 1.01s
新增产出物：
- dist/assets/AlertRulePage-C2n3ymTa.js (13.01 kB / gzip: 4.06 kB)
- dist/assets/AlertRulePage-DAF8y3E1.css (0.71 kB / gzip: 0.33 kB)
```

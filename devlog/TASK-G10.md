---
id: TASK-G10
title: 棚主 Web 管理开发
module: G10 Web棚主管理
type: task
priority: high
status: completed
created: 2026-07-15
started: 2026-07-15
completed: 2026-07-15
assignee: AI助手
related_docs:
  - DEVLOG-步骤44
tags: [web, vue3, owner, admin, element-plus]
---

# TASK-G10: 棚主 Web 管理开发

## 需求描述

Web 管理端棚主管理。管理员可查看所有棚主账号（聚合大棚数和员工数）、点击查看棚主名下大棚详情。

这是 Web 管理端（G01-G10）的最后一个任务，Web 端全部 10 个模块现已全部完成。

## 技术方案

### 后端
- 新建 `AdminOwnerController`，复用已有 `UserRepository` 和 `GreenhouseRepository`
- 棚主列表聚合 `countByOwnerId`（大棚数）和 `countByRoleAndOwnerId`（员工数）
- 大棚详情直接查 `findByOwnerId` 返回完整信息
- 零依赖新增，路径复用 SecurityConfig `hasRole('ADMIN')`

### 前端
- Vue 3 Composition API + Element Plus
- 棚主表格 + 点击弹出大棚详情弹窗

## 后端 API

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/v1/admin/owners | 棚主列表（含大棚数/员工数） |
| GET | /api/v1/admin/owners/{id}/greenhouses | 棚主名下大棚详情 |

## 关键修改

### 后端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `module/admin/controller/AdminOwnerController.java` | 95 | 棚主管理 API |

### 前端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `web/src/api/owner.js` | 12 | 棚主 API 封装 |
| `web/src/views/owner/OwnerPage.vue` | 170 | 棚主管理页面 |

### 前端修改文件

| 文件 | 变更 |
|------|------|
| `web/src/router/index.js` | 注册 /owner 路由 |
| `web/src/layouts/MainLayout.vue` | 启用"棚主管理"菜单项（移除 disabled） |

## 构建结果

### 后端
```
mvn compile -pl common,backend → BUILD SUCCESS
```

### 前端
```
vite build → ✓ built in 1.00s
新增产出物：
- dist/assets/OwnerPage-DXEt1gik.js (3.82 kB / gzip: 1.59 kB)
- dist/assets/OwnerPage-XRsRb9zJ.css (0.37 kB / gzip: 0.20 kB)
```

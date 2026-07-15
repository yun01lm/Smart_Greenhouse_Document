---
id: TASK-G09
title: 专家工作台开发
module: G9 Web专家工作台
type: task
priority: high
status: completed
created: 2026-07-15
started: 2026-07-15
completed: 2026-07-15
assignee: AI助手
related_docs:
  - DEVLOG-步骤43
tags: [web, vue3, expert, authorization, admin, element-plus]
---

# TASK-G09: 专家工作台开发

## 需求描述

Web 管理端专家工作台。管理员可查看全量专家列表（含在线状态和咨询数）、管理专家在线状态、查看所有授权记录（分页+状态筛选）、查看工作台统计数据。

## 技术方案

### 后端
- 复用已有 `ExpertService` 和 Repository，新建 ADMIN 专用 `AdminExpertService` + `AdminExpertController`
- 授权记录支持全量分页查询（不限用户范围），按状态筛选
- 扩展 3 个 Repository 添加 ADMIN 查询方法
- 零依赖新增，路径复用 SecurityConfig `hasRole('ADMIN')`

### 前端
- Vue 3 Composition API + Element Plus
- 四色统计卡片 + 双 Tab（专家列表 + 授权管理）
- 在线开关直接调用 API 切换

## 后端 API

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/v1/admin/experts | 专家列表（含在线状态/咨询数） |
| PUT | /api/v1/admin/experts/{id}/online | 切换专家在线状态（?online=true/false） |
| GET | /api/v1/admin/experts/authorizations | 全量授权记录（?status=&page=&size=） |
| GET | /api/v1/admin/experts/stats | 工作台统计（专家数/在线数/授权数/会话数） |

## 关键修改

### 后端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `module/admin/service/AdminExpertService.java` | 130 | 专家工作台服务 |
| `module/admin/controller/AdminExpertController.java` | 70 | 专家工作台 API |

### 后端修改文件

| 文件 | 变更 |
|------|------|
| `repository/DataAuthorizationRepository.java` | 新增 `findByStatus()` 分页查询 |
| `repository/ExpertAvailabilityRepository.java` | 新增 `countByIsOnline()` 计数 |
| `repository/ChatConversationRepository.java` | 新增 `countByExpertId()` 咨询数统计 |

### 前端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `web/src/api/expert.js` | 20 | 专家工作台 API 封装 |
| `web/src/views/expert/ExpertPage.vue` | 260 | 专家工作台页面 |

### 前端修改文件

| 文件 | 变更 |
|------|------|
| `web/src/router/index.js` | 注册 /expert 路由 |
| `web/src/layouts/MainLayout.vue` | 启用"专家工作台"菜单项（移除 disabled） |

## 构建结果

### 后端
```
mvn compile -pl common,backend → BUILD SUCCESS
零错误零警告
```

### 前端
```
vite build → ✓ built in 936ms
新增产出物：
- dist/assets/ExpertPage-Pa2-p3h3.js (6.29 kB / gzip: 2.39 kB)
- dist/assets/ExpertPage-Df1BFeoc.css (0.78 kB / gzip: 0.35 kB)
```

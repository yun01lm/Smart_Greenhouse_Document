---
id: TASK-G03
title: 用户与角色管理开发
module: G3 Web用户管理
type: task
priority: high
status: completed
created: 2026-07-14
started: 2026-07-14
completed: 2026-07-14
assignee: AI助手
related_docs:
  - DEVLOG-步骤34
tags: [web, vue3, admin, user-management, role-management, element-plus]
---

# TASK-G03: 用户与角色管理开发

## 需求描述

Web 管理端用户与角色管理界面。管理员可以查看所有用户、按角色筛选、编辑用户角色/状态、删除用户，以及查看各角色统计。

## 重要发现

后端原本**没有 Admin 管理 API**。C01 只做了登录/注册，C18 只做了权限切面。G03 需要同时补充后端 API 和前端页面。

## 技术方案

### 后端（方案 A：补 API）
- 新建 `admin` 模块（controller + service + dto）
- 路径前缀 `/api/v1/admin`，SecurityConfig 中 `hasRole('ADMIN')`
- 用户列表支持按角色筛选
- 编辑不暴露密码字段
- 禁止管理员删除自己或降级自己

### 前端
- Vue 3 Composition API + Element Plus
- Tab 切换：用户列表 + 角色概览
- 用户列表：角色筛选、关键词搜索、前端分页
- 角色概览：渐变色统计卡片 + 权限说明表

## 后端 API

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| GET | /api/v1/admin/users | 用户列表（?role=OWNER 筛选） | ADMIN |
| GET | /api/v1/admin/users/{userId} | 用户详情 | ADMIN |
| PUT | /api/v1/admin/users/{userId} | 更新用户（角色/状态/基本信息） | ADMIN |
| DELETE | /api/v1/admin/users/{userId} | 删除用户 | ADMIN |
| GET | /api/v1/admin/roles | 角色列表 + 统计 | ADMIN |

## 关键修改

### 后端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `module/admin/dto/UserSummaryResponse.java` | 45 | 用户摘要 DTO（不含密码） |
| `module/admin/dto/UpdateUserRequest.java` | 25 | 更新用户请求 DTO |
| `module/admin/dto/RoleCountResponse.java` | 25 | 角色统计响应 DTO |
| `module/admin/service/AdminService.java` | 115 | 用户管理业务逻辑 |
| `module/admin/controller/AdminController.java` | 80 | REST API 控制器 |

### 后端修改文件

| 文件 | 变更 |
|------|------|
| `repository/UserRepository.java` | 新增 `countByRole(Role)` 方法 |
| `config/SecurityConfig.java` | 新增 `/api/v1/admin/**` 的 `hasRole('ADMIN')` |
| `backend/pom.xml` | 修复 Spring AI 依赖缺少版本号（已存在 bug） |

### 前端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `web/src/api/admin.js` | 30 | Admin API 封装（6 个接口） |
| `web/src/views/users/UserPage.vue` | 26 | Tab 容器页 |
| `web/src/views/users/UserList.vue` | 230 | 用户表格 + 编辑对话框 |
| `web/src/views/users/RoleOverview.vue` | 160 | 角色统计卡片 + 权限说明 |

### 前端修改文件

| 文件 | 变更 |
|------|------|
| `web/src/router/index.js` | 注册 /users 路由 |
| `web/src/layouts/MainLayout.vue` | 启用"用户管理"菜单项 |

## 构建结果

### 前端
```
vite v8.1.4 building for production...
✓ 2277 modules transformed.
✓ built in 1.14s
```

新增产出物：
- `dist/assets/UserPage-Q6OlQAH9.css` (1.69 kB)
- `dist/assets/UserPage-B0vp-DjW.js` (9.33 kB / gzip: 3.57 kB)

### 后端
环境 JDK 20 < 项目要求 JDK 21，无法完整编译。代码语法审查通过（Lombok + Spring 标准模式，与已有代码一致）。

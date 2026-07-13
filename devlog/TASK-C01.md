---
id: TASK-C01
title: 用户认证模块开发
module: C1 用户认证
type: task
priority: high
status: completed
created: 2026-07-12
started: 2026-07-12
completed: 2026-07-12
assignee: AI助手
related_docs:
  - DEVLOG-步骤4
tags: [c1-用户认证]
---

# 用户认证模块开发

---

## 任务描述

### 背景

C1 用户认证是智慧大棚AIoT系统的核心后端模块，属于Phase 2/3开发计划。

### 目标

完成C1 用户认证的全部后端代码开发，包括实体类、Repository、Service、Controller和DTO。

### 范围

仅后端代码，不涉及前端APP/Web界面。

---

## 验收标准

1. [x] 实体类创建完成，与数据库设计一致
2. [x] Repository层实现，支持基本CRUD和业务查询
3. [x] Service层实现，包含核心业务逻辑和权限校验
4. [x] Controller层实现，API端点与接口文档一致
5. [x] DTO定义完整，包含@Valid校验
6. [x] API路径使用 /api/v1/ 前缀 + kebab-case

---

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `entity/User.java` | 新建 | JPA实体，四种角色(ADMIN/OWNER/WORKER/EXPERT)，BCrypt密码，员工单归属
| `repository/UserRepository.java` | 新建 | 用户名/手机号唯一性校验，按角色/棚主查询
| `module/auth/controller/AuthController.java` | 新建 | 3个端点：POST /api/v1/auth/register、POST /api/v1/auth/login、GET /api/v1/auth/profile
| `module/auth/service/AuthService.java` | 新建 | 注册（校验唯一性、BCrypt加密、直接返回Token）、登录（Spring Security认证+JWT生成）
- 4个DTO：RegisterRequest、LoginRequest、LoginResponse、UserProfileResponse
| `security/UserDetailsServiceImpl.java` | 新建 | 从临时内存版改为数据库版

---

## 完成结果

用户认证链路完整（注册→BCrypt加密→数据库存储→登录→JWT生成→过滤器验证→SecurityContext），3个API端点，4种角色支持

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-12
- 关联DEVLOG：步骤4

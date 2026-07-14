---
id: TASK-C18
title: 多角色权限模块开发
module: C18 多角色权限
type: task
priority: high
status: completed
created: 2026-07-12
started: 2026-07-12
completed: 2026-07-12
assignee: AI助手
related_docs:
  - DEVLOG-步骤7
tags: [c18-多角色权限]
---

# 多角色权限模块开发

---

## 任务描述

### 背景

C18 多角色权限是智慧大棚AIoT系统的核心后端模块，属于Phase 2/3开发计划。

### 目标

完成C18 多角色权限的全部后端代码开发，包括实体类、Repository、Service、Controller和DTO。

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
| `entity/EmployeePermission.java` | 新建 | DB第8号表，6个功能权限布尔字段
| `repository/EmployeePermissionRepository.java` | 新建 | 按员工/棚主/大棚查询
| `module/permission/controller/PermissionController.java` | 新建 | 棚主端5个端点
| `module/permission/controller/WorkerPermissionController.java` | 新建 | 员工端2个端点
| `module/permission/service/PermissionService.java` | 新建 | 添加员工（单归属校验）、权限更新（6字段按非空更新）
| `security/aop/PermissionAspect.java` | 新建 | AOP切面，@RequireGreenhouseAccess和@RequireFunction实现
- 4个DTO：AddEmployeeRequest、EmployeeResponse、UpdatePermissionRequest、PermissionResponse
- 修改：UserRepository、GreenhouseService、DeviceService、pom.xml

---

## 完成结果

多角色权限体系完整闭环，AOP自动拦截校验，WORKER返回空列表问题全部修复

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-12
- 关联DEVLOG：步骤7
- **G03 变更（2026-07-14）**：SecurityConfig 新增 `/api/v1/admin/**` 路径的 `hasRole('ADMIN')` 限制

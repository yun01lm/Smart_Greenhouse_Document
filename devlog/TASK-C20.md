---
id: TASK-C20
title: 地区管理模块开发
module: C20 地区管理
type: task
priority: high
status: completed
created: 2026-07-12
started: 2026-07-12
completed: 2026-07-12
assignee: AI助手
related_docs:
  - DEVLOG-步骤5
tags: [c20-地区管理]
---

# 地区管理模块开发

---

## 任务描述

### 背景

C20 地区管理是智慧大棚AIoT系统的核心后端模块，属于Phase 2/3开发计划。

### 目标

完成C20 地区管理的全部后端代码开发，包括实体类、Repository、Service、Controller和DTO。

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
- 与 C3 大棚管理合并开发
| `GreenhouseRepository.java` | 新建 | 地区统计查询（按省/市/区/镇/村分组统计大棚数量）
| `GreenhouseController.java` | 新建 | GET /api/v1/greenhouses/regions（地区统计端点）
| `RegionStatsResponse.java` | 新建 | 地区统计DTO，支持五级层级钻取

---

## 完成结果

五级地区统计功能随大棚管理模块一起完成，支持从省到村的逐级钻取

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-12
- 关联DEVLOG：步骤5

---

## 后续变更（R38：地区级联"选不出来"修复，2026-08-28）

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| web/src/components/RegionCascader.vue | 重写 | 懒加载时预查下一级（省→市、市→区县、区县→乡镇、乡镇→村），无下级数据的节点直接标记 leaf 可选中；预查请求加 3 秒超时兜底，级联永不卡死 |

> 根因：无下级数据的节点（如"藁城区"）永远不是叶子，点击只展开空列表、无法选中（"假筛选"）。
> 验证：实测选中"河北省/石家庄市/藁城区"并过滤出 6 条记录；已推送 GitHub（cc92eea）。
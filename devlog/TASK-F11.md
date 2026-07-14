---
id: TASK-F11
title: 角色适配功能开发
module: F11 APP角色适配
type: task
priority: high
status: completed
created: 2026-07-14
started: 2026-07-14
completed: 2026-07-14
assignee: AI助手
related_docs:
  - DEVLOG-步骤30
tags: [android, role, permission, MVVM, adapter]
---

# 角色适配功能开发

---

## 任务描述

### 背景

F11 角色适配是 APP 端最后一个任务，让 APP 根据登录用户的角色（OWNER 棚主 / WORKER 员工）自动调整界面和功能可见性。APP 端仅存在这两种角色，无 ADMIN / EXPERT。

### 目标

实现角色感知的 UI 适配：棚主全功能可用，员工按棚主分配的 6 项权限位控制功能可见性。

### 范围

Android 原生开发（Java），涉及工具类、TokenManager 扩展、Repository 补齐、以及 4 个 Fragment/Activity 的角色适配。

---

## 验收标准

1. [x] 创建 RoleAdapter 工具类：角色判断 + 6 项权限检查
2. [x] TokenManager 扩展：savePermissions() + getBoolean() 缓存权限位
3. [x] Repository 补齐：getCurrentUser / getGreenhouse / getHistoryData / getUnreadAlertCount / deleteThreshold
4. [x] DashboardFragment 适配：员工按权限隐藏无权限功能卡片
5. [x] ControlFragment 适配：员工无 can_control_device 时禁用设备控制
6. [x] ProfileFragment 适配：棚主/员工差异化显示功能入口
7. [x] MainActivity 适配：员工按权限隐藏底部 Tab
8. [x] 构建成功（BUILD SUCCESSFUL）

---

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `util/RoleAdapter.java` | 新增 | 角色判断 + 6 项权限检查 + 功能可见性方法 |
| `data/local/TokenManager.java` | 修改 | 新增 savePermissions() + getBoolean() |
| `data/repository/GreenhouseRepository.java` | 修改 | 补齐 5 个缺失的 API 封装方法 |
| `ui/dashboard/DashboardFragment.java` | 修改 | 新增 applyRoleAdapter() 按权限隐藏卡片 |
| `ui/control/ControlFragment.java` | 修改 | 员工无控制权限时显示"无权限"提示 |
| `ui/profile/ProfileFragment.java` | 修改 | 新增 applyRoleAdapter() 角色差异化 |
| `ui/common/MainActivity.java` | 修改 | 新增 applyRoleTabFilter() 按权限隐藏 Tab |

---

## 技术架构

### 角色权限模型

```
┌──────────────────────────────────────────────────┐
│                   RoleAdapter                    │
│                                                  │
│  isOwner() → true → 全权限，不限制               │
│  isWorker() → true → 按 6 位权限控制             │
│                                                  │
│  权限位（SharedPreferences 缓存）：              │
│  ┌─────────────────┬──────────────────────────┐ │
│  │ can_view_data   │ 查看实时/历史数据         │ │
│  │ can_control_device│ 控制设备/执行场景       │ │
│  │ can_diagnose    │ 病虫害诊断               │ │
│  │ can_ask_expert  │ 专家咨询                 │ │
│  │ can_view_alerts │ 查看预警                 │ │
│  │ can_view_history│ 查看历史数据             │ │
│  └─────────────────┴──────────────────────────┘ │
└──────────────────────────────────────────────────┘
```

### 适配层次

| 层次 | 组件 | 控制粒度 |
|------|------|----------|
| Tab 层 | MainActivity | 粗粒度：隐藏整个 Tab |
| 卡片层 | DashboardFragment | 细粒度：隐藏功能入口卡片 |
| 功能层 | ControlFragment | 功能级：禁用设备控制 |
| 入口层 | ProfileFragment | 入口级：角色差异化入口 |

---

## 完成结果

APP 端全部 11 个任务（F01-F11）完成，构建通过。项目从零到完整实现了数据看板、预警中心、病虫害诊断、AI 问答、设备控制、历史数据、长势评估、健康评分、个人中心、专家咨询、角色适配共 11 个功能模块。

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-14
- 关联DEVLOG：步骤30
- APP 端开发任务全部完成
- 后续：Web 端（G01-G10）、ESP32 固件、Docker 部署、文档编写

---
id: TASK-F09
title: 个人中心模块开发
module: F9 APP个人中心
type: task
priority: medium
status: completed
created: 2026-07-13
started: 2026-07-13
completed: 2026-07-13
assignee: AI助手
related_docs:
  - DEVLOG-步骤18
tags: [android, profile, MVVM]
---

# 个人中心模块开发

---

## 任务描述

### 背景

F09 个人中心是 APP 端用户信息展示和设置入口，显示用户名、角色，提供退出登录功能。

### 目标

完成用户信息展示和退出登录。

### 范围

Android Fragment 组件。

---

## 验收标准

1. [x] 显示用户真实姓名（或用户名）
2. [x] 显示角色中文名（棚主/员工/专家/管理员）
3. [x] 显示用户名（@username）
4. [x] 退出登录：清除 Token → 跳转登录页 → 清除回退栈
5. [x] Fragment 不写业务逻辑

---

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `ui/profile/ProfileFragment.java` | 新建 | 个人中心 Fragment |
| `res/layout/fragment_profile.xml` | 新建 | 个人中心布局 |
| `res/drawable/bg_avatar.xml` | 新建 | 头像占位背景 |

### 后续变更（F10 集成）

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `ui/profile/ProfileFragment.java` | 修改 | 新增"专家咨询"+"授权管理"入口跳转 |
| `res/layout/fragment_profile.xml` | 修改 | 新增两个 MaterialButton 入口按钮 |

### 后续变更（F11 集成）

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `ui/profile/ProfileFragment.java` | 修改 | 新增 applyRoleAdapter()，棚主隐藏授权管理，员工按 canAskExpert 控制专家咨询入口 |

---

## 完成结果

个人中心页面展示用户信息（姓名/角色/@用户名），退出登录功能完整（清除 SharedPreferences Token → Intent 跳转 LoginActivity 并清除回退栈）。

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-13
- 关联DEVLOG：步骤18
- 后续可扩展：修改密码、消息通知设置、关于页面

---

## 后续变更（R39：修改密码功能补全，2026-08-28）

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `ProfileFragment.java` | 修改 | btn_change_password 补点击绑定 + 修改密码对话框（原密码/新密码/确认，8 位含字母数字校验），调后端 PUT /auth/password |

> 根因：此前只有 UI 无点击监听，功能完全失效（APP 审查发现）。
> 验证：点击"修改密码"弹出对话框（截图 v3-password）。

---

## 后续变更（R41：F3 农事提醒入口 + UI 深色化，2026-08-28）

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `fragment_profile.xml`/`ProfileFragment.java` | 修改 | "我的"页新增"农事提醒"入口 → FarmingCalendarActivity |
| `ui/farming/FarmingCalendarActivity.java` | 新增 | 农事日历：作物生长周期展示（getCropCycles）+ 本地提醒列表（SharedPreferences）+ 添加提醒 |
| `ui/farming/ReminderReceiver.java` | 新增 | AlarmManager 次日 8:00 通知（BroadcastReceiver） |
| `activity_farming_calendar.xml` | 新增 | 农事日历布局（深色毛玻璃） |

> 验证：我的页入口生效、日历页打开正常；已推送 GitHub。

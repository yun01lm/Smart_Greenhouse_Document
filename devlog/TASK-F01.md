---
id: TASK-F01
title: 实时数据看板开发
module: F1 APP看板
type: task
priority: high
status: completed
created: 2026-07-13
started: 2026-07-13
completed: 2026-07-13
assignee: AI助手
related_docs:
  - DEVLOG-步骤18
  - PRD-001 (dashboard-realtime)
tags: [android, dashboard, realtime, MVVM, websocket]
---

# 实时数据看板开发

---

## 任务描述

### 背景

F01 实时数据看板是 APP 端核心展示模块，用户打开 APP 后首先看到大棚环境数据的可视化界面。包含仪表盘数值展示、大棚切换、健康评分等功能。

### 目标

完成 APP 骨架搭建 + 登录认证 + 大棚选择 + 传感器数据展示 + 健康评分 + WebSocket 实时推送。

### 范围

Android 原生 Java 开发，MVVM 架构，编译通过并生成 APK。

---

## 验收标准

1. [x] Android 项目骨架搭建完成（Gradle 8.5 + AGP 8.2 + SDK 34）
2. [x] 登录页面：用户名/密码输入 → JWT Token 存储到 SharedPreferences
3. [x] 主界面底部 Tab 导航（5 个 Tab）
4. [x] 大棚下拉选择器，加载用户大棚列表
5. [x] 11 种传感器参数实时数据卡片展示
6. [x] 健康评分圆形指示器（5 级颜色）
7. [x] STOMP over OkHttp WebSocket 客户端
8. [x] 所有网络请求在 ExecutorService 子线程执行
9. [x] Activity/Fragment 不写业务逻辑（MVVM）
10. [x] BUILD SUCCESSFUL — APK 编译通过

---

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `app/build.gradle` | 新建 | Android 模块构建配置 |
| `build.gradle` | 新建 | 项目级构建配置 |
| `settings.gradle` | 新建 | 项目设置 |
| `AndroidManifest.xml` | 新建 | 应用清单（权限+Activity声明） |
| `GreenhouseApplication.java` | 新建 | Application 入口 |
| `TokenManager.java` | 新建 | SharedPreferences Token 管理 |
| `ApiClient.java` | 新建 | Retrofit 封装+OkHttp Token 拦截器 |
| `GreenhouseApiService.java` | 新建 | API 接口定义 |
| `StompClient.java` | 新建 | STOMP over OkHttp WebSocket |
| `GreenhouseRepository.java` | 新建 | 数据仓库层 |
| `LoginViewModel.java` | 新建 | 登录 ViewModel |
| `DashboardViewModel.java` | 新建 | 看板 ViewModel |
| `LoginActivity.java` | 新建 | 登录页面 |
| `MainActivity.java` | 新建 | 主界面+底部导航 |
| `DashboardFragment.java` | 新建 | 看板 Fragment |
| `SensorAdapter.java` | 新建 | 传感器数据适配器 |
| 10个数据模型 | 新建 | ApiResponse/LoginRequest/... |
| 20个资源文件 | 新建 | 布局/图标/颜色/主题/菜单 |

---

## 完成结果

Android APP 编译通过（app-debug.apk, 7.2MB）。MVVM 架构完整：View(Activity/Fragment) → ViewModel → Repository → ApiService/StompClient。支持登录认证（JWT Token + SharedPreferences）、大棚选择、11 种传感器实时数据展示（中文名+单位+异常值红色标记）、健康评分 5 级颜色指示器。WebSocket 使用 OkHttp 实现 STOMP 1.2 协议。所有规范要求均已满足。

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-13
- 关联DEVLOG：步骤18
- 编译环境：Linux + Android SDK 34 + Gradle 8.5
- 数据模拟脚本（sensor_simulator.py）配套提供测试数据

## 变更记录

### Step 22 (2026-07-13): 预警合并到看板
- **fragment_dashboard.xml**: 传感器列表上方新增"预警中心"入口卡片（CardView）
- **DashboardFragment.java**: 添加预警入口点击跳转（→ AlertFragment, addToBackStack）
- **MainActivity.java**: 移除独立预警Tab，预警通过看板入口访问
- 变更原因：为设备控制模块(F05)腾出底部导航位置

### Step 26 (2026-07-13): 历史数据入口
- **fragment_dashboard.xml**: 预警卡片和历史数据卡片改为并排布局（各50%宽度）
- **DashboardFragment.java**: 新增历史数据入口点击 → Intent 跳转 HistoryActivity
- 变更原因：新增F06历史数据模块入口

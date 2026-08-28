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

### 后续变更（F11 集成）

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `ui/dashboard/DashboardFragment.java` | 修改 | 新增 applyRoleAdapter()，员工按权限隐藏功能卡片 |
| `ui/common/MainActivity.java` | 修改 | 新增 applyRoleTabFilter()，员工按权限隐藏底部 Tab |

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

### Step 27 (2026-07-13): 长势评估入口
- **fragment_dashboard.xml**: 入口卡片从两列改为三列布局（长势评估 + 历史数据 + 预警中心），每列改为垂直布局（图标+标题+副标题）
- **DashboardFragment.java**: 新增长势评估入口点击 → Intent 跳转 GrowthActivity
- 变更原因：新增F07作物长势评估模块入口

### Step 28 (2026-07-13): 健康评分入口
- **fragment_dashboard.xml**: 顶部健康评分 CardView 增加 id（card_health_score）和 clickable 属性
- **DashboardFragment.java**: 健康评分卡片 → 可点击 → Intent 跳转 HealthActivity
- 变更原因：新增F08多模态健康评分详情页入口

---

## 后续变更（R39/R40：看板传感器卡片中文化 + UI 一期/二期看板重绘，2026-08-28）

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `SensorAdapter.java` | 修改 | NAME_MAP 键对齐后端枚举（TEMPERATURE→空气温度、SOIL_MOISTURE→土壤湿度），看板卡片标题不再显示英文枚举 |
| `item_sensor_card.xml` | 重写 | 16dp 圆角+柔和阴影、圆角状态条、22sp 等宽数字、深色文字 |
| `colors.xml` / `styles.xml` | 重写 | 现代农业绿色彩体系 + 统一卡片/按钮/数字样式（UI 重构一期） |
| `bg_icon_chip_*.xml` | 修改 | 功能入口图标底升级柔和渐变（UI 重构二期） |

> 验证：模拟器截图确认卡片标题中文、看板现代感提升（review_shots/app/p1、p2）。

---

## 后续变更（R41：UI 方向 A 深色科技感 + F1 看板迷你趋势曲线，2026-08-28）

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `colors.xml`(values/values-night) | 重写 | 深色科技感 tokens：墨绿黑底、毛玻璃卡片、荧光绿 #3DDC84 强调、8 类传感器语义色 |
| `styles.xml`/`themes.xml` | 修改 | cardViewStyle 全局毛玻璃卡 + materialButtonStyle 荧光绿胶囊按钮 |
| `fragment_dashboard.xml` | 修改 | 功能入口改 2×2 毛玻璃卡片（长势/历史/预警/健康详情） |
| `SparklineView.java` | 新增 | 零依赖 Canvas 迷你趋势曲线（渐变填充+末点高亮+类型语义色） |
| `item_sensor_card.xml`/`SensorAdapter.java` | 修改 | 传感器卡片内嵌 sparkline，数据来自 sensors/history 近 6h |
| `DashboardViewModel.java`/`DashboardFragment.java` | 修改 | loadTrendData 拉取各类型趋势数据并下发 |

> 验证：模拟器截图确认 sparkline 语义色曲线正常；已推送 GitHub。

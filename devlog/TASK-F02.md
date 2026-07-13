---
id: TASK-F02
title: 环境预警中心开发
module: F2 APP预警
tags: [app, android, alert, threshold]
status: completed
created: 2026-07-13
completed: 2026-07-13
author: AI助手
dependencies: [TASK-C06, TASK-C21, TASK-F01]
related_docs: [PRD-002, API-005]
---

# TASK-F02：环境预警中心开发

## 一、任务概述

在 Android APP 端实现环境预警中心功能，包括预警列表（支持三级筛选）、预警详情查看、标记已读、自定义阈值设置。

## 二、实现内容

### 2.1 预警列表（AlertFragment）

- ChipGroup 实现三级筛选：全部 / 警告(WARNING) / 严重(CRITICAL)
- RecyclerView + AlertAdapter 展示预警列表
- 支持分页加载
- 空状态提示 + 加载指示器
- 点击跳转预警详情页
- 阈值设置按钮入口

### 2.2 预警详情（AlertDetailActivity）

- 接收 Intent 传递的预警数据
- 显示级别标签（颜色区分）、标题、传感器类型/数值/时间、详细内容

### 2.3 自定义阈值（ThresholdSettingsActivity）

- 合并默认传感器类型（8种）与用户已保存的阈值
- 每项可独立编辑最小/最大值
- 批量保存
- 删除阈值功能

### 2.4 业务逻辑（AlertViewModel）

- 分页加载预警列表（支持 level 筛选参数）
- 标记预警为已读
- 阈值 CRUD 操作（GET/POST/PUT/DELETE）

### 2.5 数据层

- GreenhouseApiService：新增阈值 CRUD 接口 + 预警列表 level 筛选参数
- GreenhouseRepository：新增 markAlertRead / getThresholds / setThreshold / deleteThreshold 方法

## 三、新增文件

| 文件 | 类型 | 说明 |
|------|------|------|
| AlertViewModel.java | ViewModel | 预警业务逻辑 |
| AlertAdapter.java | Adapter | 预警列表适配器 |
| ThresholdAdapter.java | Adapter | 阈值编辑适配器 |
| AlertFragment.java | Fragment | 预警列表页面 |
| AlertDetailActivity.java | Activity | 预警详情页 |
| ThresholdSettingsActivity.java | Activity | 阈值编辑页 |
| fragment_alert.xml | Layout | 预警列表布局 |
| item_alert_card.xml | Layout | 预警卡片布局 |
| activity_alert_detail.xml | Layout | 预警详情布局 |
| activity_threshold_settings.xml | Layout | 阈值编辑布局 |
| item_threshold_edit.xml | Layout | 阈值编辑卡片布局 |
| ic_alert_level_info.xml | Drawable | INFO 级别图标 |
| ic_alert_level_critical.xml | Drawable | CRITICAL 级别图标 |

## 四、修改文件

| 文件 | 修改内容 |
|------|----------|
| GreenhouseApiService.java | 新增阈值 CRUD 接口 + 预警列表 level 筛选参数 |
| GreenhouseRepository.java | 新增 markAlertRead / getThresholds / setThreshold / deleteThreshold 方法 |
| MainActivity.java | AlertFragment 替换原有占位 Fragment |
| AndroidManifest.xml | 注册 AlertDetailActivity + ThresholdSettingsActivity |

## 五、构建结果

- **BUILD SUCCESSFUL** ✅
- 修复问题：`fragment_alert.xml` 中 `android:chipBackgroundColor` → `app:chipBackgroundColor`（Material Chip 属性需使用 app 前缀）

## 六、变更记录

### Step 22 (2026-07-13): 预警合并到看板
- **变更内容**: AlertFragment 从独立Tab变为看板内嵌入口访问
- **MainActivity.java**: 移除 `nav_alert` Tab分支，不再作为独立底部导航Tab
- **fragment_dashboard.xml**: 新增"预警中心"入口卡片
- **DashboardFragment.java**: 点击入口卡片 → 跳转 AlertFragment（addToBackStack）
- **AlertFragment.java**: 代码不变，功能完整保留
- 变更原因：为设备控制模块(F05)腾出底部导航位置

## 六、API 对接

| API 端点 | 方法 | 用途 |
|----------|------|------|
| GET /api/v1/alerts?level={level}&page={page}&size={size} | 预警列表 | 分页+级别筛选 |
| PUT /api/v1/alerts/{id}/read | 标记已读 | 单条预警已读 |
| GET /api/v1/alerts/thresholds?greenhouseId={id} | 获取阈值 | 当前大棚自定义阈值 |
| POST /api/v1/alerts/thresholds | 设置阈值 | 创建或更新阈值 |
| DELETE /api/v1/alerts/thresholds/{id} | 删除阈值 | 删除自定义阈值 |

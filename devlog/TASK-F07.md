---
id: TASK-F07
title: 作物长势评估开发
type: task
module: F7 APP长势
tags: [android, growth, assessment, camera, crop-cycle]
status: completed
created: 2026-07-13
completed: 2026-07-13
author: AI助手
devlog_step: 27
dependencies: [TASK-C01, TASK-C08, TASK-C22]
---

# TASK-F07: 作物长势评估开发

> APP端作物长势评估模块，展示截帧图片、生长阶段识别、长势特征评估和种植周期信息。

---

## 任务概述

开发Android端作物长势评估功能，用户可从看板页进入长势评估页面，查看摄像头截帧图片信息、AI识别的生长阶段、株高/叶面积/叶色等长势特征指标，以及当前种植周期信息。

## 功能列表

| 序号 | 功能 | 说明 | 状态 |
|------|------|------|------|
| 1 | 长势评估查看 | 最新评估结果：生长阶段+健康评分+株高/叶面积/叶色 | ✅ |
| 2 | 截帧图片信息 | 截帧数量/最新截帧时间/分辨率/文件大小 | ✅ |
| 3 | 种植周期信息 | 作物名称/种植日期/已种植天数/当前阶段/状态 | ✅ |
| 4 | 看板入口 | 看板页三列入口卡片（长势评估+历史数据+预警中心） | ✅ |

## 功能裁剪

以下功能标记为后续版本：
- **长势历史对比** → Phase 5 完善项
- **生长时间线** → Phase 5 完善项
- **多模态融合评分** → F08 多模态健康评分

## 入口方式

- 看板页三列入口卡片，点击跳转 GrowthActivity
- 传递 greenhouse_id 参数

## 关键修改

| 文件 | 修改类型 | 变更说明 |
|------|----------|----------|
| `app/.../data/model/GrowthAssessment.java` | 新增 | 长势评估结果模型，含健康评分等级/颜色辅助方法 |
| `app/.../data/model/GrowthImage.java` | 新增 | 截帧图片模型，含文件大小格式化 |
| `app/.../viewmodel/GrowthViewModel.java` | 新增 | 长势评估/截帧图片/种植周期加载+分页 |
| `app/.../ui/growth/GrowthActivity.java` | 新增 | 长势评估展示页面（ScrollView+CardView布局） |
| `app/src/main/res/layout/activity_growth.xml` | 新增 | 三组卡片布局（评估+截帧+种植周期） |
| `app/src/main/res/drawable/ic_growth.xml` | 新增 | 叶子形状长势评估图标 |
| `app/.../data/api/GreenhouseApiService.java` | 修改 | 新增3个 growth API 端点 |
| `app/.../data/repository/GreenhouseRepository.java` | 修改 | 新增4个仓库方法（含 getCropCycles） |
| `app/.../ui/dashboard/DashboardFragment.java` | 修改 | 新增长势评估入口点击事件 |
| `app/src/main/res/layout/fragment_dashboard.xml` | 修改 | 入口卡片从两列改为三列布局 |
| `app/src/main/AndroidManifest.xml` | 修改 | 注册 GrowthActivity |

## 完成结果

**BUILD SUCCESSFUL** — 长势评估模块完整：最新评估结果展示（生长阶段+健康评分+3项长势指标）、截帧图片信息统计、种植周期信息展示。看板入口从两列扩展为三列（长势评估/历史数据/预警中心）。API层3个新端点 + Repository 4个新方法（含C22作物周期）。

## 阻塞记录

无阻塞

---
id: TASK-F08
title: 多模态健康评分开发
type: task
module: F8 APP健康评分
tags: [android, health, multimodal, fusion, score, chart]
status: completed
created: 2026-07-13
completed: 2026-07-13
author: AI助手
devlog_step: 28
dependencies: [TASK-C01, TASK-C05, TASK-C08, TASK-C15, TASK-C22]
---

# TASK-F08: 多模态健康评分开发

> APP端多模态健康评分详情页，展示综合健康评分（环境+视觉融合）、子维度评分明细、历史趋势图、环境/长势分析、健康评估报告。

---

## 任务概述

开发Android端多模态健康评分详情页，用户从看板顶部健康评分卡片点击进入，查看环境数据与视觉长势数据融合后的综合健康评分（0-100分）、各子维度评分明细、历史趋势图、详细分析和改善建议。

## 功能列表

| 序号 | 功能 | 说明 | 来源 | 状态 |
|------|------|------|------|------|
| 1 | 综合健康评分展示 | 0-100分大数字+等级颜色编码+评估时间 | F08 US-001 | ✅ |
| 2 | 子维度评分明细 | 环境健康(40%)+长势健康(40%)+天气风险 | F08 US-002 | ✅ |
| 3 | 评分历史趋势 | MPAndroidChart 3线图，7天/30天切换 | F08 US-003 | ✅ |
| 4 | 环境健康度分析 | 温度/湿度/CO₂/土壤 评分+评论 | F08 US-004 | ✅ |
| 5 | 长势健康度分析 | 叶片健康/生长速度/病害风险 评分+评论 | F08 US-005 | ✅ |
| 6 | 健康评估报告 | 天气影响+改善建议（页面展示） | F08 US-006 | ✅ |
| 7 | 多模态融合评分 | 环境+视觉数据融合（整合自F07推迟的F-004-05） | F07→F08 | ✅ |

## 功能裁剪

以下功能标记为后续版本：
- **PDF 报告导出** → Phase 5 完善项

## 入口方式

- 看板顶部健康评分摘要卡片 → 可点击 → 跳转 HealthActivity
- 传递 greenhouse_id 参数

## 关键修改

| 文件 | 修改类型 | 变更说明 |
|------|----------|----------|
| `app/.../data/model/HealthScoreData.java` | 重写 | 扩展为完整模型，含 AnalysisDetail 内部类（envDetail/visualDetail/weatherImpact） |
| `app/.../viewmodel/HealthViewModel.java` | 新增 | 评分/详情/历史/分页/时间范围切换 |
| `app/.../ui/health/HealthActivity.java` | 新增 | 6大功能区域展示页面（ScrollView+CardView+LineChart） |
| `app/src/main/res/layout/activity_health.xml` | 新增 | 完整布局（评分大卡+子维度三列+趋势图+环境分析+长势分析+报告） |
| `app/src/main/res/drawable/bg_score_badge.xml` | 新增 | 评分徽章背景 |
| `app/src/main/res/drawable/bg_dim_score.xml` | 新增 | 子维度圆形背景 |
| `app/src/main/res/drawable/bg_level_tag.xml` | 重写 | 等级标签背景 |
| `app/.../data/api/GreenhouseApiService.java` | 修改 | 新增 health/detail/{id} 端点 |
| `app/.../data/repository/GreenhouseRepository.java` | 修改 | 新增 getHealthDetail + getHealthHistory 方法 |
| `app/.../ui/dashboard/DashboardFragment.java` | 修改 | 健康评分卡片 → 可点击跳转 HealthActivity |
| `app/src/main/res/layout/fragment_dashboard.xml` | 修改 | 健康评分 CardView 增加 id/clickable |
| `app/src/main/AndroidManifest.xml` | 修改 | 注册 HealthActivity |

## 完成结果

**BUILD SUCCESSFUL** — 多模态健康评分详情页完整：综合评分大数字展示、三列子维度评分、MPAndroidChart 历史趋势图（3线）、环境/长势子项分析（评分徽章+评论）、评估报告（天气+建议）。评分颜色自动编码（绿/蓝/黄/红）。看板健康评分卡片改为可点击入口。API 新增详情端点，Repository 新增历史和详情方法。

## 阻塞记录

无阻塞

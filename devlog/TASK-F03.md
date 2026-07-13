---
id: TASK-F03
title: 病虫害拍照诊断开发
module: F3 APP诊断
tags: [app, android, diagnosis, camera, ai]
status: completed
created: 2026-07-13
completed: 2026-07-13
author: AI助手
dependencies: [TASK-C08, TASK-F01]
related_docs: [PRD-003, API-007]
---

# TASK-F03：病虫害拍照诊断开发

## 一、任务概述

在 Android APP 端实现病虫害拍照诊断功能，包括拍照/相册选取、图片压缩上传、AI 识别结果展示（置信度+防治方案）、诊断历史记录、低置信度引导求助专家。

## 二、实现内容

### 2.1 诊断主页（DiagnosisFragment）

- 拍照按钮 + 相册按钮（AndroidX Activity Result API）
- RecyclerView 诊断历史列表（DiagnosisHistoryAdapter）
- 图片 Uri→File 压缩（1024px 宽/JPEG 80%，子线程 IO）
- 空状态提示 + 加载指示器

### 2.2 诊断结果（DiagnosisResultActivity）

- 图片展示（Glide）
- 病害名称 + 置信度（进度条+三级颜色：绿≥80%/黄70-80%/红<70%）
- 防治方案文本展示
- 低置信度求助专家按钮（预埋跳转，F10 对接）

### 2.3 业务逻辑（DiagnosisViewModel）

- 图片上传诊断
- 诊断历史分页加载
- 低置信度判断（needExpert < 70%）
- 符合规范：不持有 Context/ContentResolver

### 2.4 数据层

- GreenhouseApiService：@Multipart diagnose + getDiagnosisHistory
- GreenhouseRepository：diagnose() + getDiagnosisHistory()

## 三、新增文件（15个）

| 文件 | 类型 | 说明 |
|------|------|------|
| DiagnosisResponse.java | Model | 诊断结果（含置信度颜色等级） |
| DiagnosisHistoryItem.java | Model | 历史列表项（含 fromResponse 转换） |
| DiagnosisViewModel.java | ViewModel | 诊断业务逻辑 |
| DiagnosisHistoryAdapter.java | Adapter | 历史列表适配器 |
| DiagnosisFragment.java | Fragment | 诊断主页 |
| DiagnosisResultActivity.java | Activity | 诊断结果页 |
| fragment_diagnosis.xml | Layout | 诊断主页布局 |
| activity_diagnosis_result.xml | Layout | 结果页布局 |
| item_diagnosis_history.xml | Layout | 历史卡片布局 |
| ic_camera.xml | Drawable | 相机图标 |
| ic_gallery.xml | Drawable | 相册图标 |
| ic_alert_level_critical.xml | Drawable | 严重警告图标 |
| bg_thumbnail_placeholder.xml | Drawable | 缩略图占位 |
| bg_confidence_dot.xml | Drawable | 置信度圆点 |
| file_paths.xml | XML | FileProvider 路径配置 |

## 四、修改文件（4个）

| 文件 | 修改内容 |
|------|----------|
| GreenhouseApiService.java | 新增 @Multipart diagnose + getDiagnosisHistory |
| GreenhouseRepository.java | 新增 diagnose() + getDiagnosisHistory() |
| MainActivity.java | 诊断 Tab 切换到 DiagnosisFragment |
| AndroidManifest.xml | 注册 DiagnosisResultActivity + FileProvider |
| colors.xml | 新增 confidence_high/medium/low 颜色 |

## 五、构建结果

- **BUILD SUCCESSFUL** ✅

## 六、变更记录

### Step 23 (2026-07-13): 诊断合并到AI助手
- **MainActivity.java**: DiagnosisFragment 不再作为独立Tab，改为 AiAssistantFragment 内 ViewPager2 子页
- **DiagnosisFragment.java**: 代码不变，功能完整保留
- 变更原因：与问答合并为"AI助手"统一入口

| 功能 | 说明 |
|------|------|
| F-003-06 分享导出 | 不开发，标记跳过 |

## 七、API 对接

| API 端点 | 方法 | 用途 |
|----------|------|------|
| POST /api/v1/diagnosis/recognize | 图片诊断 | @Multipart 上传图片+greenhouseId |
| GET /api/v1/diagnosis/records | 诊断历史 | 分页查询（greenhouseId/page/size） |

---
id: TASK-C08
title: 图片诊断模块开发
module: C8 图片诊断
type: task
priority: high
status: completed
created: 2026-07-13
started: 2026-07-13
completed: 2026-07-13
assignee: AI助手
related_docs:
  - DEVLOG-步骤13
tags: [c8-图片诊断]
---

# 图片诊断模块开发

---

## 任务描述

### 背景

C8 图片诊断是智慧大棚AIoT系统的核心后端模块，属于Phase 2/3开发计划。

### 目标

完成C8 图片诊断的全部后端代码开发，包括实体类、Repository、Service、Controller和DTO。

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
| `ai/DiseaseRecognitionProvider.java` | 新建 | 策略接口，RecognitionResult record（diseaseName/confidence/treatment/engineName/needExpertConsultation）
| `ai/baidu/BaiduRecognitionProvider.java` | 新建 | 百度AI植物识别：OAuth Token缓存、Base64上传、解析百科描述
| `ai/resnet/ResNetRecognitionProvider.java` | 新建 | Phase 3本地模型占位
| `entity/DiagnosticRecord.java` | 新建 | DB第12号表，userId/greenhouseId/imagePath/diseaseName/confidence/treatment/recognitionEngine
| `repository/DiagnosticRecordRepository.java` | 新建 | 按用户分页查询历史
| `module/diagnosis/service/DiagnosisService.java` | 新建 | 核心流程：保存图片→AI识别→保存记录→返回结果
| `module/diagnosis/controller/DiagnosisController.java` | 新建 | 2个端点（POST /recognize、GET /records）
| `module/diagnosis/dto/DiagnosisResponse.java` | 新建 | needExpert字段（confidence<0.70）
| `module/file/service/FileService.java` | 新建 | saveDiagnosisImage()，图片类型校验，日期分目录
- 修改：application-dev.yml（multipart 10MB）

---

## 完成结果

AI病虫害诊断链路完整，策略模式支持百度API→ResNet无缝切换

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-13
- 关联DEVLOG：步骤13

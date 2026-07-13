---
id: TASK-C10
title: 语音识别模块开发
module: C10 语音识别
type: task
priority: high
status: completed
created: 2026-07-13
started: 2026-07-13
completed: 2026-07-13
assignee: AI助手
related_docs:
  - DEVLOG-步骤14
tags: [c10-语音识别]
---

# 语音识别模块开发

---

## 任务描述

### 背景

C10 语音识别是智慧大棚AIoT系统的核心后端模块，属于Phase 2/3开发计划。

### 目标

完成C10 语音识别的全部后端代码开发，包括实体类、Repository、Service、Controller和DTO。

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
| `ai/SpeechRecognitionProvider.java` | 新建 | 策略接口，SpeechRecognitionResult record（text/rawDialectText/confidence/dialect/engineName/durationMs）
| `ai/xunfei/XunfeiSpeechProvider.java` | 新建 | 讯飞ASR：HMAC-SHA256签名鉴权、河北话方言(accent=hebei)、Base64音频上传
| `ai/whisper/WhisperSpeechProvider.java` | 新建 | Phase 3本地Whisper占位
| `module/qa/service/VoiceQaService.java` | 新建 | 语音问答：保存音频→ASR识别→RAG生成→保存VOICE记录
| `module/qa/controller/QaController.java` | 新建 | POST /api/v1/qa/ask/voice端点
- 修改：FileService（加saveAudioFile()）

---

## 完成结果

语音问答链路完整，讯飞ASR→RAG→结果，ASR失败引导文字输入

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-13
- 关联DEVLOG：步骤14

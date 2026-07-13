---
id: TASK-C09
title: RAG问答模块开发
module: C9 RAG问答
type: task
priority: high
status: completed
created: 2026-07-13
started: 2026-07-13
completed: 2026-07-13
assignee: AI助手
related_docs:
  - DEVLOG-步骤14
tags: [c9-rag问答]
---

# RAG问答模块开发

---

## 任务描述

### 背景

C9 RAG问答是智慧大棚AIoT系统的核心后端模块，属于Phase 2/3开发计划。

### 目标

完成C9 RAG问答的全部后端代码开发，包括实体类、Repository、Service、Controller和DTO。

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
| `entity/QaRecord.java` | 新建 | DB第13号表，TEXT/VOICE输入类型、引用来源JSON
| `repository/QaRecordRepository.java` | 新建 | 按用户分页查询历史
| `module/qa/service/EmbeddingService.java` | 新建 | SiliconFlow bge-m3向量化（1024维），支持单条和批量
| `module/qa/service/ChromaRetrievalService.java` | 新建 | Chroma REST API检索top-5，不可用时返回空不阻塞
| `module/qa/service/RagQaService.java` | 新建 | RAG核心：向量化→检索→Prompt组装→DeepSeek生成→保存。降级策略：Chroma不可用→通用知识；提供generateAnswerOnly()供VoiceQaService复用
| `module/qa/controller/QaController.java` | 新建 | 2个端点（POST /ask、GET /records）
- 3个DTO：QaAskRequest、QaResponse（含SourceInfo）、QaHistoryItem
- 修改：PageResult（加of(Page)工厂方法）、application-dev.yml（chroma.collection、deepseek独立配置）

---

## 完成结果

RAG问答完整链路，降级策略完备，知识库无内容时自动标注'基于通用知识'

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-13
- 关联DEVLOG：步骤14

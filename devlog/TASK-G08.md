---
id: TASK-G08
title: 方言语料管理开发
module: G8 Web语料管理
type: task
priority: high
status: completed
created: 2026-07-15
started: 2026-07-15
completed: 2026-07-15
assignee: AI助手
related_docs:
  - DEVLOG-步骤42
  - TASK-G04（知识库备注延后说明）
tags: [web, vue3, corpus, dialect, audio, admin]
---

# TASK-G08: 方言语料管理开发

## 需求描述

Web 管理端方言语料管理。管理员可上传方言音频文件（含标注文本）、按方言类型筛选和关键词搜索、分页浏览、删除语料（含音频文件清理）。

本模块在 G04 知识库开发时被延后（记录于 TASK-G04.md 备注），PRD 中对应 US-006 方言语料管理。

## 设计决策

### 功能定位

| 决策项 | 说明 |
|------|------|
| 定位 | **语料文件存储管理入口**，而非完整标注平台。专业标注工具（Praat/ELAN/PraatScript）是 NLP 领域专用软件，Web 端不应重复造轮子 |
| 音频存储 | 按日期分目录 `uploads/corpus/yyyy/MM/dd/uuid.ext`，与 FileService 设计一致 |
| 方言类型 | 支持 6 种：河北话/山东话/东北话/河南话/四川话/广东话 |
| 来源区分 | MANUAL（手动上传）/ QA_COLLECT（语音问答采集），为后续从问答记录自动采集语料预留接口 |
| 删除策略 | 同步清理音频文件（`Files.deleteIfExists`），与 G04 知识库删除策略一致 |

## 技术方案

### 后端
- 新建 `DialectCorpus` JPA 实体 + `DialectCorpusRepository`
- 新建 `AdminCorpusService`：上传校验→保存文件→创建记录，分页+筛选查询，删除清理
- 新建 `AdminCorpusController`：4 个 REST 端点
- 零依赖新增，复用已有 Multipart 配置和 SecurityConfig `hasRole('ADMIN')`

### 前端
- Vue 3 Composition API + Element Plus
- 表格+筛选+分页+上传对话框（拖拽）+ 删除确认

## 后端 API

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/v1/admin/corpus | 语料列表（?dialect=&keyword=&page=&size=） |
| GET | /api/v1/admin/corpus/dialects | 方言类型列表（去重） |
| POST | /api/v1/admin/corpus | 上传语料（multipart: audio + dialect + annotationText + ...） |
| DELETE | /api/v1/admin/corpus/{id} | 删除语料（含音频文件） |

## 关键修改

### 后端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `entity/DialectCorpus.java` | 90 | 方言语料 JPA 实体 |
| `repository/DialectCorpusRepository.java` | 30 | 语料数据访问层 |
| `module/admin/dto/DialectCorpusResponse.java` | 35 | 语料响应 DTO |
| `module/admin/service/AdminCorpusService.java` | 140 | 语料管理服务 |
| `module/admin/controller/AdminCorpusController.java` | 75 | 语料管理 API |

### 前端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `web/src/api/corpus.js` | 25 | 语料 API 封装 |
| `web/src/views/corpus/CorpusPage.vue` | 330 | 方言语料管理页面 |

### 前端修改文件

| 文件 | 变更 |
|------|------|
| `web/src/router/index.js` | 注册 /corpus 路由 |
| `web/src/layouts/MainLayout.vue` | 启用"语料管理"菜单项（移除 disabled） |

### 跨模块影响

| 受影响模块 | 变更 |
|------|------|
| TASK-G04（知识库管理） | 在 G04 日志中追加说明：US-006 方言语料管理已在 G08 完成 |

## 数据库设计

```sql
CREATE TABLE dialect_corpus (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    dialect VARCHAR(30) NOT NULL COMMENT '方言类型',
    audio_path VARCHAR(500) NOT NULL COMMENT '音频文件路径',
    audio_filename VARCHAR(255) NOT NULL COMMENT '原始文件名',
    audio_size BIGINT COMMENT '文件大小(字节)',
    annotation_text TEXT COMMENT '标注文本(标准普通话)',
    dialect_text TEXT COMMENT '方言原文(可选)',
    source VARCHAR(20) NOT NULL DEFAULT 'MANUAL' COMMENT '来源',
    remark VARCHAR(500) COMMENT '备注',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_dialect (dialect),
    INDEX idx_created_at (created_at)
);
```

## 构建结果

### 后端
```
mvn compile -pl common,backend → BUILD SUCCESS
170+ Java 文件，零错误零警告
```

### 前端
```
vite build → ✓ built in 992ms
新增产出物：
- dist/assets/CorpusPage-DoygCu8x.js (8.52 kB / gzip: 3.37 kB)
- dist/assets/CorpusPage-CYDpSRLj.css (0.96 kB / gzip: 0.37 kB)
```

## 已知限制 & 后续扩展

| 项目 | 说明 |
|------|------|
| 音频播放 | 当前不支持页面内播放。后续可加 HTML5 `<audio>` 标签在线试听 |
| 批量上传 | 当前单文件上传，后续可支持批量 |
| 自动采集 | QA_COLLECT 来源已预留，但自动采集逻辑尚未实现 |
| 语料导出 | 不支持批量导出语料包，后续可加 |

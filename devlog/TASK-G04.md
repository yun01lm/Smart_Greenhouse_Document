---
id: TASK-G04
title: 知识库管理界面开发
module: G4 Web知识库
type: task
priority: high
status: completed
created: 2026-07-15
started: 2026-07-15
completed: 2026-07-15
assignee: AI助手
related_docs:
  - DEVLOG-步骤38
tags: [web, vue3, knowledge, chroma, rag, element-plus]
---

# TASK-G04: 知识库管理界面开发

## 需求描述

Web 管理端知识库管理界面。管理员可以上传农业专业知识文档（.md/.txt），系统自动进行文档切片、向量化处理并存入 Chroma 向量数据库。同时提供问答测试功能，验证 RAG 问答效果。

## 重要说明

- **方言语料管理（US-006）延后到 TASK-G08**：PRD 中知识库与语料管理放在一起，但 INDEX 规划中两者分开。语料管理涉及音频文件处理、标注格式等独立功能，决定留给 G08。
- **PDF/DOCX 暂不支持**：当前仅支持 .md/.txt 格式，PDF/DOCX 在后续版本中添加（需引入 PDF 解析库）。

## 技术方案

### 后端
- 新建 `knowledge` 模块（controller + service + dto），遵循 Controller→Service→Repository 分层
- 路径前缀 `/api/v1/knowledge`，SecurityConfig 中 `hasRole('ADMIN')`
- **文档处理管道**：上传 → 保存文件 → 文本提取 → 切片(500-1000字/200字重叠) → 批量 Embedding → Chroma 写入 → MySQL 状态更新
- **删除同步清理**：删除文档时同步清理 Chroma 向量 + 本地文件 + MySQL 记录

### 前端
- Vue 3 Composition API + Element Plus
- Tab 切换：文档管理 + 问答测试
- 文档管理：分类筛选、关键词搜索、分页、上传对话框（拖拽）、向量化状态展示、删除确认
- 问答测试：输入问题 → RAG 检索 → 展示 AI 回答 + 检索片段

## 后端 API

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| GET | /api/v1/knowledge/documents | 文档列表（?category=&keyword=&page=&size=） | ADMIN |
| GET | /api/v1/knowledge/categories | 分类列表 | ADMIN |
| POST | /api/v1/knowledge/documents | 上传文档（multipart/form-data） | ADMIN |
| POST | /api/v1/knowledge/index | 触发向量化（?documentId=） | ADMIN |
| DELETE | /api/v1/knowledge/documents/{id} | 删除文档 | ADMIN |
| POST | /api/v1/knowledge/test | 问答测试 | ADMIN |

## 关键修改

### 后端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `module/knowledge/dto/KnowledgeDocumentResponse.java` | 59 | 文档响应 DTO（含文件大小格式化） |
| `module/knowledge/dto/KnowledgeTestRequest.java` | 20 | 问答测试请求 DTO |
| `module/knowledge/dto/KnowledgeTestResponse.java` | 37 | 问答测试响应 DTO（含检索片段） |
| `module/knowledge/service/KnowledgeService.java` | 375 | 核心业务逻辑 + 文档处理管道 |
| `module/knowledge/controller/KnowledgeController.java` | 93 | REST API 控制器 |

### 后端修改文件

| 文件 | 变更 |
|------|------|
| `repository/KnowledgeDocumentRepository.java` | 新增分页查询方法（findByCategory/findByTitleContaining/findDistinctCategories） |
| `config/SecurityConfig.java` | 新增 `/api/v1/knowledge/**` 的 `hasRole('ADMIN')` |
| `module/qa/service/RagQaService.java` | AnswerResult record 和 generateAnswerOnly() 方法改为 public（供 KnowledgeService 跨包调用） |

### 前端新建文件

| 文件 | 行数 | 说明 |
|------|------|------|
| `web/src/api/knowledge.js` | 33 | Knowledge API 封装（6 个接口） |
| `web/src/views/knowledge/KnowledgePage.vue` | 373 | 知识库管理页面（文档管理 + 问答测试） |

### 前端修改文件

| 文件 | 变更 |
|------|------|
| `web/src/router/index.js` | 注册 /knowledge 路由 |
| `web/src/layouts/MainLayout.vue` | 启用"知识库"菜单项（移除 disabled） |

## 文档处理管道流程

```
用户上传 .md/.txt
    ↓
保存文件到 uploads/knowledge/{date}/{uuid}.ext
    ↓
创建 MySQL 记录（vectorIndexed=false）
    ↓
读取文件内容（UTF-8）
    ↓
文本切片：按空行分段，每块800字，相邻块200字重叠
    ↓
批量 Embedding：调用 SiliconFlow bge-m3 API
    ↓
写入 Chroma：POST /collections/greenhouse_knowledge/add
    ↓
更新 MySQL：chunkCount + vectorIndexed=true + indexedAt
```

## 删除流程

```
删除请求
    ↓
Chroma 删除：POST /collections/greenhouse_knowledge/delete（按 doc_id 过滤）
    ↓
本地文件删除：Files.deleteIfExists()
    ↓
MySQL 记录删除：documentRepository.delete()
```

## 构建结果

### 后端
```
mvn compile -pl common,backend → BUILD SUCCESS
编译 167 个 Java 文件，零错误
```

### 前端
```
vite build → ✓ built in 1.12s
新增产出物：
- dist/assets/KnowledgePage-DjmCi26F.js (9.45 kB / gzip: 3.83 kB)
```

## 备注

- **方言语料延后**：PRD US-006 方言语料管理留给 TASK-G08 实现 ✅ **已于 2026-07-15 在 G08 完成**。详见 [TASK-G08.md](./TASK-G08.md)。G08 实现了完整的语料管理（DialectCorpus 实体 + 音频上传 + 标注文本 + 方言筛选 + 分页 + 删除），支持 6 种方言类型。
- **PDF/DOCX 延后**：当前上传时返回友好提示"暂不支持，请使用 .md 或 .txt 格式"
- **异步向量化**：当前在上传请求中同步执行向量化，大文件可能超时。后续可改为异步队列处理

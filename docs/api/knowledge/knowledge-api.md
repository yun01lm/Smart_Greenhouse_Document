---
id: API-010c
title: 知识库管理 API
type: api
module: knowledge
tags: [知识库, 文档管理, 向量化, RAG, Chroma]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

知识库管理模块供管理员在 Web 管理端维护农业知识库。支持文档上传、删除、向量化入库和问答测试。知识库文档通过 Spring AI 自动切片（每片约500字），经 SiliconFlow bge-m3 模型生成1024维向量后存储到 Chroma 向量数据库，为 RAG 问答提供检索基础。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/knowledge/documents` | 是 | ADMIN | 文档列表（分页/按分类筛选） |
| POST | `/api/v1/knowledge/documents` | 是 | ADMIN | 上传文档 |
| DELETE | `/api/v1/knowledge/documents/{id}` | 是 | ADMIN | 删除文档 |
| POST | `/api/v1/knowledge/index` | 是 | ADMIN | 触发向量化 |
| POST | `/api/v1/knowledge/test` | 是 | ADMIN | 问答测试 |

## 请求/响应示例

### 1. 文档列表

**请求：**
```
GET /api/v1/knowledge/documents?category=病虫害防治&cropType=番茄&page=1&size=10
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "list": [
      {
        "id": 1,
        "title": "番茄晚疫病综合防治技术",
        "category": "病虫害防治",
        "cropType": "番茄",
        "filePath": "/uploads/knowledge/tomato_late_blight.md",
        "chunkCount": 12,
        "vectorIndexed": true,
        "createdAt": "2026-07-01T08:00:00"
      },
      {
        "id": 2,
        "title": "大棚番茄水肥管理要点",
        "category": "栽培技术",
        "cropType": "番茄",
        "filePath": "/uploads/knowledge/tomato_fertilizer.md",
        "chunkCount": 8,
        "vectorIndexed": true,
        "createdAt": "2026-07-01T08:30:00"
      }
    ],
    "total": 25,
    "page": 1,
    "size": 10
  }
}
```

### 2. 上传文档

**请求：**
```
POST /api/v1/knowledge/documents
Authorization: Bearer <token>
Content-Type: multipart/form-data

file: (binary file, .md/.txt/.docx, max 20MB)
title: "黄瓜霜霉病防治手册"
category: "病虫害防治"
cropType: "黄瓜"
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 26,
    "title": "黄瓜霜霉病防治手册",
    "category": "病虫害防治",
    "cropType": "黄瓜",
    "filePath": "/uploads/knowledge/cucumber_downy_mildew.md",
    "chunkCount": 0,
    "vectorIndexed": false,
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

### 3. 触发向量化

**请求：**
```json
POST /api/v1/knowledge/index
Authorization: Bearer <token>
Content-Type: application/json

{
  "documentId": 26
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "documentId": 26,
    "title": "黄瓜霜霉病防治手册",
    "chunkCount": 10,
    "vectorIndexed": true,
    "indexedAt": "2026-07-11T10:31:00"
  }
}
```

### 4. 问答测试

**请求：**
```json
POST /api/v1/knowledge/test
Authorization: Bearer <token>
Content-Type: application/json

{
  "question": "黄瓜霜霉病怎么防治？",
  "topK": 5
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "question": "黄瓜霜霉病怎么防治？",
    "answer": "黄瓜霜霉病是由古巴假霜霉菌引起的真菌性病害...",
    "retrievedChunks": [
      {
        "documentId": 26,
        "documentTitle": "黄瓜霜霉病防治手册",
        "chunkIndex": 3,
        "content": "化学防治：发病初期可使用霜脲·锰锌800倍液...",
        "score": 0.92
      }
    ],
    "responseTime": 3200
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 3001 | 403 | 无访问权限（非管理员） |
| 1002 | 404 | 文档不存在 |
| 5003 | 500 | AI服务暂时不可用 |
| 5004 | 500 | 向量化服务异常 |
| 8001 | 500 | 文件上传失败 |
| 8002 | 400 | 文件大小超过限制 |
| 8003 | 400 | 不支持的文件类型 |
| 1001 | 400 | 参数错误 |

---

## 变更记录（追加，2026-08-06 · 步骤76）

### `POST /api/v1/knowledge/index`（触发向量化）
- 新增能力：**重新向量化幂等** —— 已向量化文档再次触发索引时，先按 `doc_id` 清理 Chroma 旧向量再写入，不会产生重复向量
- 新增能力：**编码兼容** —— 支持 UTF-8 与 GBK/ANSI 编码的 `.md`/`.txt` 文件（Windows 记事本默认编码可直接上传向量化），不再报 `MalformedInputException`
- 实现说明：Chroma 写入分批执行（每批 200 条），大文档（万级文本块）不再单次提交巨量请求体

### `POST /api/v1/knowledge/documents`（上传文档）
- 实现说明：上传成功后自动触发向量化（当前 provider=siliconflow 真实向量化，模型 BAAI/bge-m3，1024 维）
- 注意：13MB/2735 块文档真实向量化约 7.5 分钟，页面需等待返回（当前为同步接口）

---

## 变更记录（追加，2026-08-06 · 步骤77：上传异步化）

### `POST /api/v1/knowledge/documents`（上传文档）
- **上传与向量化解耦**：接口保存文件并创建记录后立即返回（响应消息"文档上传成功，向量化处理中"），向量化由后台单线程线程池执行（事务提交后触发）
- 前端可稍后刷新列表查看状态；未向量化文档可点击"重新向量化"手动重试
- 向量化失败不再影响上传：文档保持"待向量化"状态（vector_indexed=false）


---

## 变更记录（追加，2026-08-06 · 步骤78：文档标记信息编辑）

### `PUT /api/v1/knowledge/documents/{id}`（更新文档标记信息）
- 新增接口：更新文档编号/标题/分类/简介等标记信息（仅元数据，不影响文件内容与向量）
- 请求体：
```json
{
  "docNo": "TEST-001",
  "title": "文档标题",
  "category": "栽培技术",
  "description": "文档简介/内容摘要"
}
```
- 字段说明：
  - `docNo`：文档编号（业务编号，唯一，≤64 字符；为空自动回退默认 `DOC-xxxx`）
  - `title`：标题（必填非空，≤200 字符）
  - `category`：分类（≤100 字符，空字符串表示清除分类）
  - `description`：简介（≤2000 字符，空字符串表示清除简介）
  - 字段传 `null` 表示不修改
- 同步行为：若文档已向量化，编辑后自动更新 Chroma 中全部切片元数据（标题/分类/简介），保证 AI 问答引用来源一致；Chroma 同步失败不阻塞保存（后端记录告警日志）
- 响应：更新后的文档信息（含 docNo/description）
- 错误码：文档编号重复 → 1001（"文档编号已被占用"）；标题为空 → 1001；文档不存在 → 1001

### 元数据变更说明
- `knowledge_documents` 表新增列：`doc_no`（varchar(64)，唯一）、`description`（TEXT）
- 历史文档已回填默认编号（`DOC-` + 4 位补零 id，如 DOC-0001）
- 上传/种子文档自动生成默认编号，用户可在编辑中修改
---

## 变更记录（追加，2026-08-07 · 步骤79：分类管理 + ID 复用）

### 分类管理（方案B）
#### `GET /api/v1/knowledge/categories/managed`（分类管理列表）
- 返回全部正式分类：`id / name / description / docCount`（该分类下文档数）
#### `POST /api/v1/knowledge/categories/managed`（新建分类）
- 请求体：`{"name":"育苗技术","description":"..."}`；`name` 唯一（重复 → 400 参数错误）
#### `PUT /api/v1/knowledge/categories/managed/{id}`（编辑分类）
- 请求体：`{"name":"新名称","description":"..."}`
- **重命名级联**：自动更新该分类下全部文档（knowledge_documents.category）与 Chroma 全部切片元数据，保证 AI 问答检索与来源标注一致
#### `DELETE /api/v1/knowledge/categories/managed/{id}`（删除分类）
- 分类下有文档（docCount>0）时拒绝删除（400"该分类下仍有文档，无法删除"），防止文档分类悬挂
- 兜底机制：上传/编辑文档时使用未登记的新分类会自动登记（ensureCategoryRegistered），不阻塞上传

### ID 复用（方案A）
- **背景**：knowledge_documents.id 原为自增主键，删除后不回落、ID 出现空洞；需求为新增优先复用已删除 ID
- **实现**：
  - 删除文档：ID 写入 `knowledge_document_id_recycle(recycled_id)` 回收池
  - 新增文档：`allocateDocumentId()` 先取回收池最小 ID（`SELECT ... FOR UPDATE LIMIT 1`），池空则读取 `knowledge_document_id_seq.next_id` 并 +1
  - 并发安全：分配在事务内用行锁（FOR UPDATE），同一时刻不会分配重复 ID
- **行为变化**：文档 ID 不再严格按时间单调递增；删除后新建可能复用旧 ID（如删除 16 后再上传仍为 16）
- **Chroma 防御**：复用 ID 前先 `deleteFromChroma` 清理可能残留的旧向量，再写入新文档向量
- 种子文档/上传文档 ID 均走统一分配器，行为一致

### 数据库变更
- 新增表 `knowledge_categories`（id / name 唯一 / description / doc_count / created_at / updated_at）
- 新增表 `knowledge_document_id_recycle`（recycled_id 主键 / created_at）
- 新增表 `knowledge_document_id_seq`（next_id，当前 16）
- `knowledge_documents.id` 由自增改为服务层显式分配（实体去掉 @GeneratedValue）

---
id: TASK-F04
title: AI智能问答模块开发
type: task
module: F4 APP问答
tags: [android, qa, voice, tts, rag]
status: completed
created: 2026-07-13
completed: 2026-07-13
author: AI助手
devlog_step: 21
dependencies: [TASK-C01, TASK-C09, TASK-C10]
---

# TASK-F04: AI智能问答模块开发

> APP端AI智能问答模块，支持文字提问和语音输入，集成RAG知识检索与TTS语音播报。

---

## 任务概述

开发Android端AI智能问答功能，用户可通过文字或语音向系统提问农业相关问题，系统基于RAG（检索增强生成）返回带来源引用的专业回答，并支持TTS语音播报。

## 功能列表

| 序号 | 功能 | 说明 | 状态 |
|------|------|------|------|
| 1 | 文字问答 | EditText输入问题，发送到后端RAG服务 | ✅ |
| 2 | 语音问答 | MediaRecorder录音（AAC），上传到后端ASR+QA | ✅ |
| 3 | RAG知识检索 | 后端返回带来源引用的回答 | ✅ |
| 4 | 来源引用展示 | AI回答气泡下方展示知识来源 | ✅ |
| 5 | 问答历史 | 分页加载历史问答记录 | ✅ |
| 6 | TTS语音播报 | TextToSpeech朗读AI回答 | ✅ |

## 架构设计

```
QaFragment (UI层)
  ├── EditText + 发送按钮 → 文字输入
  ├── MediaRecorder → 语音录制
  ├── TextToSpeech → 语音播报
  └── RecyclerView + ChatAdapter → 聊天列表
       ├── UserViewHolder (绿色气泡，右对齐)
       ├── AiViewHolder (白色气泡，左对齐 + 来源引用 + TTS按钮)
       └── ErrorViewHolder (红色居中错误)

QaViewModel (业务逻辑层)
  ├── askQuestion(String) → 文字问答
  ├── askVoice(File) → 语音问答
  ├── loadHistory() / loadMoreHistory() → 分页
  ├── speakAnswer(String) → TTS (TextToSpeech由Fragment注入)
  └── ChatMessage (内部类，3种类型)

GreenhouseRepository (数据层)
  ├── ask(QaRequest) → POST /api/qa/ask
  ├── askVoice(MultipartBody.Part) → POST /api/qa/ask/voice
  └── getQaHistory(page, size) → GET /api/qa/history
```

## 开发规范检查

| 规范要求 | 状态 |
|----------|------|
| Activity/Fragment不含业务逻辑 | ✅ 通过 |
| ViewModel不持有Context/ContentResolver | ✅ 通过（TTS由Fragment注入） |
| 网络请求在后台线程 | ✅ 通过（ExecutorService） |
| 权限申请使用AndroidX ActivityResult API | ✅ 通过 |
| 使用ViewBinding | ✅ 通过 |
| ViewModel + LiveData模式 | ✅ 通过 |

## 变更文件

### 新增文件 (16个)
- `app/.../data/model/QaRequest.java`
- `app/.../data/model/QaResponse.java`
- `app/.../data/model/QaHistoryItem.java`
- `app/.../viewmodel/QaViewModel.java`
- `app/.../adapter/ChatAdapter.java`
- `app/.../ui/qa/QaFragment.java`
- `app/src/main/res/layout/fragment_qa.xml`
- `app/src/main/res/layout/item_chat_bubble_user.xml`
- `app/src/main/res/layout/item_chat_bubble_ai.xml`
- `app/src/main/res/layout/item_chat_bubble_error.xml`
- `app/src/main/res/drawable/ic_mic.xml`
- `app/src/main/res/drawable/ic_send.xml`
- `app/src/main/res/drawable/ic_volume_up.xml`
- `app/src/main/res/drawable/bg_chat_bubble_user.xml`
- `app/src/main/res/drawable/bg_chat_bubble_ai.xml`
- `app/src/main/res/drawable/bg_input_qa.xml`

### 修改文件 (3个)
- `app/.../data/api/GreenhouseApiService.java`
- `app/.../data/repository/GreenhouseRepository.java`
- `app/.../ui/common/MainActivity.java`

---

## 技术要点

### 语音录制
- 使用 `android.media.MediaRecorder`
- 输出格式：AAC（`MediaRecorder.OutputFormat.MPEG_4` + `AudioEncoder.AAC`）
- 权限：`RECORD_AUDIO`，通过 `ActivityResultContracts.RequestPermission` 申请

### TTS语音播报
- 使用 `android.speech.tts.TextToSpeech`
- 语言设置：`Locale.CHINESE`
- 由Fragment初始化后通过setter注入ViewModel

### 聊天UI
- RecyclerView + 多ViewType（`getItemViewType()` 返回 TYPE_USER/TYPE_AI/TYPE_ERROR）
- 自动滚动：`smoothScrollToPosition(adapter.getItemCount() - 1)`
- 来源引用：AiViewHolder中可展开/折叠的引用信息区域

### API接口
```
POST   /api/qa/ask          — 文字问答 (QaRequest JSON)
POST   /api/qa/ask/voice    — 语音问答 (@Multipart audio)
GET    /api/qa/history?page=&size=  — 问答历史（分页）
```

---

## 关联任务

- **TASK-C01**（用户认证）：API鉴权依赖
- **TASK-C09**（RAG问答模块）：后端RAG服务
- **TASK-C10**（语音识别模块）：后端ASR服务

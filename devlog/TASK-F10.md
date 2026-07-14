---
id: TASK-F10
title: 专家咨询模块开发
module: F10 APP专家咨询
type: task
priority: high
status: completed
created: 2026-07-14
started: 2026-07-14
completed: 2026-07-14
assignee: AI助手
related_docs:
  - DEVLOG-步骤29
tags: [android, expert, chat, websocket, stomp, MVVM]
---

# 专家咨询模块开发

---

## 任务描述

### 背景

F10 专家咨询模块是 APP 端的核心 AIoT 功能之一，让大棚用户能够在线咨询农业专家，通过文字、图片、视频、环境快照等多种方式沟通。该模块依赖后端 C16（实时聊天）和 C17（专家授权）模块。

### 目标

完成专家咨询模块的全部12项APP端功能，包括专家列表、咨询发起、实时聊天、图片/视频发送、环境快照、授权管理和未读通知。

### 范围

Android 原生开发（Java），覆盖数据模型、API接口、ViewModel、Adapter、Activity/Fragment 全栈。

---

## 验收标准

1. [x] 专家列表展示（在线状态、评分、咨询次数）
2. [x] 咨询发起（选择专家 → 输入问题主题 → 创建对话）
3. [x] 实时文字聊天（REST API + WebSocket STOMP 双通道）
4. [x] 图片消息发送（拍照 + Multipart 上传）
5. [x] 视频消息发送（录像 + 文件上传）
6. [x] 环境快照发送（一键发送大棚传感器数据卡片）
7. [x] 授权管理（待处理 Tab + 已授权 Tab，同意/拒绝/撤销）
8. [x] 对话历史查看
9. [x] 未读消息数量查询
10. [x] WebSocket 不可用时自动降级到 REST 轮询（3s）
11. [x] 构建成功，无编译错误

---

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `data/model/ExpertInfo.java` | 新增 | 专家信息模型 |
| `data/model/ConversationInfo.java` | 新增 | 对话信息模型 |
| `data/model/ChatMessage.java` | 新增 | 聊天消息模型（含 SnapshotData 内部类） |
| `data/model/CreateConversationRequest.java` | 新增 | 创建对话请求 |
| `data/model/SendMessageRequest.java` | 新增 | 发送消息请求 |
| `data/model/SnapshotRequest.java` | 新增 | 环境快照请求 |
| `data/model/AuthorizationInfo.java` | 新增 | 授权信息模型 |
| `data/model/UnreadResponse.java` | 新增 | 未读消息响应模型 |
| `data/websocket/StompClient.java` | 新增 | 轻量 STOMP 协议 WebSocket 客户端 |
| `data/api/GreenhouseApiService.java` | 修改 | 新增17个API端点 |
| `data/repository/GreenhouseRepository.java` | 修改 | 新增17个Repository方法 |
| `data/api/ApiClient.java` | 修改 | 新增 getBaseUrl() / getAuthToken() |
| `viewmodel/ExpertViewModel.java` | 新增 | 专家咨询全部业务逻辑 |
| `adapter/ExpertAdapter.java` | 新增 | 专家列表适配器 |
| `adapter/ChatMessageAdapter.java` | 新增 | 多ViewType聊天消息适配器 |
| `adapter/AuthorizationAdapter.java` | 新增 | 授权列表适配器（双模式） |
| `ui/expert/ExpertListActivity.java` | 新增 | 专家列表页 |
| `ui/expert/ChatActivity.java` | 新增 | 聊天页 |
| `ui/expert/AuthorizationActivity.java` | 新增 | 授权管理页 |
| `ui/expert/PendingAuthorizationFragment.java` | 新增 | 待处理授权 Fragment |
| `ui/expert/ActiveAuthorizationFragment.java` | 新增 | 已授权 Fragment |
| `ui/profile/ProfileFragment.java` | 修改 | 新增"专家咨询"+"授权管理"入口 |
| `res/layout/activity_expert_list.xml` | 新增 | 专家列表布局 |
| `res/layout/activity_chat.xml` | 新增 | 聊天页布局 |
| `res/layout/activity_authorization.xml` | 新增 | 授权管理布局 |
| `res/layout/item_expert.xml` | 新增 | 专家列表项布局 |
| `res/layout/item_chat_message_left.xml` | 新增 | 接收消息气泡 |
| `res/layout/item_chat_message_right.xml` | 新增 | 发送消息气泡 |
| `res/layout/item_snapshot_card.xml` | 新增 | 环境快照卡片 |
| `res/layout/item_authorization.xml` | 新增 | 授权列表项布局 |
| `res/layout/fragment_profile.xml` | 修改 | 新增两个入口按钮 |
| `build.gradle` | 修改 | 新增 viewpager2 依赖 |
| `AndroidManifest.xml` | 修改 | 注册3个新 Activity |

---

## 技术架构

### 双通道通信设计

```
┌─────────────────────────────────────────────────────────┐
│                     ExpertViewModel                     │
│                                                         │
│  ┌──────────────────┐        ┌──────────────────────┐  │
│  │   REST API 通道   │        │  WebSocket STOMP 通道  │  │
│  │  (保证送达)        │        │  (实时推送)            │  │
│  │  - 发送消息        │        │  - 接收新消息推送      │  │
│  │  - 上传图片/视频   │        │  - 订阅对话频道        │  │
│  │  - 管理对话/授权   │        │  - 10s 心跳保活        │  │
│  └────────┬─────────┘        └──────────┬───────────┘  │
│           │                              │              │
│           │  WebSocket 不可用时           │              │
│           │  ↓ 自动降级                   │              │
│           │  REST 轮询 (3s 间隔)          │              │
│           │  loadMessages() +             │              │
│           │  loadUnreadCount()            │              │
│           │                              │              │
└───────────┴──────────────────────────────┴──────────────┘
```

### 消息类型

| 类型 | ViewType | 发送方式 | UI 展示 |
|------|----------|----------|---------|
| TEXT | LEFT/RIGHT | REST + WS | 气泡文字 |
| IMAGE | LEFT/RIGHT | Multipart 上传 | 图片（Glide） |
| VIDEO | LEFT/RIGHT | 文件上传 | 视频缩略图 |
| ENV_SNAPSHOT | SNAPSHOT | REST | 传感器数据卡片 |

### 授权生命周期

```
PENDING ──同意──→ APPROVED (7天有效期) ──到期──→ EXPIRED
   │                 │
   └──拒绝──→ REJECTED  └──撤销──→ REVOKED
```

---

## 完成结果

12项APP端功能全部完整开发，构建成功（BUILD SUCCESSFUL），代码覆盖数据模型、API、WebSocket、ViewModel、Adapter、Activity/Fragment、布局全栈。后续需后端 WebSocket 服务启动后进行联调测试。

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-14
- 关联DEVLOG：步骤29
- 用户明确要求：功能不裁剪，全部完整开发
- 后续联调：需要后端 C16（实时聊天）和 C17（专家授权）模块运行

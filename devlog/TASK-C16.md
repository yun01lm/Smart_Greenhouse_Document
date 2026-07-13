---
id: TASK-C16
title: 实时聊天模块开发
module: C16 实时聊天
type: task
priority: high
status: completed
created: 2026-07-13
started: 2026-07-13
completed: 2026-07-13
assignee: AI助手
related_docs:
  - DEVLOG-步骤16
tags: [chat, conversation, message, snapshot]
---

# 实时聊天模块开发

---

## 任务描述

### 背景

C16 实时聊天是专家咨询系统的核心通信模块。用户通过APP与专家进行实时文字/图片/视频聊天，支持发送大棚环境快照。

### 目标

完成对话管理、消息收发、环境快照生成和未读统计功能。

### 范围

仅后端代码，消息实时推送复用已有 WebSocket 基础设施。

---

## 验收标准

1. [x] ChatConversation 实体创建完成，与 DB 表 21 一致
2. [x] ChatMessage 实体创建完成，支持4种消息类型
3. [x] 对话创建：校验专家角色，自动发送首条消息
4. [x] 专家首次回复自动将对话从 WAITING 变为 ACTIVE
5. [x] 环境快照生成（调用 SensorDataService 获取实时数据）
6. [x] 未读消息统计（按用户/专家/对话三维度）
7. [x] 7个 API 端点与接口文档一致

---

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `entity/ChatConversation.java` | 新建 | DB表21，WAITING/ACTIVE/CLOSED状态 |
| `entity/ChatMessage.java` | 新建 | DB表22，4种消息类型，已读追踪 |
| `repository/ChatConversationRepository.java` | 新建 | 按用户/专家/状态查询 |
| `repository/ChatMessageRepository.java` | 新建 | 分页+未读统计+批量已读 |
| `module/chat/service/ChatService.java` | 新建 | 对话管理+消息收发+快照+未读 |
| `module/chat/controller/ChatController.java` | 新建 | 7个端点 |
| `module/chat/dto/ConversationRequest.java` | 新建 | 创建对话请求 |
| `module/chat/dto/ConversationResponse.java` | 新建 | 对话响应（含未读数+最后消息） |
| `module/chat/dto/MessageResponse.java` | 新建 | 消息响应 |
| `module/chat/dto/SendMessageRequest.java` | 新建 | 发送消息请求 |

---

## 完成结果

实时聊天模块支持完整的对话生命周期：创建→消息收发→关闭。消息支持文字/图片/视频/环境快照四种类型。环境快照调用SensorDataService获取大棚实时传感器数据生成JSON快照。未读消息统计支持用户端和专家端分别计数。

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-13
- 关联DEVLOG：步骤16
- 消息实时推送通过 WebSocket STOMP 实现（复用 C11 基础设施）

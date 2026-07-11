---
id: TECH-007
title: 专家咨询系统技术设计文档
type: tech-design
module: expert-system
tags:
  - 专家咨询
  - 实时聊天
  - 环境快照
  - 7天授权
  - 双轨制
  - 兜底机制
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - /document/docs/prd/smart-greenhouse-prd.md
  - /document/docs/api/api-design.md
---

## 概述

专家咨询系统是AI诊断的兜底机制，当AI诊断置信度不足或用户需要人工专业判断时，用户可一键求助平台入驻专家。系统提供实时聊天、环境快照共享、环境数据授权（7天有效期）等功能，通过"双轨制"设计平衡专家辅助诊断的便利性和用户隐私保护。

**双轨制设计**：
- **轨道一（环境快照）**：用户可在聊天中发送当前大棚环境的即时快照，专家可查看该时刻的传感器数据。一次性使用，类似微信发位置。
- **轨道二（数据授权）**：专家发起数据授权请求，用户同意后，专家可在7天内查看该大棚的实时和历史传感器数据。到期自动过期，用户可随时撤销。

**核心流程**：
1. 用户求助 → 创建咨询对话
2. 实时聊天（文字/图片/视频/环境快照）
3. 专家判断需要更多数据 → 发起授权请求
4. 用户同意 → 7天数据查看权限
5. 诊断完成 → 关闭对话

## 架构设计

### 专家咨询系统整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层                                    │
│                                                                  │
│  ┌──────────────────────┐          ┌──────────────────────┐     │
│  │   Android APP        │          │   Vue 3 Web          │     │
│  │   (棚主/员工端)       │          │   (专家工作台)        │     │
│  │                      │          │                      │     │
│  │  • 求助入口          │          │  • 求助列表          │     │
│  │  • 聊天界面          │◄────────►│  • 聊天界面          │     │
│  │  • 环境快照发送       │ STOMP    │  • 环境数据查看       │     │
│  │  • 授权管理          │ WebSocket│  • 授权申请          │     │
│  │  • 专家列表          │          │  • 在线状态管理       │     │
│  └──────────┬───────────┘          └──────────┬───────────┘     │
│             │                                  │                 │
│             └──────────────┬───────────────────┘                 │
│                            │                                     │
├────────────────────────────┼─────────────────────────────────────┤
│                            ▼                        平台层        │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              专家咨询系统后端模块                           │   │
│  │                                                           │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │   │
│  │  │ C16 实时聊天  │  │ C17 专家授权  │  │ C11 WebSocket│    │   │
│  │  │              │  │              │  │  推送        │    │   │
│  │  │ • 对话管理   │  │ • 授权请求   │  │ • 消息推送   │    │   │
│  │  │ • 消息收发   │  │ • 同意/拒绝  │  │ • 通知推送   │    │   │
│  │  │ • 快照生成   │  │ • 自动过期   │  │ • 状态同步   │    │   │
│  │  │ • 文件存储   │  │ • 主动撤销   │  │              │    │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────────────┘    │   │
│  │         │                 │                                │   │
│  │         ▼                 ▼                                │   │
│  │  ┌──────────────────────────────────────────────────┐     │   │
│  │  │                  MySQL 存储层                      │     │   │
│  │  │  chat_conversations  │  chat_messages             │     │   │
│  │  │  data_authorizations │  expert_availability        │     │   │
│  │  └──────────────────────────────────────────────────┘     │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 数据库表关系

```
chat_conversations (对话)
├── user_id (FK → users, 求助者)
├── expert_id (FK → users, 专家)
├── greenhouse_id (FK → greenhouses, 关联大棚)
├── subject (咨询主题)
├── status: WAITING → ACTIVE → CLOSED
└── diagnostic_id (FK → diagnostic_records, 关联诊断记录, 可空)

chat_messages (消息)
├── conversation_id (FK → chat_conversations)
├── sender_id (发送者)
├── sender_type: USER | EXPERT
├── message_type: TEXT | IMAGE | VIDEO | ENV_SNAPSHOT
├── content (文字内容)
├── file_path (文件路径)
├── snapshot_data (JSON, 环境快照)
├── read_status
└── INDEX: (conversation_id, created_at)

data_authorizations (授权)
├── expert_id → user_id → greenhouse_id
├── status: PENDING → APPROVED → EXPIRED / REVOKED / REJECTED
├── requested_at → approved_at → expires_at (7天)
└── revoked_at, revoked_by, reason

expert_availability (专家在线)
├── expert_id (FK → users)
├── is_online (0/1)
├── last_active_at
├── max_concurrent (最大同时咨询数, 默认5)
└── current_count (当前进行中咨询数)
```

## 数据流

### 用户发起求助流

```
触发场景:
  a) 诊断结果置信度<70%，用户点击"求助专家"
  b) 用户主动在APP端点击"专家咨询"

流程:
  → GET /api/v1/experts (获取在线专家列表)
  → 用户选择专家 → POST /api/v1/chat/conversations
      请求体: { "expert_id": 5, "greenhouse_id": 3, "subject": "番茄叶子发黄" }
  → 创建 chat_conversations (status=WAITING)
  → WebSocket通知专家:
      /topic/expert/notifications
      { "type": "NEW_CONSULTATION", "conversationId": 1, "userName": "张农户" }
  → 专家接受 (status → ACTIVE):
      PUT /api/v1/chat/conversations/{id}/accept
  → 双方可开始聊天
```

### 实时聊天消息流

```
用户发送消息 → STOMP SEND /app/chat/send
  {
    "conversationId": 1,
    "messageType": "TEXT",
    "content": "叶子发黄，边缘有褐色斑点"
  }
  → Spring处理:
      1. 校验对话状态 (ACTIVE)
      2. 存储 chat_messages
      3. STOMP推送 /topic/conversation/1
      4. 更新未读计数
  → 专家Web端实时收到消息

专家回复 → 同上流程，sender_type=EXPERT

支持消息类型:
  TEXT:    纯文本消息
  IMAGE:   图片消息 (file_path指向上传的图片)
  VIDEO:   视频消息 (file_path指向上传的视频)
  ENV_SNAPSHOT: 环境快照 (snapshot_data包含传感器数据)
```

### 环境快照发送流

```
用户在聊天中点击"发送环境快照" → STOMP SEND /app/chat/snapshot
  {
    "conversationId": 1,
    "greenhouseId": 3,
    "groupIds": [1, 2, 3]  // 可选：指定哪些传感器组
  }
  → 后端处理:
      1. 权限校验: 用户是否有该大棚访问权限
      2. 查询InfluxDB各组最新传感器数据
      3. 查询当前预警信息
      4. 组装快照JSON:
          {
            "greenhouseName": "1号大棚",
            "capturedAt": "2026-07-11T10:30:00",
            "groups": [
              {
                "groupId": 1, "zoneLabel": "东侧",
                "sensors": { "TEMP": 28.5, "HUMIDITY": 65.2, ... }
              }
            ],
            "avgTemp": 28.1, "avgHumidity": 64.8, "alertCount": 0
          }
      5. 存储 chat_messages (messageType=ENV_SNAPSHOT)
      6. STOMP推送 /topic/conversation/1
  → 专家端展示: 可点击快照卡片展开详细传感器数据和趋势图
```

### 专家授权流（完整生命周期）

```
1. 专家发起授权请求:
   POST /api/v1/expert/authorize/request
   { "greenhouse_id": 3, "reason": "需要查看历史温度趋势辅助诊断" }
   → 创建 data_authorizations (status=PENDING)
   → WebSocket通知用户:
       /user/queue/notifications
       { "type": "AUTHORIZATION_REQUEST", "relatedId": 45, "expertName": "李明" }

2. 用户处理:
   同意:
     PUT /api/v1/expert/authorize/{id}/approve
     → status → APPROVED, expires_at = now + 7天
     → 通知专家: AUTHORIZATION_APPROVED
    
   拒绝:
     PUT /api/v1/expert/authorize/{id}/reject
     → status → REJECTED
     → 通知专家: AUTHORIZATION_REJECTED

3. 授权有效期内:
   专家查看大棚数据:
     GET /api/v1/expert/greenhouses/{id}/data
     → C18权限模块校验: 检查 data_authorizations 有效授权
     → 返回传感器数据 (含多组对比视图)
   
   权限校验逻辑:
     Authorization auth = authorizationRepository
         .findActiveByExpertAndGreenhouse(expertId, greenhouseId);
     if (auth == null) → 403 AUTHORIZATION_EXPIRED
     if (auth.getExpiresAt().isBefore(Instant.now())) → 403 AUTHORIZATION_EXPIRED

4. 自动过期:
   Spring @Scheduled (每小时执行):
     → 扫描 data_authorizations WHERE status=APPROVED AND expires_at < NOW()
     → 更新 status → EXPIRED
     → 通知专家: AUTHORIZATION_EXPIRED

5. 用户主动撤销:
   PUT /api/v1/expert/authorize/{id}/revoke
   → status → REVOKED, revoked_at=now
   → 专家立即失去数据查看权限
```

### 对话关闭流

```
用户或专家关闭对话 → PUT /api/v1/chat/conversations/{id}/close
  → status → CLOSED, closed_at = now
  → 双方不再能发送新消息
  → 聊天记录保留 (可查看历史)
  → 专家 current_count - 1
  → 如果有有效授权:
      自动撤销: data_authorizations → REVOKED
```

## 接口设计概要

### 专家咨询接口

| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/experts | 专家列表 (含在线状态) |
| POST | /api/v1/chat/conversations | 创建对话 (发起求助) |
| GET | /api/v1/chat/conversations | 对话列表 (用户/专家各自的) |
| GET | /api/v1/chat/conversations/{id}/messages | 消息历史 (分页) |
| POST | /api/v1/chat/messages | 发送消息 (REST备用) |
| POST | /api/v1/chat/snapshot | 发送环境快照 |
| PUT | /api/v1/chat/conversations/{id}/close | 关闭对话 |
| GET | /api/v1/chat/unread | 未读消息数 |

### 专家授权接口

| 方法 | 端点 | 说明 |
|------|------|------|
| POST | /api/v1/expert/authorize/request | 专家发起授权请求 |
| GET | /api/v1/expert/authorize/pending | 用户查看待处理请求 |
| PUT | /api/v1/expert/authorize/{id}/approve | 同意授权 |
| PUT | /api/v1/expert/authorize/{id}/reject | 拒绝授权 |
| PUT | /api/v1/expert/authorize/{id}/revoke | 撤销授权 |
| GET | /api/v1/expert/authorize/active | 有效授权列表 |
| GET | /api/v1/expert/authorize/history | 授权历史 |
| GET | /api/v1/expert/greenhouses | 已授权大棚列表 |

### 专家工作台接口

| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/expert/consultations | 求助列表 (待处理/进行中/已结束) |
| GET | /api/v1/expert/statistics | 咨询统计 |
| PUT | /api/v1/expert/status | 更新在线状态 |
| GET | /api/v1/expert/greenhouses/{id}/data | 查看已授权大棚传感器数据 |

### 消息分页查询

```
GET /api/v1/chat/conversations/{id}/messages?page=1&size=50&before_id=100

响应:
{
  "code": 200,
  "data": {
    "list": [
      {
        "id": 99,
        "senderType": "USER",
        "messageType": "ENV_SNAPSHOT",
        "content": "[环境快照] 1号大棚 · 2026-07-11 10:30",
        "snapshotData": { ... },
        "createdAt": "2026-07-11T10:30:00"
      },
      { "id": 98, "messageType": "TEXT", "content": "叶子发黄...", ... }
    ],
    "hasMore": true,
    "total": 156
  }
}
```

## 关键算法/逻辑

### 专家在线状态管理

```java
@Service
public class ExpertAvailabilityService {
    
    // 专家登录Web端 → 上线
    public void setOnline(Long expertId) {
        ExpertAvailability status = availabilityRepository
            .findByExpertId(expertId)
            .orElseGet(() -> {
                ExpertAvailability ea = new ExpertAvailability();
                ea.setExpertId(expertId);
                ea.setMaxConcurrent(5);
                ea.setCurrentCount(0);
                return ea;
            });
        status.setIsOnline(true);
        status.setLastActiveAt(Instant.now());
        availabilityRepository.save(status);
    }
    
    // 专家退出Web端 → 离线
    public void setOffline(Long expertId) {
        availabilityRepository.findByExpertId(expertId)
            .ifPresent(status -> {
                status.setIsOnline(false);
                availabilityRepository.save(status);
            });
    }
    
    // 检查专家是否可接受新咨询
    public boolean canAcceptConsultation(Long expertId) {
        return availabilityRepository.findByExpertId(expertId)
            .map(status -> status.getIsOnline() 
                && status.getCurrentCount() < status.getMaxConcurrent())
            .orElse(false);
    }
}
```

### 授权过期定时扫描

```java
@Component
public class AuthorizationExpiryScheduler {
    
    @Scheduled(cron = "0 0 * * * *")  // 每小时整点执行
    public void scanExpiredAuthorizations() {
        List<DataAuthorization> expired = authorizationRepository
            .findByStatusAndExpiresAtBefore(
                AuthorizationStatus.APPROVED, Instant.now());
        
        for (DataAuthorization auth : expired) {
            auth.setStatus(AuthorizationStatus.EXPIRED);
            authorizationRepository.save(auth);
            
            // 通知专家
            NotificationVO notification = NotificationVO.builder()
                .type("AUTHORIZATION_EXPIRED")
                .title("数据授权已过期")
                .content("您对大棚" + auth.getGreenhouseId() + "的数据授权已过期")
                .relatedId(auth.getId())
                .build();
            webSocketService.sendToExpert(auth.getExpertId(), notification);
            
            log.info("授权已过期: authId={}, expertId={}, greenhouseId={}", 
                     auth.getId(), auth.getExpertId(), auth.getGreenhouseId());
        }
    }
}
```

### 对话关闭时级联处理

```java
@Service
@Transactional
public class ChatConversationService {
    
    public void closeConversation(Long conversationId, Long operatorId) {
        ChatConversation conv = conversationRepository.findById(conversationId)
            .orElseThrow(() -> new BusinessException(ErrorCode.CONVERSATION_NOT_FOUND));
        
        if (conv.getStatus() != ConversationStatus.ACTIVE) {
            throw new BusinessException(ErrorCode.CONVERSATION_CLOSED);
        }
        
        // 1. 关闭对话
        conv.setStatus(ConversationStatus.CLOSED);
        conv.setClosedAt(Instant.now());
        conversationRepository.save(conv);
        
        // 2. 自动撤销有效授权
        List<DataAuthorization> activeAuths = authorizationRepository
            .findByExpertIdAndUserIdAndGreenhouseIdAndStatus(
                conv.getExpertId(), conv.getUserId(), 
                conv.getGreenhouseId(), AuthorizationStatus.APPROVED);
        for (DataAuthorization auth : activeAuths) {
            auth.setStatus(AuthorizationStatus.REVOKED);
            auth.setRevokedAt(Instant.now());
            auth.setRevokedBy(operatorId);
            authorizationRepository.save(auth);
        }
        
        // 3. 更新专家当前咨询数
        expertAvailabilityService.decrementCurrentCount(conv.getExpertId());
        
        // 4. 通知双方
        webSocketService.sendToConversation(conversationId, 
            new NotificationVO("CONVERSATION_CLOSED", "对话已关闭"));
    }
}
```

## 技术选型理由

| 决策项 | 选择 | 理由 |
|--------|------|------|
| 聊天通道 | WebSocket STOMP复用 | ADR-008决策：复用已有基础设施，不引入独立消息中间件 |
| 消息持久化 | MySQL存储 | 聊天记录需要长期保存，MySQL支持分页查询历史消息 |
| 双轨制授权 | 快照(一次性) + 授权(7天) | ADR-009决策：平衡便利性和隐私保护 |
| 自动过期 | @Scheduled定时扫描 | 轻量级方案，每小时扫描一次足够，无需引入消息队列 |
| 文件存储 | 本地文件系统 | 聊天图片/视频存储在uploads/chat/目录下 |
| 在线状态 | MySQL + WebSocket心跳 | 通过WebSocket连接状态 + 定时心跳判断在线 |

## 注意事项

1. **消息顺序保证**：聊天消息必须按 created_at 排序。chat_messages表必须建立 (conversation_id, created_at) 联合索引，确保分页查询按时间正序返回。

2. **授权安全**：专家查看大棚数据时，必须同时校验 data_authorizations 状态=APPROVED 且 expires_at > NOW()。不能仅凭 role=EXPERT 就允许访问。

3. **对话关闭时撤销授权**：对话关闭时必须自动撤销该对话关联的有效授权，防止专家在对话结束后仍能查看数据。

4. **并发咨询限制**：专家同时进行的咨询数受 max_concurrent 限制（默认5）。创建对话时需要检查专家是否达到上限。

5. **离线消息处理**：WebSocket断开期间的消息不会丢失（已存储到MySQL）。客户端重连后通过 REST API 拉取离线消息。

6. **环境快照实时性**：环境快照生成时查询的是InfluxDB最新数据（而非缓存），确保快照反映发送时刻的真实环境状态。

7. **专家角色限制**：专家只能通过Web端访问，不能通过APP登录。专家无设备控制权限、无大棚管理权限、无员工管理权限。

8. **24小时未处理请求过期**：PENDING状态的授权请求如果24小时内用户未处理，自动标记为EXPIRED。避免积累无响应请求。

---
id: TECH-006
title: 实时通信层技术设计文档
type: tech-design
module: realtime-comm
tags:
  - MQTT
  - WebSocket
  - STOMP
  - 实时推送
  - 设备通信
  - 聊天
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - /document/docs/prd/dashboard-realtime.md
  - /document/docs/prd/expert-consultation.md
  - /document/docs/api/sensor/sensor-api.md
  - /document/docs/api/chat/chat-api.md
---

## 概述

实时通信层是系统的神经中枢，承担三大核心通信职责：MQTT设备通信（传感器数据采集+设备指令下发）、WebSocket数据推送（实时数据+预警通知+健康评分）和STOMP实时聊天（专家-用户双向通信）。基于"复用WebSocket基础设施"原则，聊天功能复用Spring STOMP通道，不引入独立消息中间件。

**通信协议分工**：
- **MQTT**：ESP32设备 ↔ 后端（传感器数据上报、设备控制指令、心跳保活）
- **WebSocket + STOMP**：后端 ↔ APP/Web端（实时数据推送、预警通知、聊天消息）
- **REST API**：后端 ↔ APP/Web端（CRUD操作、文件上传、非实时查询）

## 架构设计

### 通信层整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        应用层                                    │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Android APP │  │  Vue 3 Web   │  │  Vue 3 Web   │          │
│  │  (STOMP客户端)│  │  (STOMP客户端)│  │  (STOMP客户端)│          │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │
│         │                 │                  │                   │
│         └─────────────────┼──────────────────┘                   │
│                           │ WebSocket (wss://)                   │
├───────────────────────────┼──────────────────────────────────────┤
│                           ▼                        平台层         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              Spring Boot 实时通信层                        │   │
│  │                                                           │   │
│  │  ┌──────────────────┐  ┌──────────────────────────────┐   │   │
│  │  │ MQTT 消费模块    │  │  WebSocket (STOMP) 服务       │   │   │
│  │  │ (Eclipse Paho)   │  │                              │   │   │
│  │  │                  │  │  /topic/greenhouse/{id}/*    │   │   │
│  │  │ 订阅Topic:       │  │    → 实时数据推送            │   │   │
│  │  │ greenhouse/+/    │  │  /topic/conversation/{id}    │   │   │
│  │  │ group/+/device/  │  │    → 聊天消息推送            │   │   │
│  │  │ +/sensor/+       │  │  /user/queue/*               │   │   │
│  │  │                  │  │    → 个人通知队列            │   │   │
│  │  └────────┬─────────┘  └──────────────┬───────────────┘   │   │
│  │           │                           │                    │   │
│  │           ▼                           ▼                    │   │
│  │  ┌──────────────────┐  ┌──────────────────────────────┐   │   │
│  │  │ InfluxDB 写入    │  │  STOMP消息处理器              │   │   │
│  │  │ (时序数据存储)    │  │  /app/chat/send              │   │   │
│  │  │                  │  │  /app/chat/snapshot           │   │   │
│  │  │ + 预警引擎触发    │  │  /app/chat/read              │   │   │
│  │  └──────────────────┘  └──────────────────────────────┘   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                           │                                      │
├───────────────────────────┼──────────────────────────────────────┤
│                           ▼                        网络层         │
│  ┌──────────────────┐  ┌──────────────────────────────┐        │
│  │ Mosquitto Broker │  │  frp 内网穿透                 │        │
│  │ (MQTT :1883)     │  │  (WebSocket + MQTT 公网)     │        │
│  └──────────────────┘  └──────────────────────────────┘        │
│                           │                                      │
├───────────────────────────┼──────────────────────────────────────┤
│                           ▼                        感知层         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  ESP32 × 3-5 (MQTT客户端)  +  ESP32-CAM (RTSP推流)       │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### MQTT Topic设计

```
数据上报 (ESP32 → Broker → Spring Boot):
  greenhouse/{gh_id}/group/{group_id}/device/{dev_id}/sensor/{sensor_type}
  sensor_type: temp, humidity, light, co2, o2, soil_temp, soil_humidity, ec, n, p, k

设备控制 (Spring Boot → Broker → ESP32):
  greenhouse/{gh_id}/device/{dev_id}/control/{actuator_id}

设备状态 (ESP32 → Broker → Spring Boot):
  greenhouse/{gh_id}/device/{dev_id}/status

心跳 (ESP32 → Broker → Spring Boot):
  greenhouse/{gh_id}/device/{dev_id}/heartbeat
```

### WebSocket STOMP端点设计

```
连接端点: /ws/connect

订阅 (数据推送):
  /topic/greenhouse/{id}/realtime       → 实时传感器数据 (含组信息)
  /topic/greenhouse/{id}/alerts         → 预警推送
  /topic/greenhouse/{id}/health         → 健康评分推送
  /topic/device/{id}/status             → 设备状态变更

订阅 (实时聊天):
  /user/queue/messages                  → 个人消息队列 (聊天消息)
  /topic/conversation/{id}              → 对话消息推送 (双方订阅)
  /topic/expert/notifications           → 专家通知 (新求助/授权变更)
  /topic/user/notifications             → 用户通知 (授权请求/新消息)

发送 (实时聊天):
  /app/chat/send                        → 发送消息
  /app/chat/snapshot                    → 发送环境快照
  /app/chat/read                        → 标记已读
```

## 数据流

### MQTT传感器数据消费流

```
ESP32上报 → Mosquitto Broker → Spring Boot MqttSubscriber
  → 接收消息: topic + payload
  → 解析topic: 提取 greenhouse_id, group_id, device_id, sensor_type
  → 解析payload: {"value": 25.3, "timestamp": 1690000000, "group_id": 2}
  → 数据校验: 数值范围检查、group_id一致性检查
  → 写入InfluxDB (批量缓冲):
      batchBuffer.add(WriteRecord...)
      if (buffer.size() >= 100 || elapsed >= 5000ms) → flush()
  → 发布Spring Event (异步):
      SensorDataReceivedEvent → 触发预警引擎检查
  → WebSocket推送 (STOMP):
      SimpMessagingTemplate.convertAndSend(
          "/topic/greenhouse/" + ghId + "/realtime",
          assembleRealtimeData(ghId)
      )
```

### WebSocket实时数据推送流

```
后端定时任务 (每30秒) 或 MQTT消费后触发:
  → 查询InfluxDB各传感器组最新值
  → 组装推送数据:
      {
        "greenhouseId": 3,
        "timestamp": "2026-07-11T10:30:00",
        "groups": [
          {
            "groupId": 1, "zoneLabel": "东侧", "online": true,
            "sensors": { "TEMP": 28.5, "HUMIDITY": 65.2, ... }
          },
          { "groupId": 2, "zoneLabel": "西侧", ... }
        ]
      }
  → STOMP推送: /topic/greenhouse/{id}/realtime
  → APP/Web端订阅接收 → 更新仪表盘
```

### 设备控制指令流

```
APP/Web端下发控制 → POST /api/v1/control/actuator
  → 权限校验 (@RequireGreenhouseAccess + @RequireFunction)
  → 构建MQTT消息:
      topic: greenhouse/{gh_id}/device/{dev_id}/control/{actuator_id}
      payload: {"action": "ON", "timestamp": 1690000000}
  → MQTT publish (QoS 1, 至少一次送达)
  → 记录 control_logs
  → 等待设备状态上报 (异步):
      ESP32收到指令 → 执行 → 上报状态
  → 收到状态确认 → WebSocket推送状态变更:
      /topic/device/{id}/status
  → 超时处理 (5秒未收到状态确认):
      标记指令执行超时 → WebSocket推送超时通知
```

### 聊天消息流（STOMP）

```
发送方 (APP/Web) → STOMP SEND /app/chat/send:
  {
    "conversationId": 1,
    "messageType": "TEXT",
    "content": "请问这个叶子发黄是什么问题？"
  }
  → Spring @MessageMapping("/chat/send")
  → 存储 chat_messages (MySQL)
  → STOMP推送:
      ├── /topic/conversation/1 → 双方都收到消息
      └── /user/{receiverId}/queue/messages → 接收方个人通知
  → 更新未读消息数
```

### 环境快照发送流

```
发送方 → STOMP SEND /app/chat/snapshot:
  {
    "conversationId": 1,
    "greenhouseId": 3,
    "groupIds": [1, 2, 3]
  }
  → Spring @MessageMapping("/chat/snapshot")
  → 查询InfluxDB各组最新传感器数据
  → 组装快照JSON:
      {
        "greenhouseName": "1号大棚",
        "capturedAt": "2026-07-11T10:30:00",
        "groups": [
          { "groupId": 1, "zoneLabel": "东侧", "sensors": {...} },
          ...
        ],
        "avgTemp": 28.1, "avgHumidity": 64.8, "alertCount": 0
      }
  → 存储 chat_messages (messageType=ENV_SNAPSHOT)
  → STOMP推送: /topic/conversation/{id}
```

## 接口设计概要

### MQTT配置

```yaml
mqtt:
  broker: ${MQTT_BROKER:tcp://localhost:1883}
  client-id: spring-greenhouse-server
  username: ${MQTT_USER:greenhouse}
  password: ${MQTT_PASSWORD}
  qos: 1
  keep-alive: 60
  topics:
    sensor: greenhouse/+/group/+/device/+/sensor/+
    status: greenhouse/+/device/+/status
    heartbeat: greenhouse/+/device/+/heartbeat
```

### WebSocket STOMP配置

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    
    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        // 客户端订阅前缀 (服务端→客户端推送)
        registry.enableSimpleBroker("/topic", "/user");
        // 客户端发送前缀 (客户端→服务端)
        registry.setApplicationDestinationPrefixes("/app");
        // 用户队列前缀
        registry.setUserDestinationPrefix("/user");
    }
    
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws/connect")
                .setAllowedOriginPatterns("*")
                .withSockJS();  // 降级兼容 (Web端)
    }
}
```

### STOMP消息格式规范

**聊天消息 (TEXT)**：
```json
// 发送
{"conversationId": 1, "messageType": "TEXT", "content": "请问..."}

// 接收
{
  "id": 123, "conversationId": 1,
  "senderId": 5, "senderType": "USER",
  "senderName": "张农户",
  "messageType": "TEXT", "content": "请问...",
  "createdAt": "2026-07-11T10:30:00"
}
```

**图片/视频消息**：
```json
{"messageType": "IMAGE", "content": "[图片]", "filePath": "/uploads/chat/abc.jpg"}
```

**环境快照**：
```json
// 发送
{"conversationId": 1, "greenhouseId": 3, "groupIds": [1, 2, 3]}

// 接收 (后端自动组装)
{
  "messageType": "ENV_SNAPSHOT",
  "content": "[环境快照] 1号大棚 · 2026-07-11 10:30",
  "snapshotData": {
    "greenhouseName": "1号大棚", "capturedAt": "...",
    "groups": [{...}], "avgTemp": 28.1
  }
}
```

**通知消息**：
```json
{
  "id": 789,
  "type": "AUTHORIZATION_REQUEST",
  "title": "专家李明请求查看您的1号大棚数据",
  "content": "授权有效期为7天，您可以随时撤销",
  "relatedId": 45,
  "createdAt": "2026-07-11T10:30:00"
}
```

## 关键算法/逻辑

### MQTT消息批量写入

```java
@Component
public class MqttDataBuffer {
    private final List<WriteRecord> buffer = new ArrayList<>();
    private long lastFlushTime = System.currentTimeMillis();
    private static final int BATCH_SIZE = 100;
    private static final long FLUSH_INTERVAL_MS = 5000;
    
    @EventListener
    public void onSensorData(SensorDataReceivedEvent event) {
        synchronized (buffer) {
            buffer.add(convertToWriteRecord(event));
            
            if (buffer.size() >= BATCH_SIZE || 
                System.currentTimeMillis() - lastFlushTime >= FLUSH_INTERVAL_MS) {
                flush();
            }
        }
    }
    
    @Scheduled(fixedDelay = 5000)
    public void scheduledFlush() {
        synchronized (buffer) {
            if (!buffer.isEmpty()) {
                flush();
            }
        }
    }
    
    private void flush() {
        if (buffer.isEmpty()) return;
        influxDB.write(bucket, org, buffer);
        buffer.clear();
        lastFlushTime = System.currentTimeMillis();
    }
}
```

### WebSocket连接认证

```java
@Component
public class StompAuthInterceptor implements ChannelInterceptor {
    
    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) {
        StompHeaderAccessor accessor = StompHeaderAccessor.wrap(message);
        
        // CONNECT时验证Token
        if (StompCommand.CONNECT.equals(accessor.getCommand())) {
            String token = accessor.getFirstNativeHeader("Authorization");
            if (token == null || !token.startsWith("Bearer ")) {
                throw new MessageDeliveryException("缺少认证Token");
            }
            
            String jwt = token.substring(7);
            if (!jwtTokenProvider.validateToken(jwt)) {
                throw new MessageDeliveryException("Token无效或已过期");
            }
            
            // 解析用户信息 → 设置到Session
            Long userId = jwtTokenProvider.getUserId(jwt);
            accessor.setUser(new StompPrincipal(userId));
        }
        
        return message;
    }
}
```

### 聊天消息去重与顺序保证

```java
@Service
public class ChatMessageService {
    
    public ChatMessage processMessage(ChatMessageDTO dto, Long senderId) {
        // 1. 验证对话存在且未关闭
        ChatConversation conv = conversationRepository.findById(dto.getConversationId())
            .orElseThrow(() -> new BusinessException(ErrorCode.CONVERSATION_NOT_FOUND));
        if (conv.getStatus() == ConversationStatus.CLOSED) {
            throw new BusinessException(ErrorCode.CONVERSATION_CLOSED);
        }
        
        // 2. 存储消息 (MySQL主键保证唯一性)
        ChatMessage msg = new ChatMessage();
        msg.setConversationId(dto.getConversationId());
        msg.setSenderId(senderId);
        msg.setSenderType(determineSenderType(senderId, conv));
        msg.setMessageType(dto.getMessageType());
        msg.setContent(dto.getContent());
        msg.setFilePat(dto.getFilePat());
        msg.setCreatedAt(Instant.now());
        msg = chatMessageRepository.save(msg);
        
        // 3. 推送消息 (通过WebSocket)
        ChatMessageVO vo = convertToVO(msg);
        messagingTemplate.convertAndSend(
            "/topic/conversation/" + dto.getConversationId(), vo);
        
        // 4. 更新最后活跃时间
        conv.setUpdatedAt(Instant.now());
        conversationRepository.save(conv);
        
        return msg;
    }
}
```

## 技术选型理由

| 决策项 | 选择 | 理由 |
|--------|------|------|
| MQTT协议 | Eclipse Mosquitto 2.x | 申报书要求"MQTT协议"；轻量级物联网协议，适合ESP32 |
| MQTT客户端 | Eclipse Paho 1.2.x | Spring Integration原生集成，Java生态标准MQTT客户端 |
| WebSocket协议 | Spring WebSocket + STOMP | Spring原生支持，STOMP提供发布/订阅模型，适合消息推送 |
| 聊天复用WebSocket | ADR-008决策 | 不引入RabbitMQ/Netty等独立中间件，降低运维复杂度 |
| QoS等级 | QoS 1 (至少一次送达) | 传感器数据允许少量重复，但不允许丢失 |
| SockJS降级 | Web端启用 | 兼容不支持WebSocket的浏览器环境 |

## 注意事项

1. **MQTT消息必须携带group_id**：每条传感器消息的payload中必须包含group_id字段，后端校验topic中的group_id与payload中的一致，防止配置错误导致数据归属混乱。

2. **批量写入防数据丢失**：MQTT消费模块的批量写入缓冲区必须在应用关闭时flush。通过 @PreDestroy 方法或ShutdownHook确保缓冲区数据不丢失。

3. **WebSocket连接数限制**：单体Spring Boot的WebSocket连接数有限（通常几百到几千）。本地部署场景下同时在线用户数有限，但需要注意：如果连接数过多（>1000），考虑启用STOMP relay broker（如RabbitMQ）作为消息中继。

4. **消息持久化策略**：聊天消息必须存储到MySQL（chat_messages表），WebSocket仅做实时推送。断线重连后，客户端从数据库拉取离线消息（通过REST API GET /messages）。

5. **STOMP心跳**：WebSocket连接需要心跳保活。Spring STOMP默认心跳间隔10秒。移动端（Android APP）在网络切换时（WiFi→4G）需要重新建立WebSocket连接。

6. **MQTT重连策略**：后端MQTT客户端断线后自动重连（Paho默认支持）。重连后需要重新订阅所有Topic，避免丢失消息。

7. **Topic权限控制**：MQTT Broker（Mosquitto）需要配置ACL，限制设备只能发布自己的Topic，不能跨大棚发布数据。通过用户名+Topic模式匹配实现。

8. **WebSocket安全**：STOMP连接必须通过Token认证（在CONNECT帧的Header中传递）。未认证的连接拒绝建立。每个连接与用户身份绑定，确保消息只推送给有权限的用户。

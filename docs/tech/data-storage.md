---
id: TECH-002
title: 数据存储层技术设计文档
type: tech-design
module: data-storage
tags:
  - MySQL
  - InfluxDB
  - Chroma
  - 向量数据库
  - 时序数据库
  - 数据分层
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - /document/docs/prd/smart-greenhouse-prd.md
  - /document/docs/api/api-design.md
---

## 概述

智慧大棚AIoT系统采用三库分层架构：MySQL负责业务数据与关系存储，InfluxDB负责海量时序传感器数据，Chroma负责农业知识库向量化存储与检索。三者各司其职，通过Spring Boot后端统一访问，上层业务无需感知底层存储差异。

**设计原则**：
- 写多读少的时序数据走InfluxDB（高性能时间窗口查询）
- 强一致性业务数据走MySQL（事务、关联查询、权限校验）
- 非结构化语义检索走Chroma（向量相似度匹配）
- 大文件（图片/音频/视频）走本地文件系统，数据库只存路径

## 架构设计

### 三库分层架构

```
┌─────────────────────────────────────────────────────────────────┐
│                     Spring Boot 数据访问层                        │
│                                                                  │
│  ┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐   │
│  │ JPA/Hibernate│  │ InfluxDB Java    │  │ Spring AI        │   │
│  │ + Spring Data│  │ Client (2.x)     │  │ + Chroma Client  │   │
│  └──────┬───────┘  └────────┬─────────┘  └────────┬─────────┘   │
│         │                   │                      │             │
├─────────┼───────────────────┼──────────────────────┼─────────────┤
│         ▼                   ▼                      ▼             │
│  ┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐   │
│  │   MySQL 8.0  │  │  InfluxDB 2.7    │  │  Chroma          │   │
│  │              │  │                  │  │                  │   │
│  │ • 用户/角色   │  │ • 传感器时序数据  │  │ • 知识库向量      │   │
│  │ • 大棚/设备   │  │ • 按greenhouse/  │  │ • embedding索引   │   │
│  │ • 预警/控制   │  │   device/group   │  │ • 语义检索        │   │
│  │ • 聊天/授权   │  │   三标签索引      │  │                  │   │
│  │ • 权限矩阵    │  │ • 聚合/降采样    │  │                  │   │
│  └──────────────┘  └──────────────────┘  └──────────────────┘   │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              本地文件系统 (图片/音频/视频/截帧)              │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### MySQL表结构总览（25张表）

```
业务核心层 (10张):
  users                   # 用户表 (ADMIN/OWNER/WORKER/EXPERT)
  greenhouses             # 大棚表 (含五级地址)
  devices                 # ESP32设备表
  device_groups           # 传感器组表
  sensors                 # 传感器配置表
  actuators               # 执行器配置表
  scenes                  # 场景联动表
  knowledge_documents     # 知识库文档表
  dialect_corpus          # 方言语料表
  weather_cache           # 天气数据缓存表

权限管理层 (3张):
  employee_permissions    # 员工权限矩阵表
  user_addresses          # 用户地区地址表
  user_alert_thresholds   # 用户自定义预警阈值表

业务记录层 (7张):
  alerts                  # 预警记录表
  alert_rules             # 预警规则表
  diagnostic_records      # 病虫害诊断记录
  qa_records              # AI问答记录
  control_logs            # 设备控制日志
  growth_assessments      # 长势评估记录
  health_assessments      # 多模态健康综合评估

专家咨询层 (4张):
  chat_conversations      # 专家咨询对话表
  chat_messages           # 聊天消息表
  data_authorizations     # 环境数据授权表
  expert_availability     # 专家在线状态表

作物管理层 (1张):
  crop_cycles             # 作物生长周期表
```

### InfluxDB Measurement设计

```
measurement: sensor_data
├── tags:  (高基数索引字段)
│   ├── greenhouse_id     # 大棚ID → 按棚查询
│   ├── device_id         # 设备ID → 按设备查询
│   ├── group_id          # 传感器组ID → 按组查询/对比
│   └── sensor_type       # 传感器类型 → 按参数查询
├── fields:
│   └── value (FLOAT)     # 传感器读数
└── time (timestamp)      # 自动时间戳

设计原则:
  - tags用于WHERE/GROUP BY过滤，fields用于聚合计算
  - group_id作为tag支持按传感器组对比、聚合、区域热力图
  - sensor_type作为tag而非measurement，便于跨参数联合查询
```

### Chroma Collection设计

```
collection: agricultural_knowledge
├── documents:  知识库文档切片 (每片约500字)
├── embeddings: 1024维向量 (SiliconFlow bge-m3)
├── metadatas:
│   ├── document_id       # 关联knowledge_documents.id
│   ├── title             # 文档标题
│   ├── category          # 分类: 病害防治/栽培技术/水肥管理/...
│   ├── crop_type         # 适用作物: 番茄/黄瓜/辣椒/...
│   └── chunk_index       # 切片序号
└── ids: UUID 自动生成
```

## 数据流

### 传感器数据写入流

```
ESP32上报 → MQTT Broker → Spring Boot C4(MQTT消费)
  → 解析消息: {value, timestamp, group_id}
  → InfluxDB写入:
      WriteRecord.builder()
        .measurement("sensor_data")
        .tag("greenhouse_id", ghId)
        .tag("device_id", devId)
        .tag("group_id", groupId)
        .tag("sensor_type", sensorType)
        .field("value", value)
        .time(Instant.ofEpochSecond(timestamp), WritePrecision.S)
        .build()
  → 批量写入 (每100条或每5秒flush一次)
```

### 时序数据查询流

```
APP请求实时数据 → GET /api/v1/sensors/realtime?greenhouse_id=1
  → InfluxDB Flux查询:
      from(bucket: "sensor_data")
        |> range(start: -1m)
        |> filter(fn: (r) => r.greenhouse_id == "1")
        |> last()
        |> group(columns: ["group_id", "sensor_type"])
  → 按group_id分组 → 返回各组最新值

APP请求多组对比 → GET /api/v1/sensors/compare?greenhouse_id=1&sensor_type=TEMP
  → InfluxDB Flux查询:
      from(bucket: "sensor_data")
        |> range(start: -1h)
        |> filter(fn: (r) => r.greenhouse_id == "1" and r.sensor_type == "TEMP")
        |> aggregateWindow(every: 1m, fn: mean)
        |> group(columns: ["group_id"])
  → 返回各组温度时间序列 → 前端渲染多条折线对比

APP请求聚合数据 → GET /api/v1/sensors/aggregate?greenhouse_id=1
  → InfluxDB Flux查询:
      from(bucket: "sensor_data")
        |> range(start: -5m)
        |> filter(fn: (r) => r.greenhouse_id == "1")
        |> group(columns: ["sensor_type"])
        |> mean()
  → 返回全棚各参数平均值、最高值、最低值
```

### 知识库向量检索流

```
用户提问 → Spring AI RAG
  → 将问题通过SiliconFlow bge-m3向量化 (1024维)
  → Chroma相似度检索:
      collection.query(
        query_embeddings=[question_embedding],
        n_results=5,
        where={"crop_type": "番茄"}  # 可选元数据过滤
      )
  → 返回top-5相关文档片段
  → 拼接prompt → DeepSeek生成答案
```

### 数据归档与清理策略

```
InfluxDB:
  - 原始数据保留: 90天 (30秒采集频率，数据量大)
  - 降采样: 1小时聚合 → 保留1年
  - 降采样: 1天聚合 → 保留3年

MySQL:
  - control_logs: 保留1年，超过自动归档
  - alerts: 保留2年
  - chat_messages: 保留3年
  - 其他业务表: 永久保留

Chroma:
  - 知识库更新时全量重建 (文档数量有限，<500篇)
  - 旧Collection保留作为备份

文件系统:
  - 截帧图片: 保留90天 (每30分钟一张，量大)
  - 诊断图片: 永久保留
  - 音频文件: 永久保留 (语料价值)
```

## 接口设计概要

### 后端数据访问层接口

**InfluxDB操作接口**：
```java
public interface SensorDataRepository {
    // 写入单条传感器数据
    void writeSensorData(SensorDataPoint point);
    
    // 批量写入
    void batchWriteSensorData(List<SensorDataPoint> points);
    
    // 查询最新值 (按组过滤)
    Map<String, Map<String, Double>> queryLatest(String greenhouseId, String groupId);
    
    // 查询历史数据 (时间范围+聚合+按组)
    List<SensorDataPoint> queryHistory(String greenhouseId, String sensorType,
                                       Instant start, Instant end,
                                       String groupId, AggregationWindow window);
    
    // 多组对比查询
    Map<String, List<SensorDataPoint>> queryCompare(String greenhouseId, String sensorType,
                                                     Instant start, Instant end);
    
    // 大棚聚合查询 (平均值/最高/最低)
    AggregateResult queryAggregate(String greenhouseId, String sensorType, Instant since);
}
```

**Chroma操作接口**：
```java
public interface KnowledgeVectorRepository {
    // 向量化并存储文档
    void indexDocument(KnowledgeDocument doc, List<String> chunks);
    
    // 相似度检索
    List<DocumentMatch> similaritySearch(String query, int topK, Map<String, String> filters);
    
    // 删除文档向量
    void deleteDocument(String documentId);
    
    // 重建全量索引
    void rebuildIndex(List<KnowledgeDocument> documents);
}
```

### 数据库连接配置

```yaml
# MySQL
spring:
  datasource:
    url: jdbc:mysql://${MYSQL_HOST:localhost}:3306/smart_greenhouse
    username: ${MYSQL_USER:root}
    password: ${MYSQL_PASSWORD}
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000

# InfluxDB
influxdb:
  url: ${INFLUX_URL:http://localhost:8086}
  token: ${INFLUX_TOKEN}
  org: greenhouse
  bucket: sensor_data
  batch-size: 100
  flush-interval-ms: 5000

# Chroma
chroma:
  url: ${CHROMA_URL:http://localhost:8000}
  collection: agricultural_knowledge
  embedding:
    provider: siliconflow
    model: BAAI/bge-m3
    dimensions: 1024
```

## 关键算法/逻辑

### 时序数据降采样策略

```
原始数据: 30秒/点 → 90天保留
  → 1小时降采样 (mean/max/min):
      每60分钟窗口计算平均值、最大值、最小值
      存储为单独的measurement: sensor_data_1h
      保留1年
  → 1天降采样:
      每24小时窗口计算日平均、日最高、日最低
      存储为: sensor_data_1d
      保留3年
```

### 数据分页查询策略

MySQL使用Spring Data JPA分页：
```java
Page<Alert> findByGreenhouseIdOrderByCreatedAtDesc(Long ghId, Pageable pageable);
```

InfluxDB使用offset/limit分页：
```java
// Flux查询中使用 limit() + offset 实现分页
// 对于大数据量查询，使用时间范围限制 + 聚合窗口减少返回数据量
```

### 数据库事务管理

- MySQL: Spring `@Transactional` 注解管理事务边界
- InfluxDB: 不支持ACID事务，通过批量写入 + 失败重试保证最终一致性
- Chroma: 向量操作幂等，支持重复索引（覆盖更新）
- 跨库操作: 不使用分布式事务，通过补偿机制处理失败场景

## 技术选型理由

| 决策项 | 选择 | 理由 |
|--------|------|------|
| MySQL作为主库 | MySQL 8.0 | 成熟稳定，Spring Data JPA支持好；业务数据需要事务和关联查询 |
| InfluxDB存储时序 | InfluxDB 2.7 | 申报书要求"时序数据库"；高性能时间窗口查询；tag索引天然支持多维度过滤 |
| Chroma向量库 | Chroma | 轻量级，Docker部署简单；Python生态成熟，Spring AI通过HTTP API调用 |
| 三库分离 | 各司其职 | 时序数据不适合MySQL（写入量大、查询模式不同）；向量数据不适合关系型 |
| 本地文件系统 | 不引入OSS | 一人开发运维成本考虑；Docker volume映射到宿主机目录 |
| JPA + 原生客户端 | 混合方案 | JPA适合CRUD业务操作；InfluxDB和Chroma用原生客户端获得更好的性能和灵活性 |

## 注意事项

1. **InfluxDB写入优化**：传感器数据写入必须使用批量写入（batch write），每100条或5秒flush一次。单条写入会导致性能急剧下降。MQTT消费模块需要维护写入缓冲区。

2. **MySQL连接池**：HikariCP连接池最大连接数建议10（本地单机部署），避免连接数过多消耗资源。InfluxDB和Chroma使用HTTP API，无需连接池。

3. **外键约束策略**：所有外键禁止级联删除（CASCADE）。删除大棚时必须检查关联设备、预警规则、传感器数据，手动处理关联记录后再删除。

4. **索引策略**：chat_messages表必须建立 (conversation_id, created_at) 联合索引，聊天消息列表查询是最频繁的操作之一。alerts表建立 (greenhouse_id, created_at) 索引。

5. **Chroma持久化**：Chroma的持久化目录必须映射到Docker宿主机目录，防止容器重启丢失向量数据。在docker-compose.yml中配置volumes映射。

6. **Embedding服务选型**：DeepSeek不提供Embedding API，必须使用SiliconFlow bge-m3。注意SiliconFlow的OpenAI兼容API格式，Spring AI配置需手动创建EmbeddingModel Bean。

7. **数据备份**：MySQL每日mysqldump；InfluxDB每日influx backup；Chroma定期备份持久化目录；文件系统rsync同步备份。

8. **字段命名规范**：全部使用snake_case，表名复数形式。时间字段统一使用 created_at、updated_at（DATETIME类型），状态字段统一使用 status（TINYINT）。

# 智慧大棚AIoT系统 — AI开发规则

> 本文档面向AI开发助手（Cursor、Copilot、Qoder等），所有开发必须严格遵循本文档规则。
>
> 版本：v2.2 | 基于架构方案v3.2
>
> v2.0变更：新增WebSocket消息格式规范、AI抽象层返回值DTO定义、错误码枚举、API版本前缀(/api/v1/)、Spring AI版本锁定(1.0.9)、单元测试规范、Git提交规范、环境变量默认值策略、数据库表数量统一说明
>
> v2.1变更：新增"AI助手工作规则"章节，明确任务执行前必须先列计划并获得用户同意
>
> v2.2变更：新增"日志更新工作流"章节（0.4），强制要求每次完成后更新4份日志（DEVLOG + INDEX + TASK模块日志 + 受影响模块日志），明确开发前必读文档顺序

---

## 0. AI助手工作规则（最高优先级）

> **本章节是AI助手的行为准则，优先级高于所有其他章节。AI助手在每次对话中必须严格遵守。**

### 0.1 任务执行流程

```
用户提出需求
    │
    ▼
AI分析需求，列出执行计划（步骤清单）
    │
    ▼
等待用户确认/修改计划
    │
    ├── 用户不同意 → 修改计划，重新确认
    │
    ▼
用户同意后，AI开始逐步执行
    │
    ▼
每完成一步，向用户汇报进度和结果
    │
    ▼
全部完成后，自检成果，向用户交付
```

### 0.2 计划格式要求

每次执行任务前，AI必须以表格或清单形式列出：

1. **要做什么**（任务分解，每步一句话说清楚）
2. **产出什么**（每步的交付物是什么）
3. **涉及哪些文件**（新建/修改哪些文件）
4. **预估影响范围**（会不会影响已有功能）

### 0.3 执行中的沟通规则

| 规则 | 说明 |
|------|------|
| **先计划后执行** | 任何非简单问答的开发任务，必须先列计划获得同意 |
| **有疑问停下来** | 遇到不确定的技术选型、业务逻辑，必须询问用户，不能猜 |
| **告知优缺点** | 提供多个方案时，必须说明每个方案的优缺点 |
| **自检再交付** | 完成任务后，自己先检查一遍有没有遗漏或错误 |
| **进度透明** | 每一步完成都要告知用户当前状态 |

### 0.4 日志更新工作流（强制执行）

> **AI 每次完成一个步骤/模块的开发后，必须执行以下日志更新流程，缺一不可。**

#### 需要更新的文件（4 份）

| 序号 | 文件 | 位置 | 说明 |
|------|------|------|------|
| 1 | **DEVLOG.md** | `Smart_Greenhouse_Project/devlog/DEVLOG.md` | 项目总开发日志，追加步骤条目 |
| 2 | **INDEX.md** | `document/devlog/INDEX.md` | 任务总表，更新任务状态和统计 |
| 3 | **TASK-xxx.md** | `document/devlog/TASK-xxx.md` | 对应模块独立日志，记录关键修改 |
| 4 | **受影响模块日志** | `document/devlog/TASK-yyy.md` | 如有跨模块修改，同步更新被影响模块的日志 |

#### 日志更新内容要求

**DEVLOG.md（项目总日志）**：
- 步骤编号 + 标题 + 完成状态
- 操作时间
- 每个文件的简要说明
- 整体结果总结
- 完整文件清单

**INDEX.md（任务总表）**：
- 将已完成任务的状态从 `planned` 更新为 `✅ completed`
- 填写开始时间、完成时间、负责人
- 关联文档列填写对应 TASK 日志链接
- 更新统计概览数字

**TASK-xxx.md（模块独立日志）**：
- 基于 `TASK-TEMPLATE.md` 模板创建
- YAML 头部：id/title/module/status/completed 等
- 关键修改：文件路径 + 修改类型 + 变更说明
- 完成结果：一句话总结
- 阻塞记录：有则填写，无则写"无阻塞"

#### 日志更新检查清单

- [ ] DEVLOG.md 已追加新步骤
- [ ] INDEX.md 任务状态已更新
- [ ] INDEX.md 统计数字已更新
- [ ] TASK-xxx.md 模块日志已创建/更新
- [ ] 项目仓库已 commit + push
- [ ] 文档仓库已 commit + push

#### 开发前必读

每次开始新任务前，按以下顺序读取：
1. `AI开发规则文档_v2.0.md`（本文档）
2. `document/INDEX.md` — 项目文档总索引，了解整体结构
3. `document/devlog/INDEX.md` — 任务总表，查看进度
4. 对应的 `document/devlog/TASK-xxx.md` — 模块历史，了解之前做了什么

### 0.5 不需要列计划的情况

以下情况可以直接执行，不需要列计划：
- 回答简单问题（如"这个是什么意思"）
- 读取/查看文件内容
- 纯信息查询（如查版本号、查文档）
- 用户明确说"直接做，不用列计划"

---

## 1. 项目概述

本项目为大学生创新创业训练计划项目"基于多模态融合的智慧大棚作物诊疗与智能管控APP"，研发一款AIoT智慧大棚管理APP，融合环境数值、作物图像、方言语音多模态数据，实现环境趋势智能预警、生长状态评估、病害自主诊断与设备远程联动控制。核心闭环：感知（传感器+摄像头）→ 分析（AI+知识库+多模态融合）→ 决策（预警+处方）→ 控制（设备联动）→ 兜底（专家咨询）。

### 系统角色及权限范围

| 角色 | 英文标识 | 可用端 | 权限范围 |
|------|----------|--------|----------|
| 管理员 | ADMIN | 仅Web端 | 全部大棚、全部功能、管理用户/账号/平台总览 |
| 棚主 | OWNER | APP端+Web端 | 自己的大棚、全部功能、管理名下员工 |
| 员工 | WORKER | 仅APP端 | 被授权的大棚+被授权的功能（由棚主分配，仅归属一个棚主） |
| 专家 | EXPERT | 仅Web端 | 被授权的大棚（7天有效期）、查看数据+对话功能（由用户授权） |

**关键规则**：
- APP端只有棚主(OWNER)和员工(WORKER)两种角色，**不存在管理员**
- Web端有管理员(ADMIN)、棚主(OWNER)、专家(EXPERT)三种角色，**不存在员工**
- 员工只能归属**一个**棚主，不存在切换棚主的情况
- 权限模型 = RBAC角色 + 大棚授权(数据隔离) + 功能授权(操作粒度)

---

## 2. 技术栈硬性约束

| 层级 | 技术选型 | 版本 | 禁止替换 |
|------|----------|------|----------|
| **后端框架** | Spring Boot | 3.3.x (Java 17+) | ✅ 禁止替换 |
| **AI框架** | Spring AI | 1.0.9 | ✅ 禁止替换 |
| **APP端** | Java Android原生 (MVVM + Jetpack) | target SDK 34 | ✅ 禁止替换 |
| **Web管理端** | Vue 3 + Element Plus | Vue 3.4+, Element Plus 2.x | ✅ 禁止替换 |
| **关系数据库** | MySQL | 8.0 | ✅ 禁止替换 |
| **时序数据库** | InfluxDB | 2.7 | ✅ 禁止替换 |
| **向量数据库** | Chroma | latest | ✅ 禁止替换 |
| **MQTT Broker** | Eclipse Mosquitto | 2.x | ✅ 禁止替换 |
| **实时通信** | WebSocket (STOMP协议) | Spring原生 | ✅ 禁止替换 |
| **LLM对话** | DeepSeek API (deepseek-chat) | OpenAI兼容 | ✅ 禁止替换 |
| **Embedding向量化** | 硅基流动(SiliconFlow) bge-m3 | OpenAI兼容格式 | ✅ 禁止替换 |
| **图像识别(初期)** | 百度AI开放平台 | REST API | ✅ 禁止替换 |
| **语音识别(初期)** | 讯飞开放平台 | REST API | ✅ 禁止替换 |
| **天气API** | 和风天气 | 免费版 | ✅ 禁止替换 |
| **硬件主控** | ESP32 | ESP32-DevKitC V4 | ✅ 禁止替换 |
| **摄像头** | ESP32-CAM | AI-Thinker | ✅ 禁止替换 |
| **视频截帧** | FFmpeg | 6.x | ✅ 禁止替换 |
| **内网穿透** | frp | latest | ✅ 禁止替换 |
| **部署方式** | Docker Compose | 本地Linux | ✅ 禁止替换 |
| **版本管理** | Git + GitHub | 2.x | ✅ 禁止替换 |
| **构建工具(前端)** | Vite | 5.x | ✅ 禁止替换 |
| **MQTT客户端(后端)** | Eclipse Paho | 1.2.x | ✅ 禁止替换 |
| **JWT** | jjwt | 0.12.x | ✅ 禁止替换 |

**严禁引入Python后端服务**。所有后端逻辑统一使用Java/Spring Boot，AI能力全部通过API调用。

---

## 3. 项目结构规范

### 3.1 后端项目目录结构 (Spring Boot)

```
smart-greenhouse/
├── src/main/java/com/greenhouse/
│   ├── SmartGreenhouseApplication.java          # 启动类
│   ├── config/                                    # 配置类
│   │   ├── SecurityConfig.java                   # Spring Security配置
│   │   ├── WebSocketConfig.java                  # WebSocket(STOMP)配置
│   │   ├── MqttConfig.java                       # MQTT连接配置
│   │   ├── InfluxDbConfig.java                   # InfluxDB配置
│   │   ├── ChromaConfig.java                     # Chroma向量库配置
│   │   ├── AiConfig.java                         # AI模型抽象层配置
│   │   └── SwaggerConfig.java                    # API文档配置
│   ├── security/                                  # 安全模块
│   │   ├── JwtTokenProvider.java                 # JWT生成/验证
│   │   ├── JwtAuthenticationFilter.java          # JWT过滤器
│   │   ├── UserDetailsServiceImpl.java           # 用户认证实现
│   │   └── annotations/                          # 自定义权限注解
│   │       ├── RequireGreenhouseAccess.java      # 大棚授权校验
│   │       └── RequireFunction.java              # 功能权限校验
│   ├── module/                                    # 业务模块
│   │   ├── auth/                                 # C1 用户认证模块
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   └── dto/
│   │   ├── device/                               # C2 设备管理 + C19 设备分组
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   └── dto/
│   │   ├── greenhouse/                           # C3 大棚管理
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   └── dto/
│   │   ├── mqtt/                                 # C4 MQTT消费模块
│   │   │   ├── MqttSubscriber.java
│   │   │   └── service/
│   │   ├── sensor/                               # C5 时序数据服务
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   └── dto/
│   │   ├── alert/                                # C6 预警引擎 + C21 自定义阈值
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   └── dto/
│   │   ├── control/                              # C7 设备控制 + C12 场景联动
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   └── dto/
│   │   ├── diagnosis/                            # C8 图片诊断模块
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   └── dto/
│   │   ├── qa/                                   # C9 RAG问答模块 + C10 语音识别
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   └── dto/
│   │   ├── websocket/                            # C11 WebSocket推送
│   │   │   ├── handler/
│   │   │   ├── service/
│   │   │   └── dto/
│   │   ├── file/                                 # C13 文件存储模块
│   │   │   └── service/
│   │   ├── weather/                              # C14 天气API对接
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   └── dto/
│   │   ├── health/                                # C15 多模态融合分析
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   └── dto/
│   │   ├── chat/                                  # C16 实时聊天
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   └── dto/
│   │   ├── expert/                                # C17 专家授权 + 专家管理
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   └── dto/
│   │   ├── permission/                           # C18 多角色权限模块
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   └── dto/
│   │   ├── crop/                                 # C22 作物生长周期管理
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   └── dto/
│   │   ├── knowledge/                            # 知识库管理（Phase 4）
│   │   │   ├── controller/
│   │   │   └── service/
│   │   ├── corpus/                               # 方言语料管理（Phase 4）
│   │   │   ├── controller/
│   │   │   └── service/
│   │   └── admin/                                # 管理员功能（Phase 4）
│   │       ├── controller/
│   │       └── service/
│   ├── ai/                                       # AI模型抽象层
│   │   ├── DiseaseRecognitionProvider.java       # 图像识别接口
│   │   ├── SpeechRecognitionProvider.java        # 语音识别接口
│   │   ├── baidu/                                # 百度AI实现(初期)
│   │   │   └── BaiduRecognitionProvider.java
│   │   ├── resnet/                               # ResNet实现(后期)
│   │   │   └── ResNetRecognitionProvider.java
│   │   ├── xunfei/                               # 讯飞ASR实现(初期)
│   │   │   └── XunfeiSpeechProvider.java
│   │   └── whisper/                              # Whisper实现(后期)
│   │       └── WhisperSpeechProvider.java
│   ├── entity/                                   # JPA实体
│   ├── repository/                               # JPA仓库(通用)
│   ├── exception/                                # 全局异常处理
│   │   └── GlobalExceptionHandler.java
├── src/main/resources/
│   ├── application.yml                           # 主配置
│   ├── application-dev.yml                       # 开发环境
│   └── application-prod.yml                      # 生产环境
├── src/test/java/                                # 测试
├── docker-compose.yml                            # Docker编排
├── Dockerfile                                    # 后端镜像
├── nginx.conf                                    # Nginx配置
├── frpc.toml                                     # frp客户端配置
├── mosquitto.conf                                # MQTT Broker配置
└── pom.xml                                       # Maven配置
```

> **关于公共类的说明**：`ApiResponse.java`、`PageResult.java`、`BusinessException.java`、`ErrorCode.java` 等通用类位于独立的 Maven `common` 模块（`common/src/main/java/com/greenhouse/common/`），backend 模块通过 Maven 依赖引用。模块内部的 DTO 按模块内聚原则放在各 `module/{name}/dto/` 子目录下。

### 3.2 APP端目录结构 (Android)

```
app/src/main/java/com/greenhouse/app/
├── data/                                       # 数据层
│   ├── api/                                    # Retrofit接口
│   ├── model/                                  # 数据模型
│   ├── repository/                             # 数据仓库
│   └── local/                                  # Room本地缓存
├── ui/                                         # UI层
│   ├── login/                                   # 登录页
│   ├── common/                                  # 公共UI（MainActivity等）
│   ├── dashboard/                               # 实时数据看板
│   ├── alert/                                   # 环境预警中心
│   ├── diagnosis/                               # 病虫害诊断
│   ├── growth/                                 # 作物长势评估
│   ├── qa/                                     # AI智能问答
│   ├── control/                                # 设备控制
│   ├── expert/                                 # 专家咨询
│   └── profile/                                # 个人中心
├── viewmodel/                                  # ViewModel层
├── websocket/                                  # WebSocket(STOMP)通信
├── adapter/                                    # RecyclerView适配器
└── util/                                       # 工具类
```

### 3.3 Web管理端目录结构 (Vue 3)

```
web-admin/src/
├── api/                                        # API请求封装
├── views/                                      # 页面组件
│   ├── dashboard/                              # 数据总览大屏
│   ├── device/                                 # 设备管理
│   ├── user/                                   # 用户与角色管理
│   ├── knowledge/                              # 知识库管理
│   ├── alert/                                  # 预警规则配置
│   ├── export/                                 # 数据导出与报表
│   ├── expert/                                 # 专家工作台
│   ├── owner/                                  # 棚主Web管理
│   ├── monitor/                                # 系统监控
│   └── corpus/                                 # 方言语料管理
├── components/                                 # 公共组件
├── router/                                     # 路由配置
├── stores/                                     # Pinia状态管理
├── utils/                                      # 工具函数
└── assets/                                     # 静态资源
```

### 3.4 ESP32固件目录结构

```
esp32-firmware/
├── src/
│   ├── main.cpp                                # 主程序
│   ├── sensors/                                # 传感器驱动
│   │   ├── SHT30Driver.cpp
│   │   ├── BH1750Driver.cpp
│   │   ├── MQ135Driver.cpp
│   │   ├── O2SensorDriver.cpp
│   │   ├── DS18B20Driver.cpp
│   │   ├── SoilMoistureDriver.cpp
│   │   ├── ECDriver.cpp
│   │   └── NPKDriver.cpp
│   ├── mqtt/                                   # MQTT通信
│   ├── actuator/                               # 继电器控制
│   ├── config/                                 # 配置(WiFi/MQTT/GroupID)
│   └── watchdog/                               # 看门狗
├── include/
├── lib/
└── platformio.ini
```

---

## 4. 数据库开发规范

### 4.1 表命名规范

- 必须使用**小写字母+下划线**命名（snake_case）
- 必须使用**复数形式**（如 `users`、`greenhouses`、`devices`）
- 禁止使用大写字母或驼峰命名
- 禁止使用MySQL保留字

### 4.2 字段命名规范

- 必须使用**小写字母+下划线**命名（snake_case）
- 主键统一命名 `id`（BIGINT, AUTO_INCREMENT）
- 外键命名格式：`{关联表单数}_id`（如 `user_id`、`greenhouse_id`、`group_id`）
- 时间字段：`created_at`、`updated_at`（DATETIME类型）
- 状态字段：`status`（TINYINT类型，0=禁用/离线/关，1=正常/在线/开）

### 4.3 MySQL全部表清单及核心字段（共25张表）

> **说明**：以下25张表中，前20张为核心业务表（与架构方案文档一致），后5张（sensors、actuators、user_addresses、employee_permissions、expert_availability）为辅助拆分表，用于规范化数据存储。所有表统一在MySQL中管理。

| # | 表名 | 用途 | 核心字段 |
|---|------|------|----------|
| 1 | `users` | 用户表 | id, username, password(BCrypt), phone, role(ENUM:ADMIN/OWNER/WORKER/EXPERT), real_name, expert_specialty, expert_status, owner_id(FK→users.id,员工单归属), status, created_at |
| 2 | `user_addresses` | 用户地区地址 | id, user_id(FK), province, city, district, town, village, detail, is_primary, created_at |
| 3 | `greenhouses` | 大棚表 | id, name, location, crop_type, user_id(FK→users.id,棚主), province/city/district/town/village(五级地址), status, created_at |
| 4 | `device_groups` | 传感器组表 | id, greenhouse_id(FK), group_name, zone_label(区域标注), description, sort_order, status, created_at |
| 5 | `devices` | ESP32设备表 | id, device_code(UNIQUE), greenhouse_id(FK), group_id(FK→device_groups.id,可空), name, wifi_ssid, last_heartbeat, status, created_at |
| 6 | `sensors` | 传感器配置表 | id, device_id(FK), sensor_type(ENUM:TEMP/HUMIDITY/LIGHT/CO2/O2/SOIL_TEMP/SOIL_HUMIDITY/EC/N/P/K), i2c_address, min_threshold, max_threshold, status |
| 7 | `actuators` | 执行器配置表 | id, device_id(FK), name, actuator_type(ENUM:FAN/CURTAIN/SHADE_NET/IRRIGATION/LIGHT), gpio_pin, current_state, status |
| 8 | `employee_permissions` | 员工权限表 | id, employee_id(FK), owner_id(FK), greenhouse_id(FK), can_view_data, can_control_device, can_diagnose, can_ask_expert, can_view_alerts, can_view_history, created_at, updated_at |
| 9 | `alert_rules` | 预警规则表 | id, greenhouse_id(FK), group_id(FK,可空), sensor_type, rule_type(ENUM:THRESHOLD/TREND/COMPOSITE/WEATHER), condition_json(JSON), alert_level(ENUM:INFO/WARNING/CRITICAL), scene_id(FK,可空), enabled |
| 10 | `alerts` | 预警记录表 | id, greenhouse_id(FK), group_id(FK,可空), alert_rule_id(FK), level(ENUM), title, content, sensor_value, weather_info, read_status, created_at |
| 11 | `scenes` | 场景联动表 | id, name, trigger_condition(JSON), actions_json(JSON), greenhouse_id(FK), enabled |
| 12 | `diagnostic_records` | 病虫害诊断记录 | id, user_id(FK), greenhouse_id(FK), image_path, disease_name, confidence, treatment, recognition_engine, expert_consulted, created_at |
| 13 | `qa_records` | 问答记录 | id, user_id(FK), question, answer, input_type(ENUM:TEXT/VOICE), asr_engine, sources(JSON), created_at |
| 14 | `control_logs` | 设备控制日志 | id, user_id(FK,可空), actuator_id(FK), action(ENUM:ON/OFF/SCENE), source(ENUM:MANUAL/SCENE/ALERT), scene_id(FK,可空), group_id(FK,可空), created_at |
| 15 | `knowledge_documents` | 知识库文档表 | id, title, category, crop_type, content(LONGTEXT), file_path, chunk_count, vector_indexed, created_at |
| 16 | `dialect_corpus` | 方言语料表 | id, audio_path, dialect_text, mandarin_text, category, duration_sec, usage_type(ENUM:TEST/TRAIN/BOTH), created_at |
| 17 | `growth_assessments` | 长势评估记录 | id, greenhouse_id(FK), crop_cycle_id(FK→crop_cycles.id,可空), image_path, growth_stage, plant_height, leaf_area, leaf_color, health_score, created_at |
| 18 | `health_assessments` | 多模态健康综合评估 | id, greenhouse_id(FK), env_score, visual_score, weather_risk, overall_score, analysis_json(JSON), recommendations, created_at |
| 19 | `weather_cache` | 天气数据缓存 | id, location, forecast_json(JSON), temperature, humidity, weather_code, wind_speed, forecast_time, updated_at |
| 20 | `crop_cycles` | 作物生长周期表 | id, greenhouse_id(FK), crop_type, variety, planting_date, expected_harvest_date, actual_harvest_date, current_stage, stage_source(ENUM:AUTO/MANUAL), status(ENUM:ACTIVE/COMPLETED/CANCELLED), notes, created_at, updated_at |
| 21 | `chat_conversations` | 专家咨询对话表 | id, user_id(FK), expert_id(FK), greenhouse_id(FK), subject, status(ENUM:WAITING/ACTIVE/CLOSED), diagnostic_id(FK,可空), created_at, closed_at |
| 22 | `chat_messages` | 聊天消息表 | id, conversation_id(FK), sender_id(FK), sender_type(ENUM:USER/EXPERT), message_type(ENUM:TEXT/IMAGE/VIDEO/ENV_SNAPSHOT), content, file_path, snapshot_data(JSON), read_status, created_at; INDEX: idx_conversation_created(conversation_id, created_at) |
| 23 | `data_authorizations` | 环境数据授权表 | id, expert_id(FK), user_id(FK), greenhouse_id(FK), status(ENUM:PENDING/APPROVED/REJECTED/EXPIRED/REVOKED), requested_at, approved_at, expires_at(7天), revoked_at, revoked_by(FK), reason |
| 24 | `expert_availability` | 专家在线状态表 | id, expert_id(FK), is_online, last_active_at, max_concurrent(默认5), current_count |
| 25 | `user_alert_thresholds` | 用户自定义预警阈值 | id, user_id(FK), greenhouse_id(FK), group_id(FK,可空), sensor_type(ENUM), min_threshold, max_threshold, enabled, created_at, updated_at |

### 4.4 InfluxDB Measurement规范

```
measurement: sensor_data
├── tags:
│   ├── greenhouse_id    # 大棚ID
│   ├── device_id        # 设备ID
│   ├── group_id         # 传感器组ID
│   └── sensor_type      # TEMP/HUMIDITY/LIGHT/CO2/O2/SOIL_TEMP/SOIL_HUMIDITY/EC/N/P/K
├── fields:
│   └── value (FLOAT)    # 传感器读数
└── time (timestamp)     # 自动
```

### 4.5 Chroma Collection规范

```
collection: agricultural_knowledge
├── documents: 知识库文档切片(每片500字左右)
├── embeddings: Spring AI自动生成(SiliconFlow bge-m3, 1024维)
├── metadatas:
│   ├── document_id
│   ├── title
│   ├── category
│   ├── crop_type
│   └── chunk_index
└── ids: UUID自动生成
```

### 4.6 外键关系约束

- 所有外键必须在建表时明确定义
- 外键关联删除策略：禁止级联删除（CASCADE），必须使用逻辑删除或手动处理关联数据
- `users.owner_id`：员工单归属，只指向一个棚主ID

### 4.7 数据库禁止事项

- **禁止**直接删除表或字段，必须通过迁移脚本（Flyway/Liquibase）
- **禁止**在代码中直接执行DDL语句
- **禁止**修改已有字段类型或名称（只能通过迁移脚本新增字段并废弃旧字段）
- **禁止**跳过外键约束直接插入数据
- **禁止**在生产环境手动修改数据库数据

---

## 5. API开发规范

### 5.1 URL路径命名规范

- 所有API路径必须带版本前缀：**`/api/v1/`**
- 必须使用RESTful风格
- 路径全部使用**小写字母+连字符**（kebab-case），如 `/api/v1/crop-cycles`
- 资源名使用**复数形式**，如 `/api/v1/greenhouses`、`/api/v1/devices`
- 禁止在URL中使用动词（CRUD操作通过HTTP方法区分）
- 嵌套资源最多两层，如 `/api/v1/chat/conversations/{id}/messages`

### 5.2 统一请求/响应格式

**统一响应JSON结构**：
```json
{
  "code": 200,
  "message": "success",
  "data": { ... }
}
```

**错误响应**：
```json
{
  "code": 401,
  "message": "未授权访问",
  "data": null
}
```

**统一分页响应**：
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "list": [ ... ],
    "total": 100,
    "page": 1,
    "size": 20
  }
}
```

### 5.3 认证方式

- 必须使用**JWT Token**认证
- Token通过 `Authorization: Bearer <token>` 请求头传递
- 登录接口 `POST /api/v1/auth/login` 返回JWT
- 注册接口 `POST /api/v1/auth/register` 无需认证
- WebSocket连接使用STOMP Header传递Token

### 5.4 错误码规范

#### HTTP状态码

| 错误码 | 含义 | 说明 |
|--------|------|------|
| 200 | 成功 | 请求成功 |
| 400 | 参数错误 | 请求参数不合法 |
| 401 | 未认证 | 未登录或Token过期 |
| 403 | 无权限 | 角色/大棚/功能权限不足 |
| 404 | 资源不存在 | 请求的资源未找到 |
| 409 | 冲突 | 如用户名已存在 |
| 500 | 服务器内部错误 | 未预期的异常 |

#### 业务错误码枚举 (ErrorCode.java)

所有业务异常统一使用 `BusinessException(ErrorCode)` 抛出，ErrorCode枚举定义如下：

```java
package com.greenhouse.exception;

import lombok.Getter;
import org.springframework.http.HttpStatus;

@Getter
public enum ErrorCode {

    // ===== 通用错误 (1xxx) =====
    SUCCESS(0, "操作成功", HttpStatus.OK),
    PARAM_ERROR(1001, "参数错误", HttpStatus.BAD_REQUEST),
    RESOURCE_NOT_FOUND(1002, "资源不存在", HttpStatus.NOT_FOUND),
    INTERNAL_ERROR(1003, "服务器内部错误", HttpStatus.INTERNAL_SERVER_ERROR),

    // ===== 认证错误 (2xxx) =====
    UNAUTHORIZED(2001, "未登录或Token已过期", HttpStatus.UNAUTHORIZED),
    TOKEN_INVALID(2002, "Token无效", HttpStatus.UNAUTHORIZED),
    USERNAME_EXISTS(2003, "用户名已存在", HttpStatus.CONFLICT),
    PHONE_EXISTS(2004, "手机号已注册", HttpStatus.CONFLICT),
    LOGIN_FAILED(2005, "用户名或密码错误", HttpStatus.UNAUTHORIZED),

    // ===== 权限错误 (3xxx) =====
    ACCESS_DENIED(3001, "无访问权限", HttpStatus.FORBIDDEN),
    GREENHOUSE_ACCESS_DENIED(3002, "无该大棚访问权限", HttpStatus.FORBIDDEN),
    FUNCTION_DENIED(3003, "无该功能使用权限", HttpStatus.FORBIDDEN),
    NOT_OWNER(3004, "仅棚主可执行此操作", HttpStatus.FORBIDDEN),
    EMPLOYEE_LIMIT_EXCEEDED(3005, "员工数量已达上限", HttpStatus.FORBIDDEN),

    // ===== 大棚相关 (4xxx) =====
    GREENHOUSE_NOT_FOUND(4001, "大棚不存在", HttpStatus.NOT_FOUND),
    GREENHOUSE_LIMIT_EXCEEDED(4002, "大棚数量已达上限", HttpStatus.BAD_REQUEST),
    DEVICE_NOT_FOUND(4003, "设备不存在", HttpStatus.NOT_FOUND),
    DEVICE_OFFLINE(4004, "设备已离线", HttpStatus.BAD_REQUEST),
    DEVICE_GROUP_NOT_FOUND(4005, "传感器组不存在", HttpStatus.NOT_FOUND),

    // ===== AI相关 (5xxx) =====
    AI_RECOGNITION_FAILED(5001, "图像识别失败，请重试", HttpStatus.INTERNAL_SERVER_ERROR),
    AI_SPEECH_FAILED(5002, "语音识别失败，请尝试文字输入", HttpStatus.INTERNAL_SERVER_ERROR),
    AI_LLM_FAILED(5003, "AI服务暂时不可用", HttpStatus.INTERNAL_SERVER_ERROR),
    AI_EMBEDDING_FAILED(5004, "向量化服务异常", HttpStatus.INTERNAL_SERVER_ERROR),

    // ===== 专家咨询 (6xxx) =====
    EXPERT_NOT_FOUND(6001, "专家不存在", HttpStatus.NOT_FOUND),
    EXPERT_OFFLINE(6002, "专家当前不在线", HttpStatus.BAD_REQUEST),
    CONVERSATION_NOT_FOUND(6003, "对话不存在", HttpStatus.NOT_FOUND),
    CONVERSATION_CLOSED(6004, "对话已结束", HttpStatus.BAD_REQUEST),
    AUTHORIZATION_NOT_FOUND(6005, "授权记录不存在", HttpStatus.NOT_FOUND),
    AUTHORIZATION_EXPIRED(6006, "授权已过期", HttpStatus.FORBIDDEN),
    AUTHORIZATION_ALREADY_EXISTS(6007, "已有有效授权，无需重复申请", HttpStatus.CONFLICT),

    // ===== 作物生长周期 (7xxx) =====
    CROP_CYCLE_NOT_FOUND(7001, "生长周期记录不存在", HttpStatus.NOT_FOUND),
    CROP_CYCLE_ALREADY_COMPLETED(7002, "该生长周期已结束", HttpStatus.BAD_REQUEST),
    CROP_CYCLE_DUPLICATE(7003, "该大棚已有进行中的生长周期", HttpStatus.CONFLICT),

    // ===== 文件相关 (8xxx) =====
    FILE_UPLOAD_FAILED(8001, "文件上传失败", HttpStatus.INTERNAL_SERVER_ERROR),
    FILE_TOO_LARGE(8002, "文件大小超过限制", HttpStatus.BAD_REQUEST),
    FILE_TYPE_NOT_SUPPORTED(8003, "不支持的文件类型", HttpStatus.BAD_REQUEST);

    private final int code;
    private final String message;
    private final HttpStatus httpStatus;

    ErrorCode(int code, String message, HttpStatus httpStatus) {
        this.code = code;
        this.message = message;
        this.httpStatus = httpStatus;
    }
}
```

**使用方式**：
```java
// 抛出业务异常
throw new BusinessException(ErrorCode.GREENHOUSE_ACCESS_DENIED);

// BusinessException 定义
public class BusinessException extends RuntimeException {
    private final ErrorCode errorCode;
    
    public BusinessException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }
}

// 全局异常处理器统一处理
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(BusinessException.class)
    public ApiResponse<?> handleBusinessException(BusinessException e) {
        ErrorCode ec = e.getErrorCode();
        return ApiResponse.error(ec.getCode(), ec.getMessage());
    }
}
```

### 5.5 全部API端点清单

#### 认证模块
| 方法 | 端点 | 说明 |
|------|------|------|
| POST | /api/v1/auth/register | 注册(角色:OWNER/WORKER, APP端无ADMIN) |
| POST | /api/v1/auth/login | 登录→返回JWT |
| GET | /api/v1/auth/profile | 当前用户信息(含角色+权限) |

#### 大棚管理
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/greenhouses | 大棚列表(按角色过滤) |
| POST | /api/v1/greenhouses | 创建大棚(OWNER) |
| PUT | /api/v1/greenhouses/{id} | 更新大棚 |
| DELETE | /api/v1/greenhouses/{id} | 删除大棚 |
| GET | /api/v1/greenhouses/regions | 地区分布统计(ADMIN) |

#### 设备管理(含分组)
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/devices | 设备列表 |
| POST | /api/v1/devices | 注册设备 |
| GET | /api/v1/devices/{id}/status | 设备状态 |
| POST | /api/v1/devices/{id}/sensors | 配置传感器 |
| POST | /api/v1/devices/{id}/actuators | 配置执行器 |
| GET | /api/v1/device-groups | 传感器组列表 |
| POST | /api/v1/device-groups | 创建传感器组 |
| PUT | /api/v1/device-groups/{id} | 更新传感器组(区域标注) |
| DELETE | /api/v1/device-groups/{id} | 删除传感器组 |

#### 传感器数据(含多组对比)
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/sensors/realtime | 实时数据(可按组过滤) |
| GET | /api/v1/sensors/history | 历史数据(时间范围/聚合/按组) |
| GET | /api/v1/sensors/export | 导出CSV |
| GET | /api/v1/sensors/compare | 多组数据对比 |
| GET | /api/v1/sensors/aggregate | 大棚聚合数据(平均/最高最低) |

#### 预警(含自定义阈值)
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/alerts | 预警列表(分页/筛选/含组定位) |
| PUT | /api/v1/alerts/{id}/read | 标记已读 |
| GET | /api/v1/alerts/rules | 预警规则列表 |
| POST | /api/v1/alerts/rules | 创建预警规则 |
| PUT | /api/v1/alerts/rules/{id} | 更新规则 |
| GET | /api/v1/alerts/thresholds | 用户自定义预警阈值列表 |
| POST | /api/v1/alerts/thresholds | 设置自定义预警阈值 |
| PUT | /api/v1/alerts/thresholds/{id} | 更新自定义预警阈值 |
| DELETE | /api/v1/alerts/thresholds/{id} | 删除自定义预警阈值 |

#### 设备控制
| 方法 | 端点 | 说明 |
|------|------|------|
| POST | /api/v1/control/actuator | 控制单个设备 |
| GET | /api/v1/control/scenes | 场景列表 |
| POST | /api/v1/control/scenes | 创建场景 |
| POST | /api/v1/control/scenes/{id}/execute | 执行场景 |

#### 病虫害诊断
| 方法 | 端点 | 说明 |
|------|------|------|
| POST | /api/v1/diagnosis/recognize | 上传图片→识别(multipart) |
| GET | /api/v1/diagnosis/records | 诊断历史 |

#### AI问答
| 方法 | 端点 | 说明 |
|------|------|------|
| POST | /api/v1/qa/ask | 文字问答 |
| POST | /api/v1/qa/ask/voice | 语音问答(multipart) |
| GET | /api/v1/qa/records | 问答历史 |

#### 长势评估
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/growth/latest | 最新长势评估 |
| GET | /api/v1/growth/history | 长势历史 |
| GET | /api/v1/growth/images | 截帧图片列表 |

#### 作物生长周期
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/crop-cycles | 生长周期列表(按大棚过滤) |
| POST | /api/v1/crop-cycles | 创建种植记录 |
| GET | /api/v1/crop-cycles/{id} | 周期详情 |
| PUT | /api/v1/crop-cycles/{id} | 更新(推进阶段等) |
| PATCH | /api/v1/crop-cycles/{id}/complete | 标记完成(收获) |
| GET | /api/v1/crop-cycles/{id}/timeline | 生长时间线 |

#### 健康综合评估
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/health/score | 当前综合健康评分 |
| GET | /api/v1/health/history | 健康评分历史 |
| GET | /api/v1/health/detail/{id} | 详细评估报告 |

#### 天气信息
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/weather/current | 当前天气 |
| GET | /api/v1/weather/forecast | 天气预报 |

#### 专家咨询
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/experts | 专家列表 |
| POST | /api/v1/chat/conversations | 创建对话(发起求助) |
| GET | /api/v1/chat/conversations | 对话列表 |
| GET | /api/v1/chat/conversations/{id}/messages | 对话消息历史(分页) |
| POST | /api/v1/chat/messages | 发送消息(REST备用) |
| POST | /api/v1/chat/snapshot | 发送环境快照 |
| PUT | /api/v1/chat/conversations/{id}/close | 关闭对话 |
| GET | /api/v1/chat/unread | 未读消息数 |

#### 专家授权
| 方法 | 端点 | 说明 |
|------|------|------|
| POST | /api/v1/expert/authorize/request | 专家发起授权请求 |
| GET | /api/v1/expert/authorize/pending | 用户查看待处理请求 |
| PUT | /api/v1/expert/authorize/{id}/approve | 用户同意授权 |
| PUT | /api/v1/expert/authorize/{id}/reject | 用户拒绝授权 |
| PUT | /api/v1/expert/authorize/{id}/revoke | 用户撤销授权 |
| GET | /api/v1/expert/authorize/active | 查看有效授权列表 |
| GET | /api/v1/expert/authorize/history | 授权历史 |
| GET | /api/v1/expert/greenhouses | 专家查看已授权的大棚数据 |

#### 员工管理(棚主)
| 方法 | 端点 | 说明 |
|------|------|------|
| POST | /api/v1/owner/employees | 创建/邀请员工 |
| GET | /api/v1/owner/employees | 员工列表 |
| PUT | /api/v1/owner/employees/{id} | 更新员工信息 |
| DELETE | /api/v1/owner/employees/{id} | 删除员工 |
| PUT | /api/v1/owner/employees/{id}/permissions | 分配员工权限 |
| GET | /api/v1/owner/employees/{id}/permissions | 查看员工权限 |

#### 员工权限(员工端)
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/worker/permissions | 查看自己的权限 |
| GET | /api/v1/worker/greenhouses | 可访问的大棚列表 |

#### 专家工作台(Web端)
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/expert/consultations | 求助列表 |
| GET | /api/v1/expert/statistics | 咨询统计 |
| PUT | /api/v1/expert/status | 更新在线状态 |
| GET | /api/v1/expert/greenhouses/{id}/data | 查看已授权大棚数据 |

#### 知识库管理(Web管理端)
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/knowledge/documents | 文档列表 |
| POST | /api/v1/knowledge/documents | 上传文档 |
| DELETE | /api/v1/knowledge/documents/{id} | 删除文档 |
| POST | /api/v1/knowledge/index | 触发向量化 |
| POST | /api/v1/knowledge/test | 问答测试 |

#### 语料管理(Web管理端)
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/corpus | 语料列表 |
| POST | /api/v1/corpus | 上传语料 |
| DELETE | /api/v1/corpus/{id} | 删除语料 |
| PUT | /api/v1/corpus/{id}/usage | 更新语料用途 |

#### 用户管理(Web管理端)
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/admin/users | 用户列表(含角色/地区筛选) |
| PUT | /api/v1/admin/users/{id} | 更新用户 |
| GET | /api/v1/admin/users/regions | 用户地区分布统计 |
| GET | /api/v1/admin/roles | 角色列表 |

#### AI引擎管理(Web管理端)
| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/admin/ai/config | 当前AI引擎配置 |
| PUT | /api/v1/admin/ai/config | 切换AI引擎 |
| GET | /api/v1/admin/ai/status | 各引擎状态和调用量 |

### 5.6 WebSocket (STOMP)端点

```
连接端点: /ws/connect

订阅(数据推送):
  /topic/greenhouse/{id}/realtime          实时传感器数据(可按组过滤)
  /topic/greenhouse/{id}/alerts            预警推送
  /topic/greenhouse/{id}/health            健康评分推送
  /topic/device/{id}/status                设备状态变更

订阅(实时聊天):
  /user/queue/messages                     个人消息队列
  /topic/conversation/{id}                 对话消息推送(双方订阅)
  /topic/expert/notifications              专家通知队列
  /topic/user/notifications                用户通知队列

发送(实时聊天):
  /app/chat/send                           发送消息
  /app/chat/snapshot                       发送环境快照
  /app/chat/read                           标记已读
```

### 5.7 WebSocket消息JSON格式规范

#### 聊天消息格式

**发送消息** (STOMP SEND → /app/chat/send)：
```json
{
  "conversationId": 1,
  "messageType": "TEXT",
  "content": "请问这个叶子发黄是什么问题？"
}
```

**接收消息** (STOMP SUBSCRIBE ← /topic/conversation/{id})：
```json
{
  "id": 123,
  "conversationId": 1,
  "senderId": 5,
  "senderType": "USER",
  "senderName": "张农户",
  "senderAvatar": "/avatars/5.jpg",
  "messageType": "TEXT",
  "content": "请问这个叶子发黄是什么问题？",
  "filePath": null,
  "snapshotData": null,
  "readStatus": false,
  "createdAt": "2026-07-11T10:30:00"
}
```

**messageType枚举**：`TEXT` | `IMAGE` | `VIDEO` | `ENV_SNAPSHOT`

**图片/视频消息**（content为描述，filePath为实际路径）：
```json
{
  "messageType": "IMAGE",
  "content": "[图片]",
  "filePath": "/uploads/chat/2026/07/11/abc123.jpg"
}
```

#### 环境快照格式

**发送环境快照** (STOMP SEND → /app/chat/snapshot)：
```json
{
  "conversationId": 1,
  "greenhouseId": 3,
  "groupIds": [1, 2, 3]
}
```

**接收环境快照**（后端自动组装后推送给双方）：
```json
{
  "id": 124,
  "conversationId": 1,
  "senderId": 5,
  "senderType": "USER",
  "messageType": "ENV_SNAPSHOT",
  "content": "[环境快照] 1号大棚 · 2026-07-11 10:30",
  "filePath": null,
  "snapshotData": {
    "greenhouseId": 3,
    "greenhouseName": "1号大棚",
    "capturedAt": "2026-07-11T10:30:00",
    "groups": [
      {
        "groupId": 1,
        "zoneLabel": "东侧",
        "sensors": {
          "TEMP": 28.5,
          "HUMIDITY": 65.2,
          "LIGHT": 32000,
          "CO2": 420,
          "O2": 20.8,
          "SOIL_TEMP": 24.3,
          "SOIL_HUMIDITY": 45.6,
          "EC": 1.2,
          "N": 120,
          "P": 45,
          "K": 180
        }
      }
    ],
    "avgTemp": 28.1,
    "avgHumidity": 64.8,
    "alertCount": 0
  },
  "createdAt": "2026-07-11T10:30:00"
}
```

#### 实时数据推送格式

**传感器实时数据** (STOMP SUBSCRIBE ← /topic/greenhouse/{id}/realtime)：
```json
{
  "greenhouseId": 3,
  "timestamp": "2026-07-11T10:30:00",
  "groups": [
    {
      "groupId": 1,
      "zoneLabel": "东侧",
      "online": true,
      "lastHeartbeat": "2026-07-11T10:29:45",
      "sensors": {
        "TEMP": 28.5,
        "HUMIDITY": 65.2,
        "LIGHT": 32000,
        "CO2": 420,
        "O2": 20.8,
        "SOIL_TEMP": 24.3,
        "SOIL_HUMIDITY": 45.6,
        "EC": 1.2,
        "N": 120,
        "P": 45,
        "K": 180
      }
    }
  ]
}
```

**预警推送** (STOMP SUBSCRIBE ← /topic/greenhouse/{id}/alerts)：
```json
{
  "id": 56,
  "greenhouseId": 3,
  "groupId": 2,
  "zoneLabel": "西侧",
  "level": "WARNING",
  "title": "温度过高预警",
  "content": "西侧区域温度达到35.2°C，超过预警上限35°C",
  "sensorType": "TEMP",
  "sensorValue": 35.2,
  "thresholdValue": 35.0,
  "createdAt": "2026-07-11T10:30:00"
}
```

**通知推送** (STOMP SUBSCRIBE ← /user/queue/notifications)：
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

**notification type枚举**：`AUTHORIZATION_REQUEST` | `AUTHORIZATION_APPROVED` | `AUTHORIZATION_REJECTED` | `AUTHORIZATION_EXPIRED` | `AUTHORIZATION_REVOKED` | `NEW_MESSAGE` | `ALERT_TRIGGERED`

### 5.8 分页规范

- 分页参数：`page`（页码，从1开始）、`size`（每页条数，默认20，最大100）
- 排序参数：`sort`（字段名）、`order`（asc/desc，默认desc）
- 响应必须包含 `total`（总条数）

### 5.9 API禁止事项

- **禁止**跳过认证直接访问任何业务API
- **禁止**在API响应中返回用户密码等敏感信息
- **禁止**不使用统一响应格式
- **禁止**在GET请求中传递大量数据（使用POST）
- **禁止**返回数据库原始错误信息给前端
- **禁止**不做权限校验就返回数据

---

## 6. 嵌入式开发规范

### 6.1 ESP32开发语言和框架

- 必须使用 **C/C++**（Arduino框架）
- 开发环境：**PlatformIO**（VS Code插件）
- 核心框架：**Arduino-ESP32 2.x**
- 禁止使用MicroPython或ESP-IDF原生开发

### 6.2 传感器驱动接口规范

每个传感器驱动必须实现统一接口：

```cpp
class SensorDriver {
public:
    virtual bool begin() = 0;          // 初始化传感器
    virtual bool isAvailable() = 0;    // 检测传感器是否在线
    virtual float readValue() = 0;     // 读取传感器数值
    virtual String getSensorType() = 0;// 返回传感器类型标识
};
```

### 6.3 MQTT Topic命名规范

```
传感数据 (ESP32 → Broker → Spring Boot):
  greenhouse/{gh_id}/group/{group_id}/device/{dev_id}/sensor/{sensor_type}
  sensor_type: temp, humidity, light, co2, o2, soil_temp, soil_humidity, ec, n, p, k

设备控制 (Spring Boot → Broker → ESP32):
  greenhouse/{gh_id}/device/{dev_id}/control/{actuator_id}

设备状态 (ESP32 → Broker → Spring Boot):
  greenhouse/{gh_id}/device/{dev_id}/status

设备心跳 (ESP32 → Broker → Spring Boot):
  greenhouse/{gh_id}/device/{dev_id}/heartbeat
```

### 6.4 MQTT消息JSON格式

**传感数据上报**：
```json
{"value": 25.3, "timestamp": 1690000000, "group_id": 2}
```

**设备控制指令**：
```json
{"action": "ON", "timestamp": 1690000000}
```

**设备状态上报**：
```json
{"online": true, "uptime": 3600, "group_id": 2, "actuators": {"1": "ON", "2": "OFF"}}
```

**设备心跳**：
```json
{"timestamp": 1690000000, "group_id": 2}
```

### 6.5 引脚分配表

| GPIO | 功能 | 接口类型 | 连接设备 |
|------|------|----------|----------|
| GPIO21 | I2C SDA | I2C | SHT30/BH1750/EC探头 |
| GPIO22 | I2C SCL | I2C | SHT30/BH1750/EC探头 |
| GPIO34 | ADC | ADC | MQ-135 (CO2) |
| GPIO35 | ADC | ADC | O2传感器模块 |
| GPIO32 | ADC | ADC | 土壤湿度传感器 |
| GPIO4 | 单总线 | GPIO | DS18B20 (土壤温度) |
| GPIO16 | Serial2 TX | UART | NPK传感器(RS485→TTL) |
| GPIO17 | Serial2 RX | UART | NPK传感器(RS485→TTL) |
| GPIO25 | 数字输出 | GPIO | 继电器第1路 → 通风风机 |
| GPIO26 | 数字输出 | GPIO | 继电器第2路 → 卷帘/遮阳网(共用电机) |
| GPIO27 | 数字输出 | GPIO | 继电器第3路 → 滴灌阀门 |
| GPIO14 | 数字输出 | GPIO | 继电器第4路 → 补光灯 |

### 6.6 数据上报频率

| 数据类型 | 频率 | 说明 |
|----------|------|------|
| 传感器数据 | 每30秒 | 全部传感器参数 |
| 设备心跳 | 每60秒 | 保活+状态上报 |
| RTSP推流 | 持续 | VGA(640x480), 10fps, JPEG质量50% |
| FFmpeg截帧 | 每30分钟 | 从RTSP流截帧存储 |

### 6.7 嵌入式禁止事项

- **禁止**硬编码WiFi密码和MQTT地址（必须通过配置或动态获取）
- **禁止**在传感器读取失败时不处理（必须有异常值和错误处理）
- **禁止**MQTT消息中缺少 `group_id` 字段
- **禁止**使用阻塞式延时（必须使用非阻塞方式）
- **禁止**不实现看门狗机制（断线必须自动重连，指数退避重试）

---

## 7. 编码规范

### 7.1 Java编码风格

- 类名：PascalCase（如 `GreenhouseController`）
- 方法名/变量名：camelCase（如 `getGreenhouseById`）
- 常量：UPPER_SNAKE_CASE（如 `MAX_RETRY_COUNT`）
- 包名：全小写（如 `com.greenhouse.module.auth`）
- 缩进：4个空格，禁止Tab
- 每行不超过120字符
- 必须使用Lombok简化getter/setter/constructor

### 7.2 异常处理策略

- 必须使用全局异常处理器 `@RestControllerAdvice`
- 业务异常统一抛出 `BusinessException`（含错误码+错误信息）
- 禁止吞掉异常（catch后必须处理：记录日志或重新抛出）
- 外部API调用（百度/讯飞/DeepSeek）必须有超时设置和重试机制
- 禁止向客户端暴露堆栈信息

### 7.3 日志规范

- 必须使用SLF4J + Logback
- 日志级别使用规范：
  - `ERROR`：系统异常、外部API调用失败
  - `WARN`：业务异常、权限拒绝、数据异常
  - `INFO`：关键业务流程（登录、设备上线、预警触发、AI调用）
  - `DEBUG`：开发调试信息（生产环境关闭）
- 日志格式：`[时间] [级别] [线程] [类名] - 消息`
- 禁止使用 `System.out.println`

### 7.4 安全规范

- 密码必须使用 **BCrypt** 加密存储，禁止明文
- 必须防范SQL注入：使用JPA参数化查询，禁止拼接SQL字符串
- 必须防范XSS：前端输入转义，后端输出编码
- API Key等敏感数据必须通过**环境变量**注入，禁止硬编码
- 禁止在前端（APP/Web）存储敏感信息（密码、Token明文）
- JWT Token有效期：建议2小时，支持刷新
- 文件上传必须校验类型和大小

### 7.5 注释规范

- 所有公开API方法必须有JavaDoc注释
- 复杂业务逻辑必须有行内注释
- AI抽象层接口必须有接口说明注释
- 禁止无意义注释（如 `// 获取用户` 在 `getUser()` 上面）

### 7.6 编码禁止事项

- **禁止**在Controller层编写业务逻辑（必须放在Service层）
- **禁止**直接返回Entity对象给前端（必须使用DTO）
- **禁止**在循环中执行数据库查询（N+1问题）
- **禁止**使用 `@Autowired` 字段注入（推荐构造器注入）
- **禁止**不校验参数就直接处理业务
- **禁止**在业务代码中硬编码配置值

---

## 8. 测试规范

### 8.1 测试框架与工具

| 层级 | 框架 | 用途 |
|------|------|------|
| 单元测试 | JUnit 5 + Mockito | Service层业务逻辑测试 |
| 集成测试 | Spring Boot Test + MockMvc | Controller层API测试 |
| 数据库测试 | @DataJpaTest + H2内存库 | Repository层数据访问测试 |
| API文档测试 | SpringDoc / Knife4j | API文档自动生成（可选） |

### 8.2 测试覆盖率要求

| 层级 | 最低覆盖率 | 说明 |
|------|-----------|------|
| Service层 | **80%+**（行覆盖） | 核心业务逻辑必须全覆盖 |
| Controller层 | **60%+**（行覆盖） | 至少覆盖正常请求+异常请求 |
| Repository层 | **50%+**（行覆盖） | 自定义查询方法必须测试 |
| AI抽象层 | **100%**（接口契约） | 每个Provider实现必须通过相同测试用例 |

### 8.3 测试命名规范

```
测试类命名：{被测类名}Test.java
测试方法命名：{方法名}_{场景}_{预期结果}
示例：
  - GreenhouseServiceTest.java
  - createGreenhouse_ValidInput_ShouldSucceed()
  - createGreenhouse_DuplicateName_ShouldThrowException()
  - getRealtimeData_UnauthorizedUser_ShouldReturn403()
```

### 8.4 必须测试的场景

每个Service方法至少覆盖以下场景：
1. **正常场景**：合法输入 → 预期输出
2. **边界场景**：空值/null/最大值/最小值
3. **异常场景**：权限不足/资源不存在/参数非法
4. **外部依赖异常**：AI API超时/MQTT断连/数据库异常

### 8.5 测试示例

```java
@ExtendWith(MockitoExtension.class)
class GreenhouseServiceTest {

    @Mock
    private GreenhouseRepository greenhouseRepository;
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private GreenhouseService greenhouseService;

    @Test
    @DisplayName("创建大棚 - 正常输入应成功")
    void createGreenhouse_ValidInput_ShouldSucceed() {
        // Given
        GreenhouseDTO dto = new GreenhouseDTO();
        dto.setName("1号大棚");
        dto.setLocation("河北省石家庄市");
        
        User owner = new User();
        owner.setId(1L);
        owner.setRole(Role.OWNER);
        
        when(userRepository.findById(1L)).thenReturn(Optional.of(owner));
        when(greenhouseRepository.save(any())).thenReturn(new Greenhouse());
        
        // When
        Greenhouse result = greenhouseService.createGreenhouse(1L, dto);
        
        // Then
        assertNotNull(result);
        verify(greenhouseRepository).save(any());
    }

    @Test
    @DisplayName("创建大棚 - 员工角色应抛出权限异常")
    void createGreenhouse_WorkerRole_ShouldThrowException() {
        // Given
        User worker = new User();
        worker.setId(2L);
        worker.setRole(Role.WORKER);
        
        when(userRepository.findById(2L)).thenReturn(Optional.of(worker));
        
        // When & Then
        assertThrows(BusinessException.class, () -> {
            greenhouseService.createGreenhouse(2L, new GreenhouseDTO());
        });
    }
}
```

### 8.6 Controller集成测试示例

```java
@SpringBootTest
@AutoConfigureMockMvc
class GreenhouseControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    @DisplayName("获取大棚列表 - 已认证应返回200")
    void getGreenhouses_Authenticated_ShouldReturn200() throws Exception {
        mockMvc.perform(get("/api/v1/greenhouses")
                .header("Authorization", "Bearer " + validToken))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code").value(200))
                .andExpect(jsonPath("$.data").isArray());
    }

    @Test
    @DisplayName("获取大棚列表 - 未认证应返回401")
    void getGreenhouses_Unauthenticated_ShouldReturn401() throws Exception {
        mockMvc.perform(get("/api/v1/greenhouses"))
                .andExpect(status().isUnauthorized());
    }
}
```

### 8.7 测试禁止事项

- **禁止**将测试代码提交到生产分支时测试不通过
- **禁止**在测试中使用真实的外部API（必须Mock）
- **禁止**测试之间相互依赖（每个测试独立运行）
- **禁止**在测试中硬编码环境相关的值（端口、路径等）

---

## 9. 功能模块开发指南

### 9.1 后端模块清单

| 编号 | 模块名称 | 一句话说明 |
|------|----------|-----------|
| C1 | 用户认证模块 | 注册/登录/JWT/四种角色权限(ADMIN仅Web端) |
| C2 | 设备管理模块 | ESP32注册/绑定/分组/心跳检测 |
| C3 | 大棚管理模块 | 多大棚CRUD/用户绑定/五级地区地址 |
| C4 | MQTT消费模块 | 订阅传感topic→存InfluxDB(含group_id) |
| C5 | 时序数据服务 | 查询/聚合/导出/多组对比/区域聚合 |
| C6 | 环境预警引擎 | 阈值+统计预测→分级告警+定位到传感器组+用户自定义阈值 |
| C7 | 设备控制模块 | 下发指令→MQTT publish+权限校验 |
| C8 | 图片诊断模块 | 接收图片→AI抽象层→关联知识库防治方案→低置信度引导专家 |
| C9 | RAG问答模块 | Chroma向量检索+DeepSeek生成答案+引用来源 |
| C10 | 语音识别模块 | 接收音频→AI抽象层(讯飞/Whisper)→文本→问答 |
| C11 | WebSocket推送 | 实时数据+预警推送+聊天消息推送 |
| C12 | 场景联动引擎 | 预设场景→多设备协同控制 |
| C13 | 文件存储模块 | 图片/音频/视频/截帧存储管理(本地文件系统) |
| C14 | 天气API对接 | 和风天气预报(每3小时拉取)→辅助预警 |
| C15 | 多模态融合分析 | 环境健康分(60%)+视觉健康分(40%)→综合健康评分(0-100) |
| C16 | 实时聊天模块 | WebSocket双向通信、消息持久化、环境快照收发 |
| C17 | 专家授权模块 | 授权请求→同意→7天有效期→自动过期→可撤销 |
| C18 | 多角色权限模块 | RBAC+大棚授权+功能授权矩阵+员工单归属 |
| C19 | 设备分组管理 | 传感器组CRUD、区域标注、组内设备管理 |
| C20 | 地区管理模块 | 五级地址树(省-市-区/县-乡镇-村)、地区分布统计 |
| C21 | 自定义预警阈值 | 用户设置各参数最高/最低预警值，优先于系统默认 |
| C22 | 作物生长周期管理 | 种植记录CRUD、阶段自动估算(按日期)+手动修正、生长时间线 |

### 9.2 APP端功能模块

| 编号 | 模块 | 功能概要 |
|------|------|----------|
| F1 | 实时数据看板 | 仪表盘/趋势曲线/多棚切换/多组对比/聚合视图/区域标注 |
| F2 | 环境预警中心 | 分级告警/推送/天气预判/预警定位到传感器组/自定义阈值 |
| F3 | 病虫害诊断 | 拍照→识别+防治方案/低置信度引导求助专家 |
| F4 | AI智能问答 | 文字/语音(河北话)→RAG答案+引用来源 |
| F5 | 设备控制 | 场景联动/单独控制/按区域控制 |
| F6 | 历史数据 | 时间范围/参数/趋势对比/多组对比 |
| F7 | 作物长势评估 | 截帧查看/生长阶段/株高叶面积叶色/种植周期/生长时间线 |
| F8 | 多模态健康评分 | 综合环境+图像健康状态→综合评分 |
| F9 | 个人中心 | 账号/大棚/通知/员工管理/权限分配/五级地址 |
| F10 | 专家咨询 | 聊天/图片/视频/环境快照/授权管理 |
| F11 | 角色适配 | 棚主/员工界面差异、权限自动过滤(无管理员) |

### 9.3 Web管理端功能模块

| 编号 | 模块 | 功能概要 | 适用角色 |
|------|------|----------|----------|
| G1 | 数据总览大屏 | 多棚汇总/地图/地区分布/传感器组状态 | 管理员 |
| G2 | 设备管理 | ESP32注册/分组管理/传感器/执行器配置 | 管理员 |
| G3 | 用户与角色管理 | 农户CRUD/四种角色/专家账号/棚主-员工关系/地区分布 | 管理员 |
| G4 | 知识库管理 | 文档上传/向量化/问答测试 | 管理员 |
| G5 | 预警规则配置 | 阈值/场景联动/异常模式库 | 管理员 |
| G6 | 数据导出报表 | 历史/预警/日志/咨询记录/员工日志 | 管理员 |
| G7 | 系统监控 | 服务状态/MQTT/WebSocket连接/聊天量 | 管理员 |
| G8 | 方言语料管理 | 语料上传/标注/查看 | 管理员 |
| G9 | 专家工作台 | 求助列表/对话/环境数据查看/授权管理/统计 | 专家 |
| G10 | 棚主Web管理 | 大棚总览/员工管理/权限矩阵/数据导出 | 棚主 |

### 9.4 模块间依赖关系

```
C1(认证) → 所有模块(权限校验)
C3(大棚) → C2(设备)、C5(数据)、C6(预警)
C2(设备) → C4(MQTT)、C7(控制)
C4(MQTT) → C5(时序数据)
C5(时序数据) → C6(预警)、C15(融合)
C6(预警) → C12(场景联动)、C11(推送)
C7(控制) → C4(MQTT publish)
C8(诊断) → C15(融合)、C16(低置信度引导专家)
C9(RAG) → C8(知识库检索)
C11(WebSocket) → C16(聊天)
C14(天气) → C6(预警)、C15(融合)
C16(聊天) → C17(授权)
C18(权限) → 所有模块
C19(分组) → C2(设备)、C5(数据)
C22(生长周期) → F7(长势评估)
```

### 9.5 开发顺序建议

1. **基础层**：C1(认证) → C3(大棚) → C2(设备) → C19(分组) → C18(权限)
2. **数据层**：C4(MQTT) → C5(时序数据) → C11(WebSocket推送)
3. **业务层**：C6(预警) → C7(控制) → C12(场景联动) → C21(自定义阈值)
4. **AI层**：C8(诊断) → C9(RAG) → C10(语音) → C15(融合)
5. **扩展层**：C14(天气) → C13(文件) → C16(聊天) → C17(授权) → C22(生长周期)
6. **管理层**：C20(地区) → 知识库管理 → 语料管理

---

## 10. 硬件约束

### 10.1 传感器清单

| 传感器 | 型号 | 接口 | 数量/组 | 通信方式 |
|--------|------|------|---------|----------|
| 空气温湿度 | SHT30 | I2C | 1 | I2C (GPIO21/22) |
| 光照强度 | BH1750FVI | I2C | 1 | I2C (GPIO21/22) |
| CO2浓度 | MQ-135 | ADC | 1 | ADC (GPIO34) |
| O2浓度 | O2传感器模块 | ADC | 1 | ADC (GPIO35) |
| 土壤湿度 | 电容式土壤水分 | ADC | 1 | ADC (GPIO32) |
| 土壤温度 | DS18B20防水型 | 单总线 | 1 | GPIO (GPIO4) |
| 土壤EC | 电导率探头 | I2C | 1 | I2C (GPIO21/22) |
| NPK传感器 | RS485五合一 | UART | 1 | UART (GPIO16/17, RS485→TTL) |

### 10.2 执行器清单

| 设备 | 控制方式 | 数量/组 |
|------|----------|---------|
| 通风风机 | 继电器开关 | 1 |
| 电动卷帘 | 继电器正反转 | 1 |
| 遮阳网 | 继电器开关 | 1 |
| 滴灌阀门 | 继电器开关 | 1 |
| 补光灯 | 继电器开关 | 1 |

### 10.3 4路继电器具体分配

| 路数 | GPIO | 控制设备 | 说明 |
|------|------|----------|------|
| 第1路 | GPIO25 | 通风风机 | 通风换气 |
| 第2路 | GPIO26 | 卷帘/遮阳网 | 共用电机（正转=展开卷帘，反转=收起遮阳网） |
| 第3路 | GPIO27 | 滴灌阀门 | 灌溉 |
| 第4路 | GPIO14 | 补光灯 | LED植物补光 |

> 注：申报书要求5种执行设备，4路继电器通过合并卷帘+遮阳网为1路实现。如实际场景卷帘和遮阳网为独立电机，需升级为8路继电器模块。

### 10.4 主控芯片和摄像头

| 设备 | 型号 | 数量/大棚 | 说明 |
|------|------|-----------|------|
| 主控芯片 | ESP32-DevKitC V4 (38pin) | 3-5个 | 每组传感器1个，独立WiFi+MQTT |
| 摄像头 | ESP32-CAM (AI-Thinker) | 1个 | RTSP推流，独立节点不归属任何组 |

### 10.5 硬件接口约束

- I2C总线最多挂载3个设备（SHT30+BH1750+EC），地址不能冲突
- ADC通道有限（GPIO34/35/32），每个接一个模拟传感器
- UART Serial2用于NPK传感器（RS485通信需MAX485转TTL模块）
- 单总线GPIO4仅接DS18B20
- 4路继电器占用GPIO25/26/27/14
- 每组ESP32独立WiFi连接，MQTT消息必须携带group_id

---

## 11. 架构决策记录（ADR）摘要

| ADR编号 | 标题 | 核心决策 |
|---------|------|----------|
| ADR-001 | 单体Spring Boot架构 | 单体应用+Maven多模块，不为一人开发上微服务 |
| ADR-002 | AI模型API先行+预留自训练接口 | 初期API，后期Colab训练替换，策略模式保证无缝切换 |
| ADR-003 | Spring AI统一RAG管线 | 不引入Python，Java内完成RAG |
| ADR-004 | 前后端分离，后端纯API | Spring Boot只提供REST+WebSocket，Vue 3独立部署 |
| ADR-005 | 本地Linux+frp内网穿透 | 零服务器成本，Docker restart=always保证稳定性 |
| ADR-006 | 时序预测分阶段实施 | 第一阶段统计方法，第二阶段LSTM(Colab)，预测30分钟 |
| ADR-007 | 多组传感器独立ESP32部署 | 每组独立ESP32+WiFi+MQTT，消息含group_id，3-5组/棚 |
| ADR-008 | 专家实时聊天WebSocket复用 | 复用Spring WebSocket(STOMP)，不引入独立消息中间件 |
| ADR-009 | 环境数据授权7天过期机制 | 双轨制：环境快照(一次性)+数据授权(7天有效期+可撤销) |
| ADR-010 | 棚主-员工多角色权限模型 | 四种角色，APP端仅OWNER+WORKER，员工单归属 |
| ADR-011 | ESP32-CAM RTSP推流+FFmpeg截帧 | 符合申报书RTMP/RTSP要求，VGA/10fps |
| ADR-012 | 多模态融合分析模块 | 环境健康分(60%)+视觉健康分(40%)→综合评分(0-100) |
| ADR-013 | 用户自定义预警阈值 | 用户可设置各参数最高/最低值，优先于系统默认 |
| ADR-014 | 轻量级知识图谱方案 | 以Chroma向量库+结构化分类体系代替图数据库 |
| ADR-015 | 作物生长周期管理 | 种植记录CRUD+阶段自动估算+手动修正+生长时间线 |
| ADR-016 | Embedding服务选型 | DeepSeek无Embedding API，使用SiliconFlow bge-m3 |

---

## 12. 外部服务集成规范

### 12.1 DeepSeek API调用规范（LLM对话）

- 模型名称：`deepseek-chat`
- 接入方式：通过Spring AI OpenAI兼容接口
- 配置：`spring.ai.openai.base-url=https://api.deepseek.com`
- API Key：通过环境变量 `DEEPSEEK_API_KEY` 注入
- 用途：RAG问答中的答案生成
- 注意：`spring.ai.openai.embedding.enabled=false`（DeepSeek无Embedding）

### 12.2 Embedding服务调用规范（SiliconFlow bge-m3）

- 模型名称：`BAAI/bge-m3`
- 接入方式：SiliconFlow API（OpenAI兼容格式），通过OkHttp调用或自定义Bean
- Base URL：`https://api.siliconflow.cn/v1`
- API Key：通过环境变量 `SILICONFLOW_API_KEY` 注入
- 向量维度：1024维
- 免费额度：100万tokens/天
- 备选方案：智谱embedding-2 或 阿里云DashScope text-embedding-v2

### 12.3 百度AI图像识别API规范

- 接口地址：`https://aip.baidubce.com/rest/2.0/image-classify/v1/plant`
- 认证：API Key + Secret Key → Access Token
- 配置：环境变量 `BAIDU_API_KEY`、`BAIDU_SECRET_KEY`
- 用途：病虫害图像识别（初期）
- AI抽象层接口：`DiseaseRecognitionProvider.recognize(byte[] imageData)`
- 切换配置：`ai.image.provider=baidu`

**图像识别返回值DTO**（DiseaseRecognitionResult）：
```java
package com.greenhouse.ai.dto;

@Data
public class DiseaseRecognitionResult {
    private String diseaseName;        // 病害名称，如"黄瓜霜霉病"
    private String diseaseCategory;    // 病害分类：FUNGAL/BACTERIAL/VIRAL/PEST/NUTRIENT/NORMAL
    private Double confidence;         // 置信度 0.0-1.0
    private String description;        // 病害描述
    private String treatment;          // 防治方案（综合方案）
    private TreatmentDetail treatmentDetail; // 结构化防治方案
    private String recognitionEngine;  // 识别引擎：baidu/resnet
    private String rawResponse;        // 原始API响应（调试用，可空）

    @Data
    public static class TreatmentDetail {
        private String agricultural;   // 农业防治
        private String physical;       // 物理防治
        private String biological;     // 生物防治
        private String chemical;       // 化学防治（含具体药剂和用量）
        private List<String> precautions; // 注意事项
    }
}
```

**AI抽象层接口完整定义**：
```java
package com.greenhouse.ai;

public interface DiseaseRecognitionProvider {
    /**
     * 识别病虫害
     * @param imageData 图片字节数据
     * @return 识别结果，包含病害名称、置信度、防治方案
     * @throws BusinessException(ErrorCode.AI_RECOGNITION_FAILED) 识别失败时
     */
    DiseaseRecognitionResult recognize(byte[] imageData);
    
    /**
     * 获取当前使用的识别引擎名称
     */
    String getEngineName();
}
```

### 12.4 讯飞/Whisper方言识别API规范

- 讯飞配置：环境变量 `XUNFEI_APP_ID`、`XUNFEI_API_KEY`、`XUNFEI_API_SECRET`
- 方言类型：河北话
- AI抽象层接口：`SpeechRecognitionProvider.recognize(byte[] audioData)`
- 切换配置：`ai.voice.provider=xunfei`（初期）/ `whisper`（后期）

**语音识别返回值DTO**（SpeechRecognitionResult）：
```java
package com.greenhouse.ai.dto;

@Data
public class SpeechRecognitionResult {
    private String text;               // 识别出的文本（普通话）
    private String rawDialectText;     // 方言原文（如有，可空）
    private Double confidence;         // 置信度 0.0-1.0
    private String dialect;            // 方言类型：hebei
    private String recognitionEngine;  // 识别引擎：xunfei/whisper
    private Integer durationMs;        // 音频时长（毫秒）
}
```

**语音识别接口完整定义**：
```java
package com.greenhouse.ai;

public interface SpeechRecognitionProvider {
    /**
     * 识别语音（支持方言）
     * @param audioData 音频字节数据（支持wav/mp3/amr格式）
     * @return 识别结果，包含文本和置信度
     * @throws BusinessException(ErrorCode.AI_SPEECH_FAILED) 识别失败时
     */
    SpeechRecognitionResult recognize(byte[] audioData);
    
    /**
     * 获取当前使用的语音识别引擎名称
     */
    String getEngineName();
    
    /**
     * 获取支持的方言列表
     */
    List<String> getSupportedDialects();
}
```

### 12.5 和风天气API规范

- Base URL：`https://devapi.qweather.com`
- API Key：环境变量 `QWEATHER_API_KEY`
- 免费版限制：1000次/天
- 拉取频率：每3小时定时拉取
- 存储：MySQL `weather_cache` 表

### 12.6 API Key管理方式

- **必须**通过环境变量（.env文件）注入，禁止硬编码
- **必须**在 `application.yml` 中使用 `${ENV_VAR}` 占位符引用
- **禁止**将API Key提交到Git仓库
- **禁止**在日志中打印API Key

### 12.7 调用失败处理策略

- 所有外部API调用必须设置超时（连接超时5s，读取超时10s）
- 失败时必须有重试机制（最多3次，指数退避）
- 重试失败后必须记录ERROR日志并返回友好错误提示
- AI识别失败时降级处理：图像识别失败→提示重试；语音识别失败→提示文字输入；LLM失败→返回"AI服务暂时不可用"

---

## 13. 开发禁忌总清单

以下所有条目为**强制禁止**，违反任何一条都可能导致系统不可用或偏离架构方案：

### 技术栈禁忌
1. **禁止**更换任何技术栈组件（见第2节技术栈表，全部标注"禁止替换"）
2. **禁止**引入Python后端服务
3. **禁止**使用微服务架构
4. **禁止**使用Thymeleaf等服务端模板
5. **禁止**使用Kotlin开发Android APP

### 数据库禁忌
6. **禁止**直接删除表或字段（必须通过迁移脚本）
7. **禁止**在代码中直接执行DDL语句
8. **禁止**级联删除外键关联数据
9. **禁止**修改已有字段类型或名称

### 安全禁忌
10. **禁止**硬编码API Key（必须通过环境变量注入）
11. **禁止**明文存储密码（必须BCrypt加密）
12. **禁止**跳过JWT认证直接访问业务API
13. **禁止**在前端存储敏感信息（密码、Token明文）
14. **禁止**向客户端暴露数据库错误堆栈
15. **禁止**将API Key提交到Git仓库

### 角色权限禁忌
16. **禁止**在APP端实现管理员(ADMIN)角色
17. **禁止**在Web端实现员工(WORKER)角色
18. **禁止**允许员工归属多个棚主（员工单归属）
19. **禁止**专家拥有设备控制权限

### 嵌入式禁忌
20. **禁止**MQTT消息中缺少group_id字段
21. **禁止**硬编码WiFi密码和MQTT地址
22. **禁止**不实现看门狗和断线重连机制
23. **禁止**使用阻塞式延时

### 编码禁忌
24. **禁止**在Controller层编写业务逻辑
25. **禁止**直接返回Entity对象给前端（必须用DTO）
26. **禁止**使用System.out.println（必须用SLF4J日志）
27. **禁止**在循环中执行数据库查询
28. **禁止**不校验参数就处理业务

### 架构禁忌
29. **禁止**绕过AI抽象层直接调用外部API（必须通过Provider接口）
30. **禁止**删除或修改AI抽象层接口定义
31. **禁止**引入独立消息中间件（聊天必须复用WebSocket/STOMP）
32. **禁止**使用DeepSeek进行Embedding向量化（DeepSeek不提供Embedding服务）

---

## 14. 开发优先级与顺序

### 14.1 开发计划时间表

| Phase | 时间 | 核心任务 | 产出 |
|-------|------|----------|------|
| Phase 1 | 2026.05-07 | 基础设施：需求/架构/硬件原型/Docker环境/Spring Boot骨架 | 后端收MQTT存InfluxDB，RTSP截帧跑通 |
| Phase 2 | 2026.08-10 | 核心功能：用户认证/设备管理/大棚管理/时序数据/APP核心/AI抽象层 | APP能看实时数据+控制设备+管理员工 |
| Phase 3 | 2026.09-11 | AI能力+专家系统：图像识别/RAG问答/语音/专家咨询/多模态融合 | AI问答+诊断+专家咨询全流程可用 |
| Phase 4 | 2026.11-2027.01 | 管理端+模型训练：Vue Web管理端/ResNet训练/Whisper微调/LSTM | Web管理端+自训练模型上线 |
| Phase 5 | 2027.02-05 | 集成验收：联调/实地部署/数据收集/论文/结题 | 完整交付物 |

### 14.2 模块开发依赖关系

```
Phase 1 (基础设施):
  Docker环境 → MySQL/InfluxDB/Mosquitto/Chroma → Spring Boot骨架 → MQTT消费(C4) → 时序数据(C5)

Phase 2 (核心功能):
  C1(认证) → C3(大棚) → C2(设备) → C19(分组) → C18(权限)
  → C7(控制) → C12(场景联动) → C11(WebSocket)
  → C6(预警) → C21(自定义阈值) → C14(天气)
  → APP端: F1(看板) → F5(控制) → F2(预警) → F9(个人中心)

Phase 3 (AI+专家):
  AI抽象层 → C8(诊断) → C9(RAG) → C10(语音)
  → C15(融合) → C16(聊天) → C17(授权)
  → APP端: F3(诊断) → F4(问答) → F7(长势) → F10(专家咨询)

Phase 4 (管理端+训练):
  G1-G8(Web管理端) → G9(专家工作台) → G10(棚主管理)
  → ResNet训练 → Whisper微调 → LSTM训练
  → C22(生长周期)

Phase 5 (集成):
  全功能联调 → 实地部署 → 数据收集 → 论文
```

### 14.3 建议的开发顺序

**后端开发顺序**（严格按依赖关系）：

1. Spring Boot项目骨架 + Docker环境搭建
2. C1 用户认证（注册/登录/JWT）
3. C3 大棚管理 + C20 地区管理
4. C2 设备管理 + C19 设备分组
5. C18 多角色权限
6. C4 MQTT消费模块
7. C5 时序数据服务
8. C11 WebSocket推送
9. C6 环境预警 + C21 自定义阈值
10. C7 设备控制 + C12 场景联动
11. C14 天气API对接
12. AI抽象层（接口定义）
13. C8 图片诊断（百度API实现）
14. C9 RAG问答（Spring AI + Chroma + DeepSeek）
15. C10 语音识别（讯飞API实现）
16. C15 多模态融合分析
17. C16 实时聊天 + C17 专家授权
18. C13 文件存储
19. C22 作物生长周期管理

**APP开发顺序**（在后端API可用后）：

1. F1 实时数据看板
2. F5 设备控制
3. F9 个人中心（含员工管理）
4. F2 环境预警中心
5. F3 病虫害诊断
6. F4 AI智能问答
7. F6 历史数据
8. F7 作物长势评估
9. F10 专家咨询
10. F8 多模态健康评分
11. F11 角色适配

---

## 15. Git版本管理规范

### 15.1 分支策略

| 分支 | 用途 | 说明 |
|------|------|------|
| `main` | 稳定发布分支 | 只通过PR合并，禁止直接push |
| `develop` | 日常开发分支 | 所有feature/fix分支合并到此 |
| `feature/{模块名}` | 功能开发分支 | 从develop切出，完成后合并回develop |
| `fix/{问题描述}` | Bug修复分支 | 从develop切出，修复后合并回develop |
| `release/{版本号}` | 发布准备分支 | 从develop切出，测试通过后合并到main |

### 15.2 分支命名示例

```
feature/user-auth          # 用户认证模块
feature/mqtt-consumer      # MQTT消费模块
feature/expert-chat        # 专家聊天模块
fix/login-token-expire     # 修复登录Token过期问题
release/v1.0.0             # v1.0.0发布
```

### 15.3 Commit Message规范

采用 **Conventional Commits** 格式：

```
<type>(<scope>): <subject>

[可选 body]

[可选 footer]
```

**type类型**：

| Type | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(device): 新增ESP32设备注册接口` |
| `fix` | Bug修复 | `fix(mqtt): 修复MQTT断线重连后重复订阅问题` |
| `docs` | 文档更新 | `docs(api): 更新API端点文档` |
| `style` | 代码格式（不影响逻辑） | `style: 统一缩进为4空格` |
| `refactor` | 重构（不新增功能/不修复bug） | `refactor(alert): 提取预警规则校验逻辑` |
| `test` | 测试相关 | `test(greenhouse): 补充大棚Service单元测试` |
| `chore` | 构建/工具/依赖 | `chore(deps): 升级Spring Boot到3.3.2` |
| `perf` | 性能优化 | `perf(sensor): 优化时序数据批量写入` |

**scope范围**（按后端模块编号）：

| Scope | 对应模块 |
|-------|----------|
| `auth` | C1 用户认证 |
| `device` | C2 设备管理 / C19 设备分组 |
| `greenhouse` | C3 大棚管理 |
| `mqtt` | C4 MQTT消费 |
| `sensor` | C5 时序数据 |
| `alert` | C6 预警 / C21 自定义阈值 |
| `control` | C7 设备控制 / C12 场景联动 |
| `diagnosis` | C8 图片诊断 |
| `qa` | C9 RAG问答 |
| `speech` | C10 语音识别 |
| `websocket` | C11 WebSocket / C16 聊天 |
| `weather` | C14 天气 |
| `fusion` | C15 多模态融合 |
| `chat` | C16 实时聊天 |
| `authorization` | C17 专家授权 |
| `permission` | C18 权限 |
| `crop` | C22 生长周期 |
| `app` | Android APP端 |
| `web` | Vue Web管理端 |
| `esp32` | ESP32固件 |
| `docker` | Docker/部署相关 |

### 15.4 Commit示例

```bash
git commit -m "feat(device): 新增ESP32设备注册接口

支持设备编码唯一校验和自动绑定大棚
- 新增DeviceController.register()接口
- 新增DeviceService.registerDevice()业务逻辑
- 新增DeviceCodeExistsException异常

Closes #12"

git commit -m "fix(mqtt): 修复MQTT断线重连后重复订阅问题"
```

### 15.5 Git禁止事项

- **禁止**直接push到 `main` 分支
- **禁止**force push到共享分支（`main`、`develop`）
- **禁止**提交包含API Key/密码的代码
- **禁止**提交未通过编译的代码
- **禁止**提交超过500行的大提交（应拆分为多个有意义的提交）
- **禁止**提交 `TODO` 注释而不创建对应的Issue跟踪

---

## 16. 环境变量与配置管理

### 16.1 配置分层策略

```
application.yml              # 通用配置（端口、框架设置）
application-dev.yml          # 开发环境（本地默认值，可提交Git）
application-prod.yml         # 生产环境（通过环境变量注入，不提交敏感值）
.env.example                 # 环境变量模板（可提交Git，不含真实密钥）
.env                         # 真实环境变量（禁止提交Git！）
```

### 16.2 application-dev.yml 开发环境默认值

```yaml
# 开发环境配置 — 本地开发时使用，所有敏感值为占位符
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/smart_greenhouse?useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: root  # 本地开发用简单密码
  
  ai:
    openai:
      api-key: ${DEEPSEEK_API_KEY:sk-dev-placeholder}
      base-url: https://api.deepseek.com
      chat:
        options:
          model: deepseek-chat
      embedding:
        enabled: false

# 本地MQTT（连Docker中的Mosquitto）
mqtt:
  broker: tcp://localhost:1883
  username: greenhouse
  password: greenhouse_dev

# 本地InfluxDB（连Docker中的InfluxDB）
influxdb:
  url: http://localhost:8086
  token: ${INFLUX_TOKEN:dev-token-placeholder}
  org: greenhouse
  bucket: sensor_data

# 本地Chroma（连Docker中的Chroma）
chroma:
  url: http://localhost:8000

# 文件存储（本地目录）
file:
  upload-dir: ./uploads
  max-file-size: 10MB

# 日志级别
logging:
  level:
    com.greenhouse: DEBUG
    org.springframework.ai: DEBUG

# 第三方API（开发时使用占位符，实际值从环境变量读取）
baidu:
  ai:
    api-key: ${BAIDU_API_KEY:dev-baidu-key}
    secret-key: ${BAIDU_SECRET_KEY:dev-baidu-secret}

xunfei:
  app-id: ${XUNFEI_APP_ID:dev-xunfei-id}
  api-key: ${XUNFEI_API_KEY:dev-xunfei-key}
  api-secret: ${XUNFEI_API_SECRET:dev-xunfei-secret}

siliconflow:
  api-key: ${SILICONFLOW_API_KEY:dev-siliconflow-key}
  base-url: https://api.siliconflow.cn/v1
  embedding-model: BAAI/bge-m3

qweather:
  api-key: ${QWEATHER_API_KEY:dev-qweather-key}
  base-url: https://devapi.qweather.com

# AI引擎切换（开发阶段用API）
ai:
  image:
    provider: baidu
  voice:
    provider: xunfei
```

### 16.3 .env.example 模板

```env
# ===== 数据库 =====
MYSQL_PASSWORD=your_mysql_password_here

# ===== InfluxDB =====
INFLUX_USER=greenhouse_admin
INFLUX_PASSWORD=your_influx_password_here
INFLUX_TOKEN=your_influx_token_here

# ===== 第三方API =====
BAIDU_API_KEY=your_baidu_api_key_here
BAIDU_SECRET_KEY=your_baidu_secret_key_here
XUNFEI_APP_ID=your_xunfei_app_id_here
XUNFEI_API_KEY=your_xunfei_api_key_here
XUNFEI_API_SECRET=your_xunfei_api_secret_here
DEEPSEEK_API_KEY=your_deepseek_api_key_here
SILICONFLOW_API_KEY=your_siliconflow_api_key_here
QWEATHER_API_KEY=your_qweather_api_key_here

# ===== MQTT =====
MQTT_USER=greenhouse
MQTT_PASSWORD=your_mqtt_password_here

# ===== frp内网穿透 =====
FRP_SERVER_ADDR=your_frp_server_address_here
FRP_SERVER_PORT=7000
FRP_TOKEN=your_frp_token_here
```

### 16.4 配置管理禁止事项

- **禁止**将 `.env` 文件提交到Git（必须在 `.gitignore` 中排除）
- **禁止**在 `application.yml` 中硬编码真实API Key
- **禁止**在日志中打印环境变量值
- **禁止**生产环境使用 `application-dev.yml` 配置
- `.env.example` 可以提交Git，作为配置模板参考
```

### 16.5 .gitignore 必须包含的条目

```gitignore
# 环境变量（含真实密钥）
.env

# 上传文件目录
uploads/

# IDE
.idea/
*.iml
.vscode/

# 构建产物
target/
dist/
build/
*.class
*.jar
*.war

# 依赖
node_modules/

# 日志
logs/
*.log

# OS
.DS_Store
Thumbs.db

# Docker volumes
docker/volumes/

# 临时文件
*.tmp
*.swp
*.bak
```

---
id: DB-001
title: 数据库设计文档
type: db
module: database
tags: [mysql, influxdb, chroma, schema]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs: [docs/database/table-module-mapping.md]
---

## 概述

### 项目背景

本文档为"基于多模态融合的智慧大棚作物诊疗与智能管控APP"的数据库设计文档。系统采用三库架构：

| 数据库 | 版本 | 用途 | 存储内容 |
|--------|------|------|----------|
| MySQL | 8.0 | 业务数据 | 用户、大棚、设备、预警、诊断、聊天、授权、权限等 |
| InfluxDB | 2.7 | 时序数据 | 传感器读数（含 group_id 标签） |
| Chroma | latest | 向量数据 | 农业知识库 Embedding（SiliconFlow bge-m3, 1024维） |

### 设计原则

1. **命名规范**：表名/字段名使用小写字母+下划线（snake_case），表名使用复数形式
2. **主键**：统一使用 `id BIGINT AUTO_INCREMENT`
3. **外键**：格式 `{关联表单数}_id`，禁止级联删除（CASCADE），使用逻辑删除或手动处理
4. **时间字段**：`created_at`、`updated_at` 使用 DATETIME 类型
5. **状态字段**：`status` 使用 TINYINT，0=禁用/离线/关，1=正常/在线/开
6. **字符集**：utf8mb4，排序规则 utf8mb4_unicode_ci

### 表清单总览（25张表）

| # | 表名 | 用途 | 分类 |
|---|------|------|------|
| 1 | `users` | 用户表（四种角色） | 用户 |
| 2 | `user_addresses` | 用户地区地址表 | 用户 |
| 3 | `greenhouses` | 大棚表 | 大棚 |
| 4 | `device_groups` | 传感器组表 | 设备 |
| 5 | `devices` | ESP32设备表 | 设备 |
| 6 | `sensors` | 传感器配置表 | 设备 |
| 7 | `actuators` | 执行器配置表 | 设备 |
| 8 | `employee_permissions` | 员工权限表 | 权限 |
| 9 | `alert_rules` | 预警规则表 | 预警 |
| 10 | `alerts` | 预警记录表 | 预警 |
| 11 | `scenes` | 场景联动表 | 控制 |
| 12 | `diagnostic_records` | 病虫害诊断记录 | 诊断 |
| 13 | `qa_records` | 问答记录 | AI |
| 14 | `control_logs` | 设备控制日志 | 控制 |
| 15 | `knowledge_documents` | 知识库文档表 | 知识库 |
| 16 | `dialect_corpus` | 方言语料表 | 语料 |
| 17 | `growth_assessments` | 长势评估记录 | 长势 |
| 18 | `health_assessments` | 多模态健康综合评估 | 融合 |
| 19 | `weather_cache` | 天气数据缓存 | 天气 |
| 20 | `crop_cycles` | 作物生长周期表 | 作物 |
| 21 | `chat_conversations` | 专家咨询对话表 | 聊天 |
| 22 | `chat_messages` | 聊天消息表 | 聊天 |
| 23 | `data_authorizations` | 环境数据授权表 | 授权 |
| 24 | `expert_availability` | 专家在线状态表 | 专家 |
| 25 | `user_alert_thresholds` | 用户自定义预警阈值 | 预警 |

---

## MySQL表结构

### 1. users — 用户表

**用途**：存储系统所有用户信息，支持 ADMIN/OWNER/WORKER/EXPERT 四种角色。ADMIN 仅 Web 端，APP 端只有 OWNER 和 WORKER。员工（WORKER）通过 owner_id 字段单归属一个棚主。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| username | VARCHAR(50) | NOT NULL, UNIQUE | 用户名 |
| password | VARCHAR(255) | NOT NULL | 密码（BCrypt加密） |
| phone | VARCHAR(20) | NULL | 手机号 |
| role | ENUM('ADMIN','OWNER','WORKER','EXPERT') | NOT NULL | 角色：管理员/棚主/员工/专家 |
| real_name | VARCHAR(50) | NULL | 真实姓名 |
| expert_specialty | VARCHAR(100) | NULL | 专家专业领域（仅EXPERT角色） |
| expert_status | TINYINT | NULL | 专家在线状态（仅EXPERT角色，0=离线 1=在线） |
| owner_id | BIGINT | NULL, FK→users.id | 棚主ID（仅WORKER角色，员工单归属） |
| status | TINYINT | NOT NULL, DEFAULT 1 | 状态：0=禁用 1=正常 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| uk_username | 唯一 | username | 用户名唯一 |
| idx_role | 普通 | role | 按角色查询 |
| idx_owner_id | 普通 | owner_id | 查询某棚主下所有员工 |
| idx_phone | 普通 | phone | 按手机号查询 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_users_owner | owner_id | users.id | RESTRICT |

---

### 2. user_addresses — 用户地区地址表

**用途**：存储用户注册时的五级地区地址（省-市-区/县-乡镇-村），支持默认地址标记。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| user_id | BIGINT | NOT NULL, FK→users.id | 用户ID |
| province | VARCHAR(50) | NOT NULL | 省 |
| city | VARCHAR(50) | NOT NULL | 市 |
| district | VARCHAR(50) | NOT NULL | 区/县 |
| town | VARCHAR(50) | NULL | 乡镇 |
| village | VARCHAR(50) | NULL | 村 |
| detail | VARCHAR(255) | NULL | 详细地址 |
| is_primary | TINYINT | NOT NULL, DEFAULT 0 | 是否默认地址：0=否 1=是 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_user_id | 普通 | user_id | 按用户查询地址 |
| idx_province_city | 联合 | (province, city) | 地区分布统计 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_ua_user | user_id | users.id | RESTRICT |

---

### 3. greenhouses — 大棚表

**用途**：存储大棚信息，包含五级地区地址，关联棚主。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| name | VARCHAR(100) | NOT NULL | 大棚名称 |
| location | VARCHAR(255) | NULL | 位置描述 |
| crop_type | VARCHAR(50) | NULL | 种植作物类型 |
| user_id | BIGINT | NOT NULL, FK→users.id | 棚主ID |
| province | VARCHAR(50) | NULL | 省 |
| city | VARCHAR(50) | NULL | 市 |
| district | VARCHAR(50) | NULL | 区/县 |
| town | VARCHAR(50) | NULL | 乡镇 |
| village | VARCHAR(50) | NULL | 村 |
| status | TINYINT | NOT NULL, DEFAULT 1 | 状态：0=禁用 1=正常 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_user_id | 普通 | user_id | 按棚主查询大棚 |
| idx_province_city | 联合 | (province, city) | 地区分布统计 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_gh_user | user_id | users.id | RESTRICT |

---

### 4. device_groups — 传感器组表

**用途**：管理大棚内的传感器分组，支持区域标注（如"东侧"、"西侧"、"入口"）。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| greenhouse_id | BIGINT | NOT NULL, FK→greenhouses.id | 大棚ID |
| group_name | VARCHAR(50) | NOT NULL | 组名，如"东侧传感器组" |
| zone_label | VARCHAR(50) | NULL | 区域标注，如"东侧"/"西侧"/"入口" |
| description | VARCHAR(255) | NULL | 描述 |
| sort_order | INT | NOT NULL, DEFAULT 0 | 排序 |
| status | TINYINT | NOT NULL, DEFAULT 1 | 状态：0=禁用 1=正常 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_greenhouse_id | 普通 | greenhouse_id | 按大棚查询分组 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_dg_greenhouse | greenhouse_id | greenhouses.id | RESTRICT |

---

### 5. devices — ESP32设备表

**用途**：存储 ESP32 设备注册信息，包含心跳检测、分组归属。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| device_code | VARCHAR(50) | NOT NULL, UNIQUE | 设备编号 |
| greenhouse_id | BIGINT | NOT NULL, FK→greenhouses.id | 大棚ID |
| group_id | BIGINT | NULL, FK→device_groups.id | 归属传感器组（可空） |
| name | VARCHAR(100) | NULL | 设备名称 |
| wifi_ssid | VARCHAR(100) | NULL | WiFi SSID |
| last_heartbeat | DATETIME | NULL | 最后心跳时间 |
| status | TINYINT | NOT NULL, DEFAULT 0 | 状态：0=离线 1=在线 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| uk_device_code | 唯一 | device_code | 设备编号唯一 |
| idx_greenhouse_id | 普通 | greenhouse_id | 按大棚查询设备 |
| idx_group_id | 普通 | group_id | 按分组查询设备 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_dev_greenhouse | greenhouse_id | greenhouses.id | RESTRICT |
| fk_dev_group | group_id | device_groups.id | RESTRICT |

---

### 6. sensors — 传感器配置表

**用途**：存储 ESP32 上各传感器的配置信息，包括类型、I2C地址、默认阈值。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| device_id | BIGINT | NOT NULL, FK→devices.id | 所属设备ID |
| sensor_type | ENUM('TEMP','HUMIDITY','LIGHT','CO2','O2','SOIL_TEMP','SOIL_HUMIDITY','EC','N','P','K') | NOT NULL | 传感器类型 |
| i2c_address | VARCHAR(10) | NULL | I2C地址 |
| min_threshold | DECIMAL(10,2) | NULL | 系统默认最低阈值 |
| max_threshold | DECIMAL(10,2) | NULL | 系统默认最高阈值 |
| status | TINYINT | NOT NULL, DEFAULT 1 | 状态：0=禁用 1=正常 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_device_sensor | 联合 | (device_id, sensor_type) | 查询设备下某类型传感器 |
| idx_sensor_type | 普通 | sensor_type | 按类型查询 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_sensor_device | device_id | devices.id | RESTRICT |

---

### 7. actuators — 执行器配置表

**用途**：存储执行器（继电器）配置，包括类型、GPIO引脚、当前状态。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| device_id | BIGINT | NOT NULL, FK→devices.id | 所属设备ID |
| name | VARCHAR(100) | NOT NULL | 执行器名称 |
| actuator_type | ENUM('FAN','CURTAIN','SHADE_NET','IRRIGATION','LIGHT') | NOT NULL | 执行器类型 |
| gpio_pin | INT | NOT NULL | GPIO引脚号 |
| current_state | TINYINT | NOT NULL, DEFAULT 0 | 当前状态：0=关 1=开 |
| status | TINYINT | NOT NULL, DEFAULT 1 | 状态：0=禁用 1=正常 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_device_actuator | 联合 | (device_id, actuator_type) | 查询设备下某类型执行器 |
| idx_gpio | 普通 | gpio_pin | 按GPIO查询 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_actuator_device | device_id | devices.id | RESTRICT |

---

### 8. employee_permissions — 员工权限表

**用途**：棚主为员工分配大棚访问权限和功能权限矩阵。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| employee_id | BIGINT | NOT NULL, FK→users.id | 员工ID |
| owner_id | BIGINT | NOT NULL, FK→users.id | 棚主ID |
| greenhouse_id | BIGINT | NOT NULL, FK→greenhouses.id | 授权的大棚ID |
| can_view_data | TINYINT | NOT NULL, DEFAULT 1 | 可查看数据 |
| can_control_device | TINYINT | NOT NULL, DEFAULT 0 | 可控制设备 |
| can_diagnose | TINYINT | NOT NULL, DEFAULT 0 | 可拍照诊断 |
| can_ask_expert | TINYINT | NOT NULL, DEFAULT 0 | 可求助专家 |
| can_view_alerts | TINYINT | NOT NULL, DEFAULT 1 | 可查看预警 |
| can_view_history | TINYINT | NOT NULL, DEFAULT 1 | 可查看历史数据 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| updated_at | DATETIME | NULL, ON UPDATE CURRENT_TIMESTAMP | 更新时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_employee_owner | 联合 | (employee_id, owner_id) | 查询员工在某棚主下的权限 |
| idx_greenhouse | 普通 | greenhouse_id | 查询某大棚的所有权限分配 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_ep_employee | employee_id | users.id | RESTRICT |
| fk_ep_owner | owner_id | users.id | RESTRICT |
| fk_ep_greenhouse | greenhouse_id | greenhouses.id | RESTRICT |

---

### 9. alert_rules — 预警规则表

**用途**：定义环境预警规则，支持阈值/趋势/复合/天气四种类型，可绑定到传感器组。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| greenhouse_id | BIGINT | NOT NULL, FK→greenhouses.id | 大棚ID |
| group_id | BIGINT | NULL, FK→device_groups.id | 传感器组ID（可空，为空则全棚生效） |
| sensor_type | VARCHAR(20) | NOT NULL | 传感器类型 |
| rule_type | ENUM('THRESHOLD','TREND','COMPOSITE','WEATHER') | NOT NULL | 规则类型 |
| condition_json | JSON | NOT NULL | 条件配置（JSON格式） |
| alert_level | ENUM('INFO','WARNING','CRITICAL') | NOT NULL | 预警级别 |
| scene_id | BIGINT | NULL, FK→scenes.id | 关联场景联动（可空） |
| enabled | TINYINT | NOT NULL, DEFAULT 1 | 是否启用：0=禁用 1=启用 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_greenhouse_sensor | 联合 | (greenhouse_id, sensor_type) | 查询大棚下某类型的规则 |
| idx_enabled | 普通 | enabled | 查询启用的规则 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_ar_greenhouse | greenhouse_id | greenhouses.id | RESTRICT |
| fk_ar_group | group_id | device_groups.id | RESTRICT |
| fk_ar_scene | scene_id | scenes.id | RESTRICT |

---

### 10. alerts — 预警记录表

**用途**：存储预警触发记录，包含传感器值、天气信息、已读状态。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| greenhouse_id | BIGINT | NOT NULL, FK→greenhouses.id | 大棚ID |
| group_id | BIGINT | NULL, FK→device_groups.id | 传感器组ID（定位预警区域） |
| alert_rule_id | BIGINT | NULL, FK→alert_rules.id | 触发的预警规则ID |
| level | ENUM('INFO','WARNING','CRITICAL') | NOT NULL | 预警级别 |
| title | VARCHAR(200) | NOT NULL | 预警标题 |
| content | TEXT | NULL | 预警内容 |
| sensor_value | DECIMAL(10,2) | NULL | 触发时的传感器值 |
| weather_info | VARCHAR(255) | NULL | 天气信息（辅助预判） |
| read_status | TINYINT | NOT NULL, DEFAULT 0 | 已读状态：0=未读 1=已读 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_greenhouse_created | 联合 | (greenhouse_id, created_at) | 按大棚+时间查询预警 |
| idx_read_status | 普通 | read_status | 查询未读预警 |
| idx_alert_rule | 普通 | alert_rule_id | 按规则查询预警记录 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_alert_greenhouse | greenhouse_id | greenhouses.id | RESTRICT |
| fk_alert_group | group_id | device_groups.id | RESTRICT |
| fk_alert_rule | alert_rule_id | alert_rules.id | SET NULL |

---

### 11. scenes — 场景联动表

**用途**：定义场景联动规则，包含触发条件和多设备执行动作。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| name | VARCHAR(100) | NOT NULL | 场景名称，如"高温通风" |
| trigger_condition | JSON | NOT NULL | 触发条件（JSON格式） |
| actions_json | JSON | NOT NULL | 执行动作列表（JSON格式） |
| greenhouse_id | BIGINT | NOT NULL, FK→greenhouses.id | 大棚ID |
| enabled | TINYINT | NOT NULL, DEFAULT 1 | 是否启用：0=禁用 1=启用 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_greenhouse_enabled | 联合 | (greenhouse_id, enabled) | 查询大棚下启用的场景 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_scene_greenhouse | greenhouse_id | greenhouses.id | RESTRICT |

---

### 12. diagnostic_records — 病虫害诊断记录

**用途**：存储病虫害图像诊断记录，包含识别结果、置信度、防治方案。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| user_id | BIGINT | NOT NULL, FK→users.id | 用户ID |
| greenhouse_id | BIGINT | NULL, FK→greenhouses.id | 大棚ID |
| image_path | VARCHAR(255) | NOT NULL | 图片存储路径 |
| disease_name | VARCHAR(100) | NULL | 病害名称 |
| confidence | DECIMAL(5,2) | NULL | 置信度（0.00-100.00） |
| treatment | TEXT | NULL | 防治方案 |
| recognition_engine | VARCHAR(20) | NULL | 识别引擎：baidu/resnet |
| expert_consulted | TINYINT | NOT NULL, DEFAULT 0 | 是否转求助专家：0=否 1=是 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_user_created | 联合 | (user_id, created_at) | 按用户+时间查询诊断历史 |
| idx_greenhouse | 普通 | greenhouse_id | 按大棚查询 |
| idx_expert_consulted | 普通 | expert_consulted | 查询转专家的记录 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_dr_user | user_id | users.id | RESTRICT |
| fk_dr_greenhouse | greenhouse_id | greenhouses.id | RESTRICT |

---

### 13. qa_records — 问答记录

**用途**：存储 AI 问答记录，支持文字和语音两种输入方式。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| user_id | BIGINT | NOT NULL, FK→users.id | 用户ID |
| question | TEXT | NOT NULL | 问题内容 |
| answer | TEXT | NULL | 回答内容 |
| input_type | ENUM('TEXT','VOICE') | NOT NULL | 输入类型 |
| asr_engine | VARCHAR(20) | NULL | 语音识别引擎：xunfei/whisper（VOICE时填写） |
| sources | JSON | NULL | 引用来源（JSON格式） |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_user_created | 联合 | (user_id, created_at) | 按用户+时间查询问答历史 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_qa_user | user_id | users.id | RESTRICT |

---

### 14. control_logs — 设备控制日志

**用途**：记录所有设备控制操作，支持手动/场景/预警三种触发来源。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| user_id | BIGINT | NULL, FK→users.id | 操作用户ID（系统自动触发时可空） |
| actuator_id | BIGINT | NOT NULL, FK→actuators.id | 执行器ID |
| action | ENUM('ON','OFF','SCENE') | NOT NULL | 动作类型 |
| source | ENUM('MANUAL','SCENE','ALERT') | NOT NULL | 触发来源 |
| scene_id | BIGINT | NULL, FK→scenes.id | 关联场景ID（SCENE时填写） |
| group_id | BIGINT | NULL, FK→device_groups.id | 传感器组ID |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_actuator_created | 联合 | (actuator_id, created_at) | 按执行器+时间查询日志 |
| idx_user_created | 联合 | (user_id, created_at) | 按用户+时间查询日志 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_cl_user | user_id | users.id | SET NULL |
| fk_cl_actuator | actuator_id | actuators.id | RESTRICT |
| fk_cl_scene | scene_id | scenes.id | SET NULL |
| fk_cl_group | group_id | device_groups.id | SET NULL |

---

### 15. knowledge_documents — 知识库文档表

**用途**：存储上传的农业知识文档元数据，跟踪向量化状态。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| title | VARCHAR(200) | NOT NULL | 文档标题 |
| category | VARCHAR(50) | NULL | 分类 |
| crop_type | VARCHAR(50) | NULL | 适用作物类型 |
| content | LONGTEXT | NULL | 文档内容 |
| file_path | VARCHAR(255) | NULL | 文件存储路径 |
| chunk_count | INT | NOT NULL, DEFAULT 0 | 切片数量 |
| vector_indexed | TINYINT | NOT NULL, DEFAULT 0 | 是否已向量化：0=未向量化 1=已向量化 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_category | 普通 | category | 按分类查询 |
| idx_crop_type | 普通 | crop_type | 按作物类型查询 |
| idx_vector_indexed | 普通 | vector_indexed | 查询待向量化的文档 |

---

### 16. dialect_corpus — 方言语料表

**用途**：存储河北方言语音语料，支持训练/测试分类管理。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| audio_path | VARCHAR(255) | NOT NULL | 音频文件路径 |
| dialect_text | VARCHAR(500) | NOT NULL | 方言文本 |
| mandarin_text | VARCHAR(500) | NOT NULL | 普通话翻译 |
| category | VARCHAR(50) | NULL | 分类 |
| duration_sec | DECIMAL(10,2) | NULL | 音频时长（秒） |
| usage_type | ENUM('TEST','TRAIN','BOTH') | NOT NULL | 用途类型：测试/训练/两者 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_usage_type | 普通 | usage_type | 按用途筛选语料 |
| idx_category | 普通 | category | 按分类查询 |

---

### 17. growth_assessments — 长势评估记录

**用途**：存储作物长势评估记录，关联生长周期，包含株高、叶面积、叶色等指标。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| greenhouse_id | BIGINT | NOT NULL, FK→greenhouses.id | 大棚ID |
| crop_cycle_id | BIGINT | NULL, FK→crop_cycles.id | 生长周期ID（可空） |
| image_path | VARCHAR(255) | NULL | 截帧图片路径 |
| growth_stage | VARCHAR(50) | NULL | 生长阶段 |
| plant_height | DECIMAL(10,2) | NULL | 株高（cm） |
| leaf_area | DECIMAL(10,2) | NULL | 叶面积（cm²） |
| leaf_color | VARCHAR(50) | NULL | 叶色 |
| health_score | DECIMAL(5,2) | NULL | 健康评分 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_greenhouse_created | 联合 | (greenhouse_id, created_at) | 按大棚+时间查询长势历史 |
| idx_crop_cycle | 普通 | crop_cycle_id | 按生长周期查询 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_ga_greenhouse | greenhouse_id | greenhouses.id | RESTRICT |
| fk_ga_crop_cycle | crop_cycle_id | crop_cycles.id | SET NULL |

---

### 18. health_assessments — 多模态健康综合评估

**用途**：存储环境+图像融合后的综合健康评分。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| greenhouse_id | BIGINT | NOT NULL, FK→greenhouses.id | 大棚ID |
| env_score | DECIMAL(5,2) | NULL | 环境健康分（0-100） |
| visual_score | DECIMAL(5,2) | NULL | 视觉健康分（0-100） |
| weather_risk | VARCHAR(50) | NULL | 天气风险等级 |
| overall_score | DECIMAL(5,2) | NULL | 综合健康评分（0-100） |
| analysis_json | JSON | NULL | 详细分析数据（JSON） |
| recommendations | TEXT | NULL | 建议措施 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_greenhouse_created | 联合 | (greenhouse_id, created_at) | 按大棚+时间查询健康历史 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_ha_greenhouse | greenhouse_id | greenhouses.id | RESTRICT |

---

### 19. weather_cache — 天气数据缓存

**用途**：缓存和风天气API拉取的天气数据，每3小时更新。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| location | VARCHAR(100) | NOT NULL | 位置标识 |
| forecast_json | JSON | NULL | 完整预报数据（JSON） |
| temperature | DECIMAL(5,2) | NULL | 温度（°C） |
| humidity | DECIMAL(5,2) | NULL | 湿度（%） |
| weather_code | VARCHAR(20) | NULL | 天气代码 |
| wind_speed | DECIMAL(5,2) | NULL | 风速（m/s） |
| forecast_time | DATETIME | NULL | 预报时间 |
| updated_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 更新时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_location_updated | 联合 | (location, updated_at) | 按位置查询最新天气 |

---

### 20. crop_cycles — 作物生长周期表

**用途**：管理作物种植记录，支持阶段自动估算和手动修正。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| greenhouse_id | BIGINT | NOT NULL, FK→greenhouses.id | 大棚ID |
| crop_type | VARCHAR(50) | NOT NULL | 作物名称，如"番茄"/"黄瓜" |
| variety | VARCHAR(50) | NULL | 品种 |
| planting_date | DATE | NOT NULL | 种植日期 |
| expected_harvest_date | DATE | NULL | 预计收获日期 |
| actual_harvest_date | DATE | NULL | 实际收获日期 |
| current_stage | VARCHAR(20) | NOT NULL, DEFAULT '育苗期' | 当前阶段：育苗期/生长期/开花期/结果期/收获期 |
| stage_source | ENUM('AUTO','MANUAL') | NOT NULL, DEFAULT 'AUTO' | 阶段来源：自动估算/手动设置 |
| status | ENUM('ACTIVE','COMPLETED','CANCELLED') | NOT NULL, DEFAULT 'ACTIVE' | 状态 |
| notes | TEXT | NULL | 备注 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| updated_at | DATETIME | NULL, ON UPDATE CURRENT_TIMESTAMP | 更新时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_greenhouse_status | 联合 | (greenhouse_id, status) | 查询大棚下进行中的周期 |
| idx_crop_type | 普通 | crop_type | 按作物类型查询 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_cc_greenhouse | greenhouse_id | greenhouses.id | RESTRICT |

---

### 21. chat_conversations — 专家咨询对话表

**用途**：管理用户与专家之间的咨询对话会话。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| user_id | BIGINT | NOT NULL, FK→users.id | 用户/员工ID |
| expert_id | BIGINT | NOT NULL, FK→users.id | 专家ID |
| greenhouse_id | BIGINT | NULL, FK→greenhouses.id | 关联大棚 |
| subject | VARCHAR(255) | NOT NULL | 咨询主题 |
| status | ENUM('WAITING','ACTIVE','CLOSED') | NOT NULL, DEFAULT 'WAITING' | 状态：等待中/进行中/已关闭 |
| diagnostic_id | BIGINT | NULL, FK→diagnostic_records.id | 关联诊断记录（可空） |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| closed_at | DATETIME | NULL | 关闭时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_user_expert | 联合 | (user_id, expert_id) | 查询用户与专家的对话 |
| idx_expert_status | 联合 | (expert_id, status) | 查询专家的待处理对话 |
| idx_status_created | 联合 | (status, created_at) | 按状态+时间查询 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_conv_user | user_id | users.id | RESTRICT |
| fk_conv_expert | expert_id | users.id | RESTRICT |
| fk_conv_greenhouse | greenhouse_id | greenhouses.id | RESTRICT |
| fk_conv_diagnostic | diagnostic_id | diagnostic_records.id | SET NULL |

---

### 22. chat_messages — 聊天消息表

**用途**：存储对话中的消息，支持文字/图片/视频/环境快照四种类型。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| conversation_id | BIGINT | NOT NULL, FK→chat_conversations.id | 对话ID |
| sender_id | BIGINT | NOT NULL, FK→users.id | 发送者ID |
| sender_type | ENUM('USER','EXPERT') | NOT NULL | 发送者身份 |
| message_type | ENUM('TEXT','IMAGE','VIDEO','ENV_SNAPSHOT') | NOT NULL | 消息类型 |
| content | TEXT | NULL | 文字内容（TEXT时使用） |
| file_path | VARCHAR(255) | NULL | 文件路径（IMAGE/VIDEO时使用） |
| snapshot_data | JSON | NULL | 环境快照数据（ENV_SNAPSHOT时使用） |
| read_status | TINYINT | NOT NULL, DEFAULT 0 | 已读状态：0=未读 1=已读 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_conversation_created | 联合 | (conversation_id, created_at) | 按对话+时间排序查询消息 |
| idx_sender | 普通 | sender_id | 按发送者查询 |
| idx_read_status | 普通 | read_status | 查询未读消息 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_cm_conversation | conversation_id | chat_conversations.id | RESTRICT |
| fk_cm_sender | sender_id | users.id | RESTRICT |

---

### 23. data_authorizations — 环境数据授权表

**用途**：管理专家查看大棚数据的授权，支持7天有效期和用户随时撤销。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| expert_id | BIGINT | NOT NULL, FK→users.id | 专家ID |
| user_id | BIGINT | NOT NULL, FK→users.id | 用户ID（授权方） |
| greenhouse_id | BIGINT | NOT NULL, FK→greenhouses.id | 授权的大棚ID |
| status | ENUM('PENDING','APPROVED','REJECTED','EXPIRED','REVOKED') | NOT NULL, DEFAULT 'PENDING' | 状态 |
| requested_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 请求时间 |
| approved_at | DATETIME | NULL | 同意时间 |
| expires_at | DATETIME | NULL | 过期时间（同意后+7天） |
| revoked_at | DATETIME | NULL | 撤销时间 |
| revoked_by | BIGINT | NULL, FK→users.id | 撤销操作人 |
| reason | VARCHAR(255) | NULL | 请求理由 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_expert_user_greenhouse | 联合 | (expert_id, user_id, greenhouse_id) | 查询专家对某用户某大棚的授权 |
| idx_status_expires | 联合 | (status, expires_at) | 定时任务扫描过期授权 |
| idx_user_status | 联合 | (user_id, status) | 用户查看授权状态 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_da_expert | expert_id | users.id | RESTRICT |
| fk_da_user | user_id | users.id | RESTRICT |
| fk_da_greenhouse | greenhouse_id | greenhouses.id | RESTRICT |
| fk_da_revoked_by | revoked_by | users.id | SET NULL |

---

### 24. expert_availability — 专家在线状态表

**用途**：管理专家的在线状态和并发咨询数。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| expert_id | BIGINT | NOT NULL, UNIQUE, FK→users.id | 专家ID |
| is_online | TINYINT | NOT NULL, DEFAULT 0 | 是否在线：0=离线 1=在线 |
| last_active_at | DATETIME | NULL | 最后活跃时间 |
| max_concurrent | INT | NOT NULL, DEFAULT 5 | 最大同时咨询数 |
| current_count | INT | NOT NULL, DEFAULT 0 | 当前进行中咨询数 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| uk_expert_id | 唯一 | expert_id | 专家ID唯一 |
| idx_is_online | 普通 | is_online | 查询在线专家 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_ea_expert | expert_id | users.id | RESTRICT |

---

### 25. user_alert_thresholds — 用户自定义预警阈值

**用途**：用户（棚主）自定义各传感器参数的最高/最低预警值，优先级高于系统默认阈值。

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| user_id | BIGINT | NOT NULL, FK→users.id | 设置者ID（通常为棚主） |
| greenhouse_id | BIGINT | NOT NULL, FK→greenhouses.id | 大棚ID |
| group_id | BIGINT | NULL, FK→device_groups.id | 传感器组ID（可空，为空则全棚生效） |
| sensor_type | ENUM('TEMP','HUMIDITY','LIGHT','CO2','O2','SOIL_TEMP','SOIL_HUMIDITY','EC','N','P','K') | NOT NULL | 传感器类型 |
| min_threshold | DECIMAL(10,2) | NULL | 最低预警值（可空，为空则不设下限） |
| max_threshold | DECIMAL(10,2) | NULL | 最高预警值（可空，为空则不设上限） |
| enabled | TINYINT | NOT NULL, DEFAULT 1 | 是否启用：0=禁用 1=启用 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| updated_at | DATETIME | NULL, ON UPDATE CURRENT_TIMESTAMP | 更新时间 |

**索引**：

| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| PRIMARY | 主键 | id | 主键索引 |
| idx_user_greenhouse_sensor | 联合 | (user_id, greenhouse_id, sensor_type) | 查询用户某大棚某类型的阈值 |
| idx_greenhouse_sensor | 联合 | (greenhouse_id, sensor_type) | 预警引擎按大棚+类型查询阈值 |

**外键**：

| 外键名 | 字段 | 引用 | 删除策略 |
|--------|------|------|----------|
| fk_uat_user | user_id | users.id | RESTRICT |
| fk_uat_greenhouse | greenhouse_id | greenhouses.id | RESTRICT |
| fk_uat_group | group_id | device_groups.id | RESTRICT |

---

### 外键关系总图

```
users (id)
├── users.owner_id ──────────────→ users.id (员工→棚主)
├── user_addresses.user_id ──────→ users.id
├── greenhouses.user_id ─────────→ users.id (棚主)
├── employee_permissions.employee_id → users.id
├── employee_permissions.owner_id ───→ users.id
├── diagnostic_records.user_id ──→ users.id
├── qa_records.user_id ──────────→ users.id
├── control_logs.user_id ────────→ users.id (可空)
├── chat_conversations.user_id ──→ users.id
├── chat_conversations.expert_id → users.id
├── chat_messages.sender_id ─────→ users.id
├── data_authorizations.expert_id → users.id
├── data_authorizations.user_id ─→ users.id
├── data_authorizations.revoked_by → users.id (可空)
├── expert_availability.expert_id → users.id
├── user_alert_thresholds.user_id → users.id

greenhouses (id)
├── device_groups.greenhouse_id ─→ greenhouses.id
├── devices.greenhouse_id ───────→ greenhouses.id
├── employee_permissions.greenhouse_id → greenhouses.id
├── alert_rules.greenhouse_id ───→ greenhouses.id
├── alerts.greenhouse_id ────────→ greenhouses.id
├── scenes.greenhouse_id ────────→ greenhouses.id
├── diagnostic_records.greenhouse_id → greenhouses.id
├── growth_assessments.greenhouse_id → greenhouses.id
├── health_assessments.greenhouse_id → greenhouses.id
├── crop_cycles.greenhouse_id ───→ greenhouses.id
├── chat_conversations.greenhouse_id → greenhouses.id
├── data_authorizations.greenhouse_id → greenhouses.id
├── user_alert_thresholds.greenhouse_id → greenhouses.id

device_groups (id)
├── devices.group_id ─────────────→ device_groups.id
├── alert_rules.group_id ─────────→ device_groups.id
├── alerts.group_id ─────────────→ device_groups.id
├── control_logs.group_id ────────→ device_groups.id
├── user_alert_thresholds.group_id → device_groups.id

devices (id)
├── sensors.device_id ────────────→ devices.id
├── actuators.device_id ──────────→ devices.id

actuators (id)
└── control_logs.actuator_id ────→ actuators.id

scenes (id)
├── alert_rules.scene_id ─────────→ scenes.id
└── control_logs.scene_id ────────→ scenes.id

alert_rules (id)
└── alerts.alert_rule_id ────────→ alert_rules.id

diagnostic_records (id)
└── chat_conversations.diagnostic_id → diagnostic_records.id

crop_cycles (id)
└── growth_assessments.crop_cycle_id → crop_cycles.id
```

---

## InfluxDB设计

### Measurement: sensor_data

**用途**：存储所有传感器上报的时序读数，通过 tags 实现多维度过滤和聚合。

| 属性 | 说明 |
|------|------|
| Measurement | `sensor_data` |
| Retention Policy | `autogen`（永久保留，可配置30天自动清理） |
| 写入频率 | 每30秒（每个传感器） |
| 预估写入量 | 11种传感器 × 3-5组 × 每分钟2条 = 66-110条/分钟 |

#### Tags（标签，用于过滤和分组）

| Tag Key | 类型 | 说明 | 示例值 |
|---------|------|------|--------|
| greenhouse_id | STRING | 大棚ID | "1" |
| device_id | STRING | ESP32设备ID | "3" |
| group_id | STRING | 传感器组ID | "2" |
| sensor_type | STRING | 传感器类型 | "TEMP" |

#### Fields（字段，存储实际数值）

| Field Key | 类型 | 说明 |
|-----------|------|------|
| value | FLOAT | 传感器读数 |

#### 查询示例

```sql
-- 查询某组传感器最近1小时的温度数据
SELECT value FROM sensor_data
WHERE greenhouse_id = '1' AND group_id = '2' AND sensor_type = 'TEMP'
  AND time >= now() - 1h

-- 查询全棚各组温度对比（聚合）
SELECT MEAN(value) AS mean_temp FROM sensor_data
WHERE greenhouse_id = '1' AND sensor_type = 'TEMP'
  AND time >= now() - 1h
GROUP BY group_id

-- 查询全棚平均温度
SELECT MEAN(value) FROM sensor_data
WHERE greenhouse_id = '1' AND sensor_type = 'TEMP'
  AND time >= now() - 1h

-- 查询今天CO2/O2浓度变化（按组）
SELECT value FROM sensor_data
WHERE greenhouse_id = '1' AND sensor_type = 'CO2'
  AND time >= today()
GROUP BY group_id

-- 过去7天土壤温度和EC平均值（按组）
SELECT MEAN(value) FROM sensor_data
WHERE greenhouse_id = '1' AND (sensor_type = 'SOIL_TEMP' OR sensor_type = 'EC')
  AND time >= now() - 7d
GROUP BY group_id, sensor_type

-- 多组数据对比（同参数跨组曲线）
SELECT value FROM sensor_data
WHERE greenhouse_id = '1' AND sensor_type = 'TEMP'
  AND time >= now() - 6h
GROUP BY group_id
```

#### Retention Policy 配置

```bash
# 创建30天自动清理策略
influx bucket update --name sensor_data --retention 30d

# 查询当前retention
influx bucket list --name sensor_data
```

---

## Chroma设计

### Collection: agricultural_knowledge

**用途**：存储农业知识库文档的向量化切片，支持基于语义相似度的RAG检索。

| 属性 | 说明 |
|------|------|
| Collection Name | `agricultural_knowledge` |
| Embedding Model | SiliconFlow `BAAI/bge-m3` |
| Vector Dimension | 1024 |
| Chunk Size | 约500字/片 |
| 预估文档量 | 200-500篇 → 约1000-5000个切片 |

#### Document 结构

```
documents: 知识库文档切片（每片约500字）
  - 来源：知识库文档经过 Spring AI 自动分片
  - 分片策略：按段落语义边界切分，保留上下文
```

#### Embeddings

```
embeddings: Spring AI 自动生成
  - 模型：SiliconFlow bge-m3（OpenAI兼容格式）
  - 维度：1024
  - 调用时机：
    - 文档首次上传 → 全量向量化
    - 新增文档 → 增量向量化
    - 查询时 → 实时向量化查询文本
```

#### Metadata 结构

| Metadata Key | 类型 | 说明 | 示例值 |
|-------------|------|------|--------|
| document_id | STRING | 关联的 knowledge_documents 表 ID | "15" |
| title | STRING | 文档标题 | "番茄晚疫病防治技术" |
| category | STRING | 分类 | "病害防治" |
| crop_type | STRING | 适用作物 | "番茄" |
| chunk_index | INT | 切片序号（从0开始） | 2 |

#### 检索流程

```
用户提问 → 文本向量化(SiliconFlow bge-m3)
  → Chroma向量检索(top_k=5, 余弦相似度)
  → 返回相关文档切片 + 相似度分数
  → 组装提示词: [切片1, 切片2, ...] + 用户问题
  → DeepSeek生成答案
  → 返回 {答案 + 引用来源}
```

#### Collection 创建示例（Spring AI 配置）

```java
@Bean
public VectorStore chromaVectorStore(EmbeddingModel embeddingModel, ChromaApi chromaApi) {
    return new ChromaVectorStore(embeddingModel, chromaApi, "agricultural_knowledge");
}
```

---

## 数据迁移策略

### 数据库初始化

使用 Flyway 管理数据库版本迁移：

```
src/main/resources/db/migration/
├── V1__init_users.sql
├── V2__init_user_addresses.sql
├── V3__init_greenhouses.sql
├── V4__init_device_groups.sql
├── V5__init_devices.sql
├── V6__init_sensors.sql
├── V7__init_actuators.sql
├── V8__init_employee_permissions.sql
├── V9__init_alert_rules.sql
├── V10__init_alerts.sql
├── V11__init_scenes.sql
├── V12__init_diagnostic_records.sql
├── V13__init_qa_records.sql
├── V14__init_control_logs.sql
├── V15__init_knowledge_documents.sql
├── V16__init_dialect_corpus.sql
├── V17__init_growth_assessments.sql
├── V18__init_health_assessments.sql
├── V19__init_weather_cache.sql
├── V20__init_crop_cycles.sql
├── V21__init_chat_conversations.sql
├── V22__init_chat_messages.sql
├── V23__init_data_authorizations.sql
├── V24__init_expert_availability.sql
└── V25__init_user_alert_thresholds.sql
```

### 迁移原则

1. **禁止直接删除表或字段**：必须通过迁移脚本新增字段并废弃旧字段
2. **禁止在代码中直接执行DDL**：所有表结构变更通过 Flyway 脚本管理
3. **禁止修改已有字段类型或名称**：只能新增字段
4. **禁止跳过外键约束直接插入数据**
5. **禁止在生产环境手动修改数据库数据**
6. **禁止级联删除（CASCADE）**：所有外键使用 RESTRICT 或 SET NULL
7. **数据备份**：MySQL 每日 mysqldump，InfluxDB 每日 influx backup，Chroma 持久化卷映射

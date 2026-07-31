---
id: TASK-SEC02
title: MQTT Broker 认证加固
module: mqtt
type: task
priority: critical
status: completed
created: 2026-08-01
started: 2026-08-01
completed: 2026-08-01
assignee: AI助手
related_docs: [CODE_REVIEW.md, mosquitto.conf]
tags: [安全加固, MQTT, Mosquitto, 认证]
---

# MQTT Broker 认证加固

---

## 任务描述

### 背景
代码审查发现 Mosquitto MQTT Broker 允许匿名连接（`allow_anonymous true`）。攻击者可向 Broker 注入伪造传感器数据或劫持设备控制指令，造成虚假告警或设备误操作。

### 目标
禁止 MQTT 匿名连接，启用用户名/密码认证。同步更新后端配置和两个 Python 模拟器脚本。

### 范围
Mosquitto 配置、后端 Spring Boot 配置、Python 模拟器脚本。

---

## 验收标准

1. [x] `mosquitto.conf` 中 `allow_anonymous` 设为 `false`
2. [x] 配置密码文件和 ACL 文件路径
3. [x] `application-dev.yml` 中 MQTT 用户名/密码改为 `greenhouse / greenhouse_dev`
4. [x] `simulator/devices.json` 中 MQTT 凭据同步更新
5. [x] `tools/sensor_simulator.py` 新增 `username_pw_set()` 认证调用
6. [x] 后端 `MqttConfig.java` 无需修改（已从配置读取凭据）

---

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `mosquitto.conf` | 修改 | `allow_anonymous false`，新增 `password_file` 和 `acl_file` |
| `backend/.../application-dev.yml` | 修改 | MQTT username/password 默认值 |
| `simulator/devices.json` | 修改 | 添加 MQTT username/password |
| `tools/sensor_simulator.py` | 修改 | 新增 `MQTT_USERNAME`/`MQTT_PASSWORD` 环境变量 + `username_pw_set()` 调用 |

---

## 阻塞记录
无阻塞

---

## 备注
- 首次启动前需在 Mosquitto 容器内创建密码文件：`docker exec -it greenhouse-mosquitto mosquitto_passwd -c /mosquitto/config/passwd greenhouse`
- 后端 `MqttConfig.java` 使用 `@ConfigurationProperties(prefix = "mqtt")` 自动绑定，username/password 为空时跳过认证（向后兼容）
- 模拟器脚本通过环境变量 `MQTT_USER` / `MQTT_PASSWORD` 可覆盖默认凭据

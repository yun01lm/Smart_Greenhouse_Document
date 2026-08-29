---
id: TECH-009
title: 设备生命周期与自助注册方案设计文档
type: tech-design
module: device-lifecycle
tags:
  - MQTT
  - ESP32
  - 设备注册
  - 固件
  - InfluxDB
  - 自助注册
status: draft
created: 2026-08-29
last_modified: 2026-08-29
author: AI助手
related_docs:
  - /document/docs/tech/perception-layer.md
  - /document/docs/tech/realtime-comm.md
  - /document/docs/api/sensor/sensor-api.md
---

## 1. 背景与目标

当前设备接入流程是**「先注册、后烧录」**：用户先在 Web/APP 设备管理里添加设备（拿到数据库主键 `devices.id`），再把该 id 写进固件编译烧录。本方案目标是改为**「先烧录、后注册」**：

> 固件出厂/烧录时只需写死 WiFi、broker、`greenhouseId`、`deviceSn`、传感器类型；设备上电即开始上报；用户随后在 Web 端自助添加设备（填同一个 SN），注册后数据自动续流，无需重新烧录。

核心设计思想：**设备身份解析从「数据库主键 deviceId」改为「SN 优先」**。SN 本身就在 MQTT topic 里（`greenhouse/{ghId}/device/{sn}`），天然可用作路由与匹配键。

## 2. 现状与问题分析

### 2.1 devices 表结构（设备静态档案）

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | BIGINT 自增 | 设备主键，即固件上报 JSON 里的 `deviceId` |
| `name` | VARCHAR(100) | 设备名称，同大棚内唯一 |
| `device_sn` | VARCHAR(50) NOT NULL | 设备编号，用户自定义，**同一大棚下唯一** |
| `device_type` | ENUM(`SENSOR`,`CONTROLLER`) | 传感器 / 控制器 |
| `sensor_type` | VARCHAR(30) | 仅传感器用：`TEMPERATURE / HUMIDITY / LIGHT / CO2 / SOIL_MOISTURE / SOIL_TEMP / SOIL_PH / WIND_SPEED`；控制器为 NULL |
| `status` | ENUM(`ONLINE`,`OFFLINE`,`ALARM`) | 由上报数据驱动，默认 OFFLINE |
| `greenhouse_id` | BIGINT NOT NULL | 所属大棚 |
| `last_data_time` | DATETIME(6) | 上次数据上报时间 |
| `last_value` | VARCHAR(100) | 传感器存最新读数；控制器存最新开关状态 |
| `mqtt_topic` | VARCHAR(200) | **`@PrePersist` 自动生成**：`greenhouse/{greenhouseId}/device/{deviceSn}` |
| `description` | VARCHAR(500) | 描述 |
| `install_location` | VARCHAR(200) | 安装位置 |
| `created_at` / `updated_at` | DATETIME | 时间戳 |

注册接口 `POST /api/v1/greenhouses/{greenhouseId}/devices`，入参 `DeviceRequest`：
`name`(必填)、`deviceSn`(必填)、`deviceType`(必填)、`sensorType`(传感器必填)、`installLocation`、`description`。
唯一性校验：同大棚下 `deviceSn`、`name` 均不可重复。

### 2.2 当前 MQTT 处理链路（为什么先烧录必失败）

`MqttSubscriber.messageArrived()` 现状（`backend/.../module/mqtt/MqttSubscriber.java`）：

```java
Long greenhouseId = root.get("greenhouseId").asLong();
Long deviceId = root.get("deviceId").asLong();   // ← 强依赖数据库主键
```

- payload 缺 `deviceId` 或该 id 在库中不存在 → 抛异常 → 数据被丢弃，仅打 error 日志；
- 设备未注册时没有任何"待注册"缓冲，上电到注册之间的数据全部丢失；
- 因此固件必须先知道自己的 DB id，`先烧录后注册`无法成立。

### 2.3 相关约束

- `deviceSn` 同大棚唯一（`existsByGreenhouseIdAndDeviceSn`），故 `(greenhouseId, deviceSn)` 可作为无歧义解析键；
- broker 账号 `greenhouse / greenhouse_dev` 为设备共享凭据（mosquitto password_file 配置），任何持账号者都可向任意大棚 topic 上报 —— 现状即如此，本方案不放大该风险；
- 控制指令下行 topic：`greenhouse/{ghId}/device/{sn}/command`，payload `{"action":"ON","timestamp":...}`（QoS 1）。

## 3. 目标方案总览

```
┌──────────┐  ① 烧录：只写死 WiFi/broker/ghId/SN/类型   ┌──────────────┐
│  ESP32   │ ──── 上电即上报（周期 30s）─────────────→ │ MqttSubscriber│
└──────────┘    topic: greenhouse/1/device/TEMP-001      └──────┬───────┘
                                                                 │ ② 解析顺序：
                                                                 │    payload.deviceId 存在→用
                                                                 │    → (ghId, SN) 查库→用
                                                                 │    → 未注册：入缓冲+日志
                                                                 ▼
┌──────────┐  ③ 用户 Web 添加设备（填同一 SN）          ┌──────────────┐
│  用户    │ ──── POST /api/v1/greenhouses/1/devices →│ createDevice  │
└──────────┘    自动带出 SN，只需填名称/类型             └──────┬───────┘
                                                                 │ ④ 注册成功 → 缓冲数据补写 InfluxDB
                                                                 ▼
                                                       下一帧上报 → ONLINE、
                                                       InfluxDB、WebSocket 推送、
                                                       告警引擎，全链路生效
```

## 4. 详细设计

### 4.1 后端：MQTT 解析改造（核心，P1）

**文件**：`MqttSubscriber.java`、`SensorDataService.java`（新增解析方法）、`DeviceRepository` 注入。

**解析顺序**（`messageArrived` 内）：
1. `payload.deviceId` 存在且 `deviceRepository.findById` 命中 → 使用该 id；
2. 否则取 `payload.deviceSn`（缺省从 topic `greenhouse/{ghId}/device/{sn}` 解析），按 `(greenhouseId, deviceSn)` 查库 → 命中则使用；
3. 仍查不到 → **未注册分支**：写入待注册缓冲 + warn 日志，本次数据不写 InfluxDB、不推 WebSocket、不进告警引擎。

**待注册缓冲**（内存实现，注册即续流）：
- 结构：`ConcurrentHashMap<String, PendingDeviceBuffer>`，key = `greenhouseId + ":" + deviceSn`；
- `PendingDeviceBuffer`：`deviceSn`、`greenhouseId`、`sensorType`、`首次上报时间`、`最近数值`、`数据队列`（保留最近 100 条 `(value, timestamp)`，TTL 24h 过期清理）；
- 上报限流：同一未注册设备 5 秒内只记 1 条日志，防刷日志。

**注册补写**（P4 增强）：`createDevice` 成功后调用 `flushPendingData(greenhouseId, deviceSn, deviceId)`，把缓冲中的 100 条数据补写 InfluxDB → 历史曲线从"上电时刻"起完整。

**对外能力**（供 Web 待注册入口用）：
- `GET /api/v1/admin/devices/pending`：列出缓冲中见过的未注册设备（SN、大棚、首次上报时间、最近数值、sensorType）；
- 注册接口不变（前端零改动即可跑通基础流程）。

### 4.2 固件：ESP32 模板（P2，Arduino IDE 版）

**config.h（烧录时只需改这里）**：

```c
#define WIFI_SSID      "xxx"
#define WIFI_PASS      "xxx"
#define MQTT_HOST      "192.168.x.x"   // 生产 = 服务器公网 IP
#define MQTT_PORT      1883
#define MQTT_USER      "greenhouse"
#define MQTT_PASS      "greenhouse_dev"
#define GREENHOUSE_ID  1
#define DEVICE_SN      "TEMP-001"      // 出厂/烧录时定，用户注册时填同一个
#define SENSOR_TYPE    "TEMPERATURE"   // 控制器设备可留空
#define REPORT_SEC     30              // 上报周期
```

**上报 payload（deviceId 可省略）**：

```json
{"greenhouseId":1,"deviceSn":"TEMP-001","sensorType":"TEMPERATURE","value":25.6,"timestamp":1753088400000}
```

**控制器心跳**：

```json
{"greenhouseId":1,"deviceSn":"PUMP-001","deviceType":"CONTROLLER","timestamp":1753088400000}
```

**命令订阅**：`greenhouse/+/device/+/command`，收到 `{"action":"ON","timestamp":...}` 后驱动继电器/水泵/风机，建议 QoS 1。

**上电时序**：WiFi 连接（断线重连+指数退避）→ MQTT 连接（重连+心跳 keepalive 60s）→ 订阅 command → 立即上报一帧 → 周期上报。传感器数据采集用 `millis()` 调度，不阻塞主循环。

### 4.3 Web 前端：待注册设备入口（P3，推荐）

设备管理页新增"**待注册设备**"区块：调用 `GET /api/v1/admin/devices/pending` 列出未注册设备（SN/大棚/首次上报时间/最近数值），点"添加"打开注册弹窗并自动带出 `deviceSn` 与 `sensorType`，用户只需填名称、确认类型即可提交。APP 端同步入口列为二期。

### 4.4 数据库

无需新增表：缓冲放内存即可满足"注册前数据不丢"（保留最近 100 条）。若后续要求长时间离线缓冲，再考虑 `pending_device_data` 表（列为三期）。

## 5. 实施步骤与验收标准

| Phase | 内容 | 验收标准 |
|---|---|---|
| **P1** | 后端解析改造：SN 优先解析 + 未注册分支 + 缓冲 + 日志限流 | 未注册设备上报：后端日志出现 `未注册设备: greenhouseId=1 sn=TEMP-001，等待注册`，数据入缓冲不崩溃 |
| **P2** | ESP32 固件模板（Arduino） | 烧录上电 → broker 收到 publish → 后端缓冲记录 |
| **P3** | Web「待注册设备」入口 | 未注册设备出现在列表，一键带出 SN 注册 |
| **P4** | 注册补写 InfluxDB | 注册后历史曲线从"上电时刻"起完整 |
| **P5** | 回归 + 文档 | 现有注册流程（先注册后烧录）不受影响；补开发日志（只追加） |

**关键冒烟脚本（验证链路）**：

```bash
# 1. 模拟未注册设备上报（mosquitto_pub 或 Python paho）
mosquitto_pub -h localhost -u greenhouse -P greenhouse_dev \
  -t "greenhouse/1/device/TEMP-999" \
  -m '{"greenhouseId":1,"deviceSn":"TEMP-999","sensorType":"TEMPERATURE","value":25.6}'

# 2. 观察后端日志出现"未注册设备"
# 3. Web 添加设备 TEMP-999 → 再上报一帧
# 4. 设备列表状态 ONLINE、InfluxDB 有数据、WebSocket 推送、告警生效
```

## 6. 边界与安全

- **SN 伪造**：broker 账号共享，持账号者可伪造 SN 上报。先烧录后注册不放大此风险；**每设备独立 token 的强认证列为二期**（固件出厂预置 token，后端校验后再入库），本方案不实现。
- **SN 冲突**：`(greenhouseId, deviceSn)` 唯一，注册时仍走既有唯一性校验，无歧义。
- **缓冲上限**：每设备 100 条 + TTL 24h，防内存膨胀；限流防刷日志。
- **兼容性**：解析顺序保留 `payload.deviceId` 优先，老固件（携带 deviceId）不受影响，`先注册后烧录`流程完全兼容。

## 7. 待确认事项

1. 缓冲是否补写 InfluxDB（P4）：默认做；若希望"注册前数据不展示"，可不补写只记录日志。
2. Web「待注册设备」入口（P3）是否本期做：默认做；最小可行版可只做 P1+P2（后端+固件），前端后置。
3. 每设备独立 token（二期）是否需要提前预留固件字段（如 `DEVICE_TOKEN` 占位）。

## 8. 实施记录（2026-08-30，R44 已完成）

本方案已按上文设计全量实施（代码提交 22bd4ec，开发日志见 devlog/DEVLOG.md R44）：

- 固件预注册：firmwares 表 + 批量预注册 API + Web 固件管理页（管理员）；
- 用户绑定：设备创建填固件ID，校验存在/未绑定/类型一致，SN 自动生成 `GH{大棚ID}-{序号}`，删除自动解绑；
- 通信：MQTT 双订阅，新固件走 `device/{firmwareId}/data` 上报、`device/{firmwareId}/command` 收指令，未绑定固件丢弃+日志；旧格式保留兼容；
- 存量迁移：选 B 方案一次性迁移，38 台存量设备建档绑定（00000001~00000038，`tools/migrate_firmware_id.sql`）；
- 固件：`firmware/greenhouse_esp32`（WiFiManager 现场配网，只填 WiFi，大棚ID无需填写）+ 标签模板 `tools/device_label.html`；
- 回归：`tools/test_firmware_flow.py` 14/14 通过；顺带修复健康评分 NaN 500 与日汇总 Flux 分号编译错误。
- 待办：每设备独立 token 强认证（二期）；APP 端「待注册设备」/扫码绑定（二期）。

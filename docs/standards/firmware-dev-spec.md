---
id: STD-FW-001
title: 固件代码配置规范与硬件接入流程
type: standard
module: firmware
tags:
  - ESP32
  - 固件
  - MQTT
  - 硬件接入
  - 配网
status: final
created: 2026-08-30
last_modified: 2026-08-30
author: AI助手
related_docs:
  - /document/docs/tech/device-lifecycle.md
  - /document/docs/tech/realtime-comm.md
---

# 固件代码配置规范与硬件接入流程

## 1. 目的与适用范围

本文档定义智慧大棚 AIoT 系统 **ESP32 固件**的开发配置规范、通信协议要求，以及**硬件从出厂到接入系统的完整流程**。

适用对象：
- 固件开发者（按本规范编写/修改固件）
- 出厂操作员（预注册、烧录、贴标签）
- 大棚用户（配网、绑定设备）

固件模板：`firmware/greenhouse_esp32/`（Arduino，与本规范一一对应）。

---

## 2. 固件身份与配置规范（config.h）

固件所有出厂参数集中在 `config.h`，烧录时**只需修改此文件**。

### 2.1 必填字段

| 字段 | 示例 | 规则 | 来源 |
|---|---|---|---|
| `FIRMWARE_ID` | `"00000001"` | **8 位数字**，全局唯一，与系统 firmwares 表一致 | 管理员预注册后下发 |
| `DEVICE_TYPE` | `"SENSOR"` | 仅 `SENSOR` / `CONTROLLER`，与固件档案一致 | 出厂定 |
| `SENSOR_TYPE` | `"TEMPERATURE"` | 仅传感器固件必填，见 §2.2 枚举 | 出厂定 |
| `FIRMWARE_VERSION` | `"1.0.0"` | 语义化版本号，与档案 firmware_version 一致 | 出厂定 |
| `MQTT_HOST` | `"192.168.1.100"` | 服务器地址（生产=公网IP或穿透域名） | 部署方给 |
| `MQTT_PORT` | `1883` | 固定 1883 | 固定 |
| `MQTT_USER` / `MQTT_PASS` | `greenhouse` / `greenhouse_dev` | broker 认证账号（共享凭据） | 部署方给 |
| `REPORT_INTERVAL_MS` | `30000` | 上报周期，建议 30s（可配 10~300s） | 可调 |

### 2.2 传感器类型枚举（必须与后端一致，全大写）

```
TEMPERATURE（温度） HUMIDITY（湿度） LIGHT（光照） CO2（CO₂浓度）
SOIL_MOISTURE（土壤湿度） SOIL_TEMP（土壤温度） SOIL_PH（土壤pH） WIND_SPEED（风速）
```

### 2.3 引脚规范（GPIO 规划）

| 引脚 | 用途 | 说明 |
|---|---|---|
| GPIO 4 | DHT22 数据（传感器固件） | 上拉 4.7kΩ |
| GPIO 2 | 继电器控制（控制器固件） | 高电平开 |
| GPIO 0 | BOOT 键 | 长按 5 秒清除配网，重新进入配网模式 |

> 新传感器接入必须：① 在 `config.h` 声明引脚常量；② 在 `readSensorValue()` 中实现读取；③ 保持上报 JSON 字段不变。

---

## 3. 通信协议规范

### 3.1 Topic 规范（必须严格遵守，见 `MqttTopicConstants.java`）

| 方向 | Topic | 说明 |
|---|---|---|
| 上报（传感器/心跳） | `device/{firmwareId}/data` | QoS 1，不保留 |
| 下行（控制指令） | `device/{firmwareId}/command` | QoS 1，订阅 |

> ⚠️ Topic 中的 `{firmwareId}` 与 `FIRMWARE_ID` 完全一致；后端按固件ID反查设备定位大棚，**固件无需知道大棚ID**。

### 3.2 Payload 格式

**传感器上报：**

```json
{"firmwareId":"00000001","sensorType":"TEMPERATURE","value":25.6,"timestamp":1753088400000}
```

**控制器心跳：**

```json
{"firmwareId":"00000002","deviceType":"CONTROLLER","timestamp":1753088400000}
```

**命令回调（订阅 command 收到）：**

```json
{"action":"ON","timestamp":1753088400000}
```

| 字段 | 规则 |
|---|---|
| `timestamp` | Unix 毫秒；**生产固件必须接 NTP** 校准，禁止用 `millis()` 伪造 |
| `value` | 数值，建议保留 1~2 位小数 |
| 字段命名 | 一律驼峰，与后端 JSON 字段完全一致 |

### 3.3 连接与重连规范

- MQTT keepalive 60s；断线后**指数退避重连**（3s 起，上限 60s），不得疯狂重连
- 上电时序：WiFi 连接（含断线自动重连）→ MQTT 连接 → 订阅 command → 立即上报一帧 → 按 `REPORT_INTERVAL_MS` 周期上报
- 控制器固件周期发**心跳**（含 `deviceType:"CONTROLLER"`），维持在线状态
- 禁止使用 retained 消息（后端按普通消息处理）

### 3.4 配网规范

- **只配 WiFi**：配网页面仅 WiFi 账号/密码，**不填大棚ID**（后端按固件ID自动定位）
- 热点命名：`GH-{firmwareId}`（如 `GH-00000001`），密码出厂固定
- 配网门户超时：3 分钟无操作自动关闭，凭据存 NVS 断电不丢
- 重置：长按 BOOT 键 5 秒清除 NVS 配网信息并重启，重新进入配网模式

---

## 4. 固件版本与批次管理

| 项目 | 规则 |
|---|---|
| 版本号 | 语义化 `主.次.修订`，如 `1.0.0`；协议变更必须升主版本 |
| 批次号 | `B{YYYYMMDD}`，如 `B20260830`；同批同配置 |
| 出厂记录 | 每台记录：固件ID、版本、批次、烧录时间、烧录人（建议登记表） |
| 变更管理 | 固件代码变更必须同步更新本规范相关段落 + 开发日志 |

---

## 5. 出厂流程（管理员 + 烧录员）

```
① 管理员在 Web「固件管理」页批量预注册
   → 生成 8 位固件ID（00000039 起）+ 填类型/版本/批次
② 烧录员按固件ID修改 config.h（FIRMWARE_ID/类型/版本/批次）→ 编译烧录
③ 打印标签 tools/device_label.html → 贴到硬件外壳
   （标签必须含：固件ID、设备类型、批次号、二维码）
④ 出厂自检（见 §7 检查项）→ 装箱
```

---

## 6. 硬件接入系统流程（用户端，5 步）

```
① 上电 → 设备开热点 GH-{固件ID}
② 手机连热点 → 浏览器 192.168.4.1 → 填自家 WiFi 保存
③ 设备重启自动连网 → 开始上报 device/{固件ID}/data
④ 用户在 Web/APP「添加设备」：
   选大棚 → 填标签上的 8 位固件ID + 设备名称 → 选类型
   → 系统校验固件未绑定 → 自动生成设备编号 GH{大棚ID}-{序号}
⑤ 验证：设备列表状态 ONLINE、实时数据/曲线出现、告警生效
```

> 说明：设备可**先上电后绑定**，未绑定期间数据由后端丢弃并记日志，绑定后即恢复上报。

---

## 7. 验收标准与检查项

### 出厂自检（烧录员）

- [ ] `FIRMWARE_ID` 为 8 位数字，与预注册记录一致
- [ ] 设备类型/传感器类型与固件档案一致
- [ ] 标签信息与固件ID一致、二维码可扫
- [ ] 上电后热点名称 `GH-{固件ID}` 正确
- [ ] 配网后 MQTT 连接成功（broker 日志可见客户端上线）

### 接入验收（用户/测试）

- [ ] 添加设备后 SN 自动生成 `GH{大棚ID}-{序号}`，状态 ONLINE
- [ ] 实时数据与告警正常（Web/APP 可见）
- [ ] 控制器设备可下发 ON/OFF，设备端执行并反馈
- [ ] 断网 30 秒后恢复，设备自动重连并继续上报
- [ ] 长按 BOOT 5 秒可重新配网

### 常见问题排查

| 现象 | 排查 |
|---|---|
| 上报后设备仍离线 | 固件ID是否已绑定（Web 固件管理页查状态）|
| 数据不显示 | broker 是否放行该固件ID topic；后端日志是否报「固件未绑定」|
| 配网失败 | 热点是否被占用；密码是否正确；3 分钟超时后重新上电 |
| 重复设备 | 同一固件ID被绑定两台设备会被拒绝（唯一索引）|
| 命令无响应 | 检查固件是否订阅 command topic；继电器接线 |

---

## 8. 与系统其他模块的关系

- 固件档案：`firmwares` 表（预注册时创建，绑定后回填 bound_device_id）
- 设备绑定：`devices.firmware_id` 全局唯一索引
- MQTT 解析：`MqttSubscriber` 按 firmwareId 反查（新格式），未绑定丢弃+日志
- 命令下发：`ControlService` 发布 `device/{firmwareId}/command`
- 方案文档：`docs/tech/device-lifecycle.md`（TECH-009，含二期待办）

---
id: TECH-001
title: 设备感知层技术设计文档
type: tech-design
module: perception-layer
tags:
  - ESP32
  - MQTT
  - RTSP
  - 传感器
  - 设备分组
  - 硬件设计
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - /document/docs/prd/smart-greenhouse-prd.md
  - /document/docs/api/api-design.md
---

## 概述

设备感知层是智慧大棚AIoT系统的数据源头，负责环境数据采集、设备控制执行和视频流推流。每个大棚部署3-5组独立ESP32传感器节点和1个ESP32-CAM摄像头节点，通过MQTT协议上报数据，通过RTSP协议推流视频。每组传感器节点配备4路继电器控制执行器，实现环境感知与设备控制的闭环。

**核心目标**：
- 实时采集11种环境参数（温湿度、光照、CO2/O2、土壤温湿度、EC、N/P/K）
- 每30秒上报一次传感器数据，每60秒发送心跳保活
- 支持MQTT指令下发控制4路继电器执行器
- ESP32-CAM RTSP推流供FFmpeg定时截帧
- 断线自动重连，看门狗保障稳定运行

**硬件清单**（每组ESP32节点）：
| 传感器 | 型号 | 接口 | 采集参数 |
|--------|------|------|----------|
| 空气温湿度 | SHT30 | I2C (0x44) | 温度/湿度 |
| 光照强度 | BH1750 | I2C (0x23) | 光照lux |
| CO2浓度 | MQ-135 | ADC (GPIO34) | CO2 ppm |
| O2浓度 | O2传感器模块 | ADC (GPIO35) | O2 % |
| 土壤湿度 | 电容式 | ADC (GPIO32) | 含水率% |
| 土壤温度 | DS18B20 | 单总线(GPIO4) | 土壤温度 |
| 土壤EC | EC探头 | I2C (0x75) | 电导率mS/cm |
| N/P/K | RS485五合一 | UART(GPIO16/17) | N/P/K mg/kg |

## 架构设计

### 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                         大棚 (3-5组独立节点)                       │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │ ESP32 #1     │  │ ESP32 #2     │  │ ESP32 #3     │  ...      │
│  │ group_id=1   │  │ group_id=2   │  │ group_id=3   │           │
│  │ 东侧传感组   │  │ 西侧传感组   │  │ 入口传感组   │           │
│  │              │  │              │  │              │           │
│  │ 8个传感器    │  │ 8个传感器    │  │ 8个传感器    │           │
│  │ 4路继电器    │  │ 4路继电器    │  │ 4路继电器    │           │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘           │
│         │                 │                 │                    │
│  ┌──────┴─────────────────┴─────────────────┴───────┐            │
│  │              ESP32-CAM (独立节点)                  │            │
│  │              不归属任何组，RTSP推流                 │            │
│  └──────────────────────┬───────────────────────────┘            │
│                         │                                        │
└─────────────────────────┼────────────────────────────────────────┘
                          │
            WiFi局域网 ───┴─── MQTT (1883) + RTSP (8554)
                          │
                          ▼
              ┌───────────────────────┐
              │    Mosquitto Broker   │
              │    + FFmpeg截帧服务    │
              └───────────────────────┘
```

### ESP32固件架构

```
main.cpp (主循环)
├── config/          # WiFi/MQTT/GroupID配置管理
│   └── ConfigManager
├── sensors/         # 传感器驱动层（统一接口）
│   ├── SHT30Driver      # I2C 空气温湿度
│   ├── BH1750Driver     # I2C 光照
│   ├── MQ135Driver      # ADC CO2
│   ├── O2SensorDriver   # ADC O2
│   ├── DS18B20Driver    # 单总线 土壤温度
│   ├── SoilMoistureDriver # ADC 土壤湿度
│   ├── ECDriver         # I2C 土壤EC
│   └── NPKDriver        # UART N/P/K (RS485→TTL)
├── mqtt/            # MQTT通信模块
│   └── MqttClient   # PubSubClient封装
├── actuator/        # 执行器控制
│   └── RelayController # 4路继电器
├── watchdog/        # 看门狗+断线重连
└── rtsp/            # ESP32-CAM RTSP服务端
```

### 传感器驱动统一接口

```cpp
class SensorDriver {
public:
    virtual bool begin() = 0;           // 初始化传感器
    virtual bool isAvailable() = 0;     // 检测传感器是否在线
    virtual float readValue() = 0;      // 读取传感器数值
    virtual String getSensorType() = 0; // 返回传感器类型标识
    virtual String getUnit() = 0;       // 返回单位
};
```

### 引脚分配

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
| GPIO25 | 数字输出 | GPIO | 继电器第1路→通风风机 |
| GPIO26 | 数字输出 | GPIO | 继电器第2路→卷帘/遮阳网 |
| GPIO27 | 数字输出 | GPIO | 继电器第3路→滴灌阀门 |
| GPIO14 | 数字输出 | GPIO | 继电器第4路→补光灯 |

### 4路继电器分配方案

申报书要求5种执行设备，通过合并卷帘+遮阳网为1路（共用电机）实现4路控制：

| 路数 | GPIO | 控制设备 | 说明 |
|------|------|----------|------|
| 第1路 | GPIO25 | 通风风机 | 通风换气 |
| 第2路 | GPIO26 | 卷帘/遮阳网 | 共用电机，正转=展开卷帘，反转=收起遮阳网 |
| 第3路 | GPIO27 | 滴灌阀门 | 灌溉 |
| 第4路 | GPIO14 | 补光灯 | LED植物补光 |

## 数据流

### 传感器数据采集上报流

```
定时器触发(30秒)
  → 遍历传感器列表
    → 调用 SensorDriver.readValue()
    → 异常值过滤（超出合理范围的读数标记为无效）
    → 组装JSON: {"value": 25.3, "timestamp": 1690000000, "group_id": 2}
  → MQTT publish:
      Topic: greenhouse/{gh_id}/group/{group_id}/device/{dev_id}/sensor/{sensor_type}
  → 等待下个采集周期
```

### 设备控制指令流

```
后端下发控制指令
  → MQTT publish: greenhouse/{gh_id}/device/{dev_id}/control/{actuator_id}
  → ESP32 MQTT回调接收
  → 解析指令: {"action": "ON", "timestamp": 1690000000}
  → RelayController.setRelay(gpio_pin, ON/OFF)
  → 上报执行状态: greenhouse/{gh_id}/device/{dev_id}/status
      {"online": true, "uptime": 3600, "group_id": 2, "actuators": {"1": "ON", "2": "OFF"}}
```

### RTSP截帧流

```
ESP32-CAM → RTSP推流 (rtsp://esp32-cam-ip:8554/stream)
  → FFmpeg定时截帧 (每30分钟)
    → 命令: ffmpeg -rtsp_transport tcp -i rtsp://... -vframes 1 -q:v 2 output.jpg
  → 本地文件系统存储
  → 触发长势评估 (可选AI分析)
  → 更新 growth_assessments 表
```

### 心跳保活流

```
定时器触发(60秒)
  → MQTT publish: greenhouse/{gh_id}/device/{dev_id}/heartbeat
      消息: {"timestamp": 1690000000, "group_id": 2}
  → 后端接收 → 更新 devices.last_heartbeat
  → 心跳超时(>120秒未收到) → 标记设备离线
  → WebSocket推送设备离线通知
```

## 接口设计概要

### MQTT Topic定义

**数据上报（ESP32 → Broker → 后端）**：
```
greenhouse/{gh_id}/group/{group_id}/device/{dev_id}/sensor/{sensor_type}
```
- sensor_type: temp, humidity, light, co2, o2, soil_temp, soil_humidity, ec, n, p, k
- 消息格式: `{"value": 25.3, "timestamp": 1690000000, "group_id": 2}`

**设备控制（后端 → Broker → ESP32）**：
```
greenhouse/{gh_id}/device/{dev_id}/control/{actuator_id}
```
- 消息格式: `{"action": "ON", "timestamp": 1690000000}`

**设备状态上报（ESP32 → Broker → 后端）**：
```
greenhouse/{gh_id}/device/{dev_id}/status
```
- 消息格式: `{"online": true, "uptime": 3600, "group_id": 2, "actuators": {"1": "ON", "2": "OFF"}}`

**心跳（ESP32 → Broker → 后端）**：
```
greenhouse/{gh_id}/device/{dev_id}/heartbeat
```
- 消息格式: `{"timestamp": 1690000000, "group_id": 2}`

### 传感器类型枚举

```
TEMP (空气温度, °C)    | HUMIDITY (空气湿度, %)  | LIGHT (光照, lux)
CO2 (二氧化碳, ppm)    | O2 (氧气, %)           | SOIL_TEMP (土壤温度, °C)
SOIL_HUMIDITY (土壤湿度, %) | EC (电导率, mS/cm)
N (氮, mg/kg) | P (磷, mg/kg) | K (钾, mg/kg)
```

## 关键算法/逻辑

### 异常值过滤

```cpp
// 传感器读数异常值过滤
bool isValidReading(float value, float minValid, float maxValid) {
    if (isnan(value)) return false;
    if (value < minValid || value > maxValid) return false;
    // 连续3次读取异常值则判定传感器故障
    return true;
}
```

各传感器合理范围：
| 参数 | 最小值 | 最大值 |
|------|--------|--------|
| 温度 | -10°C | 60°C |
| 湿度 | 0% | 100% |
| 光照 | 0 lux | 200000 lux |
| CO2 | 300 ppm | 5000 ppm |
| O2 | 15% | 25% |
| 土壤温度 | -5°C | 50°C |
| 土壤湿度 | 0% | 100% |
| EC | 0 mS/cm | 10 mS/cm |

### 断线重连机制

```cpp
// 指数退避重连策略
int retryDelay = 1000;  // 初始1秒
int maxRetryDelay = 60000;  // 最大60秒

void reconnect() {
    while (!mqttClient.connected()) {
        if (mqttClient.connect(clientId, mqttUser, mqttPass)) {
            // 重新订阅控制Topic
            mqttClient.subscribe(controlTopic);
            retryDelay = 1000;  // 重置
        } else {
            delay(retryDelay);
            retryDelay = min(retryDelay * 2, maxRetryDelay);
        }
    }
}
```

### 传感器采集调度

```
主循环:
  loop() {
    mqttClient.loop();       // MQTT消息处理（高优先级）
    
    if (millis() - lastSensorRead >= 30000) {  // 30秒采集周期
        readAllSensors();
        publishAllData();
        lastSensorRead = millis();
    }
    
    if (millis() - lastHeartbeat >= 60000) {   // 60秒心跳周期
        sendHeartbeat();
        lastHeartbeat = millis();
    }
  }
```

## 技术选型理由

| 决策项 | 选择 | 理由 |
|--------|------|------|
| 多组独立ESP32 | 3-5组，每组独立WiFi+MQTT | 大棚30-50米长，I2C线缆超1米信号衰减严重；单点故障不影响其他组 |
| PlatformIO开发 | VS Code插件 | 库管理方便，编译速度快，团队协作友好 |
| PubSubClient | MQTT客户端库 | Arduino生态最成熟的MQTT库，轻量稳定 |
| 非阻塞采集 | millis()定时器 | 避免delay()阻塞MQTT消息接收，保证指令响应实时性 |
| 指数退避重连 | retryDelay *= 2 | 避免WiFi恢复时大量设备同时冲击Broker |
| ESP32-CAM独立部署 | 不归属任何传感器组 | 摄像头位置通常在大棚中央，与传感器组位置不同 |
| 4路继电器(5种设备) | 卷帘+遮阳网合并1路 | 共用电机驱动机构，正反转控制两种设备 |

## 注意事项

1. **I2C地址冲突**：SHT30(0x44)、BH1750(0x23)、EC探头(0x75)必须在不同地址，接线前确认地址不冲突。

2. **MQTT消息必须携带group_id**：每条传感器数据和控制消息必须包含group_id字段，这是后端实现多组数据对比和聚合查询的基础。

3. **传感器异常处理**：读取失败时不能阻塞主循环，应记录错误次数，连续失败超过阈值后通过状态消息上报传感器故障。

4. **WiFi凭证管理**：禁止硬编码WiFi密码，推荐首次配网时通过BLE或SmartConfig配置，或通过MQTT配置消息动态更新。

5. **RTSP推流稳定性**：ESP32-CAM推流建议降分辨率(VGA 640x480)、降帧率(10fps)、JPEG质量50%，避免因带宽不足导致花屏或断流。

6. **继电器安全**：控制指令下发后应有状态回报确认机制，避免指令丢失导致设备状态不一致。

7. **组ID配置**：每个ESP32固件烧录时配置不同的group_id，可通过PlatformIO编译宏或MQTT配置消息设置，确保不同节点group_id唯一。

8. **看门狗**：必须实现硬件看门狗和软件看门狗，ESP32在异常死锁时能自动重启恢复。

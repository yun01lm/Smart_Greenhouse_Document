---
id: TECH-004
title: 环境预警引擎技术设计文档
type: tech-design
module: alert-engine
tags:
  - 预警引擎
  - 阈值检测
  - LSTM预测
  - 场景联动
  - 自定义阈值
  - 多级预警
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - /document/docs/prd/alert-center.md
  - /document/docs/prd/device-control.md
  - /document/docs/api/alert/alert-api.md
  - /document/docs/api/control/control-api.md
---

## 概述

环境预警引擎是系统的核心决策模块，负责实时监测传感器数据，通过阈值检测、统计趋势分析和LSTM时序预测三种策略，对异常环境条件进行分级预警（INFO/WARNING/CRITICAL），并触发场景联动实现自动调控。支持用户自定义预警阈值，优先于系统默认值生效。

**预警策略分层**：
- **第一层（阈值检测）**：实时比较当前值与阈值，即时触发（零延迟）
- **第二层（统计趋势）**：基于滑动窗口的趋势分析，检测缓慢恶化（分钟级延迟）
- **第三层（LSTM预测）**：基于历史数据的未来趋势预测，提前30分钟预判（后期上线）

**预警级别**：
- INFO：轻度偏离，建议关注（如湿度偏低5%）
- WARNING：中度偏离，需要处理（如温度超限3°C）
- CRITICAL：严重偏离，紧急处理（如CO2浓度超限50%）

## 架构设计

### 预警引擎整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        预警引擎 (C6)                              │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    预警策略调度器                          │    │
│  │              (每30秒检查一次，与传感器采集频率对齐)          │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│         ┌─────────────────┼─────────────────┐                   │
│         ▼                 ▼                  ▼                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ 阈值检测器   │  │ 趋势分析器   │  │ LSTM预测器   │          │
│  │              │  │              │  │ (后期上线)   │          │
│  │ 当前值 vs    │  │ 滑动窗口     │  │ 30分钟预测   │          │
│  │ 自定义/默认  │  │ 统计分析     │  │ 时序模型     │          │
│  │ 阈值         │  │              │  │              │          │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │
│         │                 │                  │                   │
│         └─────────────────┼──────────────────┘                   │
│                           ▼                                      │
│                  ┌──────────────────┐                            │
│                  │   预警分级合并    │                            │
│                  │  (去重+升级逻辑)  │                            │
│                  └────────┬─────────┘                            │
│                           │                                      │
│         ┌─────────────────┼─────────────────┐                   │
│         ▼                 ▼                  ▼                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ 预警存储     │  │ WebSocket    │  │ 场景联动     │          │
│  │ MySQL alerts │  │ 实时推送     │  │ 自动控制     │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

### 预警规则引擎

```
预警规则 = 触发条件 + 预警级别 + 传感器组定位 + 联动场景

规则类型:
  THRESHOLD  → 阈值规则:  当前值 > max 或 < min → 触发
  TREND      → 趋势规则:  连续N分钟上升/下降 → 触发
  COMPOSITE  → 复合规则:  高温+低湿 同时满足 → 触发
  WEATHER    → 天气规则:  未来24小时预报有极端天气 → 提前预警
```

### 阈值优先级机制

```
预警阈值查询优先级:
  1. user_alert_thresholds (用户自定义)  → 最高优先级
  2. sensors.min_threshold / max_threshold (系统默认) → 回退

查询逻辑:
  SELECT * FROM user_alert_thresholds
  WHERE user_id = ? AND greenhouse_id = ? AND sensor_type = ? AND enabled = 1
    AND (group_id = ? OR group_id IS NULL)
  
  如果找到 → 使用用户自定义阈值
  如果未找到 → 使用 sensors 表的系统默认阈值
```

## 数据流

### 实时预警检测流

```
传感器数据到达 (MQTT消费 → InfluxDB写入)
  → 触发预警检查 (异步事件驱动)
  → 获取当前大棚的所有传感器最新值
  → 对每个传感器类型执行:
      ├── 阈值检测:
      │   ├── 查询阈值 (用户自定义优先，系统默认回退)
      │   ├── 比较: value > max_threshold → WARNING/CRITICAL
      │   │         value < min_threshold → WARNING/CRITICAL
      │   └── 记录超限幅度确定级别
      │
      ├── 趋势检测:
      │   ├── 查询最近30分钟数据 (60个点)
      │   ├── 线性回归计算趋势斜率
      │   └── 如果持续恶化 → 提前预警 (INFO级别)
      │
      └── 天气辅助:
          ├── 查询 weather_cache 最新预报
          └── 如果未来有极端天气 → 提前预警
  → 预警去重:
      └── 同一传感器组+同一参数+同一级别的未处理预警
          → 5分钟内不重复生成
  → 预警升级:
      └── 如果已存在同参数INFO预警 → 升级为WARNING
  → 存储预警记录 (MySQL alerts表)
  → WebSocket推送预警通知 (含group_id和zone_label)
  → 检查场景联动规则:
      └── 如果预警级别匹配场景触发条件 → 执行场景联动
```

### 预警通知推送流

```
预警生成
  → WebSocket推送 (STOMP → /topic/greenhouse/{id}/alerts):
      {
        "id": 56,
        "greenhouseId": 3,
        "groupId": 2,
        "zoneLabel": "西侧",
        "level": "WARNING",
        "title": "温度过高预警",
        "content": "西侧区域温度达到35.2°C，超过您设置的最高值32°C",
        "sensorType": "TEMP",
        "sensorValue": 35.2,
        "thresholdValue": 32.0,
        "createdAt": "2026-07-11T10:30:00"
      }
  → APP端:
      ├── 通知栏推送 (Firebase/FCM)
      └── APP内预警中心实时更新
```

### 场景联动触发流

```
预警触发 → 查询 scenes 表:
  WHERE trigger_condition 匹配当前预警类型+级别
    AND greenhouse_id = ?
    AND enabled = 1
  → 解析 actions_json:
      [
        {"device_id": 1, "actuator_id": 1, "action": "ON"},   // 开启通风风机
        {"device_id": 1, "actuator_id": 2, "action": "ON"},   // 展开遮阳网
        {"device_id": 1, "actuator_id": 3, "action": "ON"}    // 开启滴灌
      ]
  → 逐条执行设备控制指令:
      POST /api/v1/control/actuator
  → 记录 control_logs (source=ALERT)
  → WebSocket推送场景执行结果
```

### LSTM预测流（Phase 3上线）

```
定时任务 (每10分钟)
  → 查询过去24小时传感器数据 (按group_id分组)
  → 数据预处理: 归一化、缺失值填充、滑动窗口构建
  → LSTM模型推理 (部署为Spring Boot内嵌模型或HTTP服务):
      ├── 输入: [T-30min ... T-0min] 30分钟窗口数据
      └── 输出: [T+5min ... T+30min] 未来30分钟预测序列
  → 预测值 vs 阈值比较:
      └── 如果预测未来30分钟内将超限 → 提前发出INFO预警
  → 存储预测结果 → 与后续实际值对比评估模型准确率
```

## 接口设计概要

### 预警查询接口

| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/alerts | 预警列表 (分页/按级别筛选/按大棚/按组) |
| PUT | /api/v1/alerts/{id}/read | 标记已读 |

### 预警规则管理

| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/alerts/rules | 预警规则列表 |
| POST | /api/v1/alerts/rules | 创建预警规则 |
| PUT | /api/v1/alerts/rules/{id} | 更新规则 |
| DELETE | /api/v1/alerts/rules/{id} | 删除规则 |

### 自定义阈值管理

| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/alerts/thresholds | 自定义阈值列表 |
| POST | /api/v1/alerts/thresholds | 设置自定义阈值 |
| PUT | /api/v1/alerts/thresholds/{id} | 更新自定义阈值 |
| DELETE | /api/v1/alerts/thresholds/{id} | 删除(回退系统默认) |

### 自定义阈值请求体

```json
{
  "greenhouse_id": 1,
  "group_id": 2,
  "sensor_type": "TEMP",
  "min_threshold": 15.0,
  "max_threshold": 32.0,
  "enabled": true
}
```

### 预警规则请求体

```json
{
  "greenhouse_id": 1,
  "group_id": null,
  "sensor_type": "TEMP",
  "rule_type": "THRESHOLD",
  "condition_json": {
    "max_threshold": 35.0,
    "min_threshold": 10.0,
    "duration_minutes": 5
  },
  "alert_level": "WARNING",
  "scene_id": 1,
  "enabled": true
}
```

## 关键算法/逻辑

### 阈值检测算法

```java
public AlertResult checkThreshold(String greenhouseId, String groupId, 
                                   String sensorType, double currentValue) {
    // 1. 查询阈值 (用户自定义优先)
    Threshold threshold = getEffectiveThreshold(greenhouseId, groupId, sensorType);
    if (threshold == null) return AlertResult.NORMAL;
    
    // 2. 阈值比较
    if (threshold.getMaxThreshold() != null && currentValue > threshold.getMaxThreshold()) {
        double deviation = currentValue - threshold.getMaxThreshold();
        AlertLevel level = determineLevel(deviation, threshold.getMaxThreshold());
        return new AlertResult(level, "超过上限", currentValue, threshold.getMaxThreshold());
    }
    
    if (threshold.getMinThreshold() != null && currentValue < threshold.getMinThreshold()) {
        double deviation = threshold.getMinThreshold() - currentValue;
        AlertLevel level = determineLevel(deviation, threshold.getMinThreshold());
        return new AlertResult(level, "低于下限", currentValue, threshold.getMinThreshold());
    }
    
    return AlertResult.NORMAL;
}

private AlertLevel determineLevel(double deviation, double threshold) {
    double ratio = deviation / threshold;
    if (ratio > 0.3) return AlertLevel.CRITICAL;  // 偏差超过30%
    if (ratio > 0.1) return AlertLevel.WARNING;   // 偏差超过10%
    return AlertLevel.INFO;                       // 偏差10%以内
}
```

### 趋势分析算法

```java
public AlertResult checkTrend(String greenhouseId, String groupId, 
                               String sensorType, int windowMinutes) {
    // 1. 查询最近N分钟数据
    List<SensorDataPoint> recentData = sensorDataRepository
        .queryHistory(greenhouseId, sensorType, 
                      Instant.now().minus(windowMinutes, ChronoUnit.MINUTES), 
                      Instant.now(), groupId, null);
    
    if (recentData.size() < windowMinutes * 2) return AlertResult.NORMAL; // 数据不足
    
    // 2. 线性回归计算趋势
    double[] slope = simpleLinearRegression(recentData);
    
    // 3. 如果斜率超过阈值 → 预警
    double criticalSlope = getCriticalSlope(sensorType); // 各参数恶化速率阈值
    
    if (Math.abs(slope[0]) > criticalSlope) {
        String direction = slope[0] > 0 ? "持续上升" : "持续下降";
        return new AlertResult(AlertLevel.INFO, direction + "趋势", 
                               slope[0], null);
    }
    
    return AlertResult.NORMAL;
}
```

### 预警去重与升级逻辑

```java
public void processAlert(AlertResult result, String greenhouseId, String groupId) {
    // 1. 查重: 同一大棚+同一组+同一参数+同一级别的未处理预警
    Optional<Alert> existing = alertRepository
        .findUnresolvedByGreenhouseAndGroupAndSensorAndLevel(
            greenhouseId, groupId, result.getSensorType(), result.getLevel());
    
    if (existing.isPresent()) {
        // 5分钟内有重复预警 → 跳过
        if (existing.get().getCreatedAt()
                .isAfter(Instant.now().minus(5, ChronoUnit.MINUTES))) {
            return;
        }
    }
    
    // 2. 升级检查: 如果存在同参数低级别预警 → 升级
    if (result.getLevel() == AlertLevel.WARNING) {
        alertRepository.findUnresolvedByGreenhouseAndGroupAndSensorAndLevel(
            greenhouseId, groupId, result.getSensorType(), AlertLevel.INFO)
            .ifPresent(alert -> {
                alert.setLevel(AlertLevel.WARNING);
                alert.setTitle("预警升级: " + result.getTitle());
                alertRepository.save(alert);
            });
    }
    
    // 3. 存储新预警
    Alert alert = new Alert();
    alert.setGreenhouseId(Long.valueOf(greenhouseId));
    alert.setGroupId(groupId != null ? Long.valueOf(groupId) : null);
    alert.setLevel(result.getLevel());
    alert.setTitle(result.getTitle());
    alert.setContent(result.getContent());
    alert.setSensorValue(BigDecimal.valueOf(result.getCurrentValue()));
    alertRepository.save(alert);
    
    // 4. 推送通知
    webSocketService.pushAlert(alert);
    
    // 5. 触发场景联动
    sceneEngine.checkAndExecute(alert);
}
```

## 技术选型理由

| 决策项 | 选择 | 理由 |
|--------|------|------|
| 事件驱动检测 | Spring ApplicationEvent | 传感器数据写入后异步触发预警检查，不阻塞MQTT消费 |
| 阈值优先策略 | 用户自定义 > 系统默认 | 不同作物/生长阶段对环境要求不同，必须支持用户自主控制 |
| LSTM分阶段实施 | 先统计后LSTM | 申报书进度表9月才开始AI训练，统计方法可立即上线 |
| 预警去重 | 5分钟窗口+同级别去重 | 避免同一问题反复推送通知，减少用户打扰 |
| 场景联动 | 数据库规则驱动 | 规则可动态配置无需重启服务，灵活性高 |
| 定时任务 | Spring @Scheduled | 轻量级定时任务，无需引入Quartz等框架 |

## 注意事项

1. **阈值优先级必须正确**：预警引擎在查询阈值时，必须先查 user_alert_thresholds 表（条件：user_id + greenhouse_id + sensor_type + enabled=1），未找到时再回退到 sensors 表的系统默认值。优先级错误会导致用户自定义阈值失效。

2. **预警定位到传感器组**：预警记录必须包含 group_id 字段，推送通知必须包含 zone_label（如"东侧"），让用户能精确定位问题区域。对于全棚级别预警，group_id 可为 NULL。

3. **场景联动安全**：自动执行的场景联动涉及设备控制，必须记录完整的 control_logs（source=ALERT），便于追溯。联动前应检查设备在线状态，避免下发无效指令。

4. **LSTM数据准备**：LSTM模型需要至少1个月的历史数据才能有效训练。在Phase 1-2阶段，预留数据积累管道（按组存储，保留原始精度），为后期训练做准备。

5. **预警频率控制**：同一传感器组的同一参数，在5分钟内不重复生成相同级别的预警。避免传感器抖动导致预警风暴。

6. **天气辅助预警**：天气数据每3小时从和风天气API拉取。结合天气预报的预警应标记 weather_info 字段，便于用户理解预警来源。

7. **预警规则可扩展**：alert_rules 的 condition_json 使用JSON格式存储条件，便于后续新增复合条件（如高温+低湿同时触发）而不需修改表结构。

8. **自定义阈值权限**：只有棚主(OWNER)可以设置自定义阈值。员工(WORKER)只能查看，不能修改。如果员工的 can_view_alerts 功能被禁用，则完全看不到预警信息。

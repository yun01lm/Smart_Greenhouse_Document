---
id: TASK-F05
title: 设备控制模块开发
type: task
module: F5 APP控制
tags: [android, control, scene, actuator, switch]
status: completed
created: 2026-07-13
completed: 2026-07-13
author: AI助手
devlog_steps: [22, 23, 24, 25]
dependencies: [TASK-C01, TASK-C07, TASK-C12]
---

# TASK-F05: 设备控制模块开发

> APP端设备控制模块，支持单设备开关控制和一键场景联动。

---

## 任务概述

开发Android端设备控制功能，用户可通过底部Tab进入设备控制页面，查看大棚内所有执行器设备并远程开关，同时支持一键场景联动操作。

## 前置重构（Step 22-23, 25）

为给设备控制模块腾出底部导航位置，进行了导航重构：

| 步骤 | 内容 | 说明 |
|------|------|------|
| Step 22 | 预警合并到看板 | 看板页面新增"预警中心"入口卡片，AlertFragment 不再作为独立Tab |
| Step 23 | 诊断+问答合并为AI助手 | 新建 AiAssistantFragment（TabLayout+ViewPager2），内含诊断/问答子页 |
| Step 25 | 导航更新为4Tab | 看板 / AI助手 / 设备控制 / 我的 |

## 功能列表

| 序号 | 功能 | 说明 | 状态 |
|------|------|------|------|
| 1 | 设备列表展示 | 显示所有执行器设备（名称/类型/状态/区域） | ✅ |
| 2 | 单设备开关控制 | Switch按钮控制ON/OFF，离线设备禁用 | ✅ |
| 3 | 设备状态反馈 | 在线运行中(绿色)/停止(灰色)/离线(红色) | ✅ |
| 4 | 场景联动列表 | 横向展示预设场景卡片 | ✅ |
| 5 | 一键场景执行 | 点击执行按钮触发场景联动 | ✅ |
| 6 | 5种设备类型 | 风机/卷帘/遮阳网/阀门/补光灯 | ✅ |

## 架构设计

```
ControlFragment (UI层)
  ├── SceneAdapter (横向) → 场景卡片 + 执行按钮
  ├── DeviceAdapter → 设备卡片 + Switch开关
  └── Toast → 操作结果反馈

ControlViewModel (业务逻辑层)
  ├── loadDevices(long) → 加载设备列表
  ├── loadScenes(long) → 加载场景列表
  ├── controlActuator(ActuatorInfo, String) → 单设备控制
  └── executeScene(SceneInfo) → 场景执行

GreenhouseRepository (数据层)
  ├── getActuators(greenhouseId) → GET devices/actuators
  ├── controlActuator(id, action, gid) → POST control/actuator
  ├── getScenes(greenhouseId) → GET control/scenes
  └── executeScene(id, gid) → POST control/scenes/{id}/execute
```

## 开发规范检查

| 规范要求 | 状态 |
|----------|------|
| Activity/Fragment不含业务逻辑 | ✅ 通过 |
| ViewModel不持有Context | ✅ 通过 |
| 网络请求在后台线程 | ✅ 通过（ExecutorService） |
| 使用ViewBinding | ✅ 通过 |
| ViewModel + LiveData模式 | ✅ 通过 |
| Switch避免循环触发 | ✅ 通过（setOnCheckedChangeListener(null)） |

## 变更文件

### 新增文件 (20个)
- `data/model/ActuatorInfo.java`
- `data/model/SceneInfo.java`
- `data/model/ControlResponse.java`
- `data/model/ControlRequest.java`
- `data/model/SceneExecuteRequest.java`
- `viewmodel/ControlViewModel.java`
- `adapter/DeviceAdapter.java`
- `adapter/SceneAdapter.java`
- `ui/control/ControlFragment.java`
- `layout/fragment_control.xml`
- `layout/item_device_control.xml`
- `layout/item_scene.xml`
- `drawable/ic_control.xml`
- `drawable/ic_device_fan.xml`
- `drawable/ic_device_roller.xml`
- `drawable/ic_device_shade.xml`
- `drawable/ic_device_valve.xml`
- `drawable/ic_device_light.xml`
- `drawable/ic_device_default.xml`
- `ui/assistant/AiAssistantFragment.java`（前置重构）
- `layout/fragment_ai_assistant.xml`（前置重构）
- `drawable/ic_ai_assistant.xml`（前置重构）

### 修改文件 (6个)
- `data/api/GreenhouseApiService.java`
- `data/repository/GreenhouseRepository.java`
- `ui/common/MainActivity.java`
- `ui/dashboard/DashboardFragment.java`
- `res/layout/fragment_dashboard.xml`
- `res/menu/bottom_nav_menu.xml`
- `res/values/strings.xml`

---

## API接口

```
POST   control/actuator          — 控制单个设备 (ControlRequest)
GET    control/scenes             — 获取场景列表 (greenhouseId)
POST   control/scenes/{id}/execute — 执行场景 (SceneExecuteRequest)
GET    devices/actuators          — 获取设备列表 (greenhouseId)
```

---

## 关联任务

- **TASK-C01**（用户认证）：API鉴权依赖
- **TASK-C07**（设备控制模块）：后端控制指令下发
- **TASK-C12**（场景联动引擎）：后端场景逻辑

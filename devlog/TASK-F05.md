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

### 后续变更（F11 集成）

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `ui/control/ControlFragment.java` | 修改 | 员工无 can_control_device 时隐藏设备/场景列表，显示"无权限"提示 |
- **TASK-C12**（场景联动引擎）：后端场景逻辑

---

## 后续变更（APP-E01：按大棚分组 + 场景添加，2026-08-13）

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `data/model/DeviceGroup.java` | 新增 | 设备分组模型（大棚ID/名称/设备列表） |
| `data/model/CreateSceneRequest.java` | 新增 | 创建场景请求模型（POST /control/scenes body） |
| `adapter/DeviceAdapter.java` | 修改 | 改为双 ViewType 分组展示（分组标题+设备项），修复开关监听顺序 |
| `adapter/SceneDevicePickAdapter.java` | 新增 | 添加场景对话框设备动作选择适配器 |
| `res/layout/item_device_group_header.xml` | 新增 | 分组标题布局（大棚名+设备数） |
| `res/layout/dialog_create_scene.xml` | 新增 | 添加场景对话框布局（大棚/名称/描述/设备动作） |
| `res/layout/item_scene_device_pick.xml` | 新增 | 对话框设备勾选行布局 |
| `res/layout/fragment_control.xml` | 修改 | 场景区域新增"＋ 添加"入口 |
| `viewmodel/ControlViewModel.java` | 修改 | 新增 loadDeviceGroups / loadAllScenes / createScene |
| `data/repository/ControlRepository.java` | 修改 | 新增 getGreenhouses / createScene |
| `data/api/GreenhouseApiService.java` | 修改 | 新增 POST control/scenes |
| `ui/control/ControlFragment.java` | 修改 | 接入分组数据渲染、添加场景对话框交互 |

> 验证：离线 Gradle 编译通过（EXIT=0）；运行时验证待模拟器环境恢复后进行。

---

## 后续变更（R36：设备控制页防抖 + 设备控制记录页，2026-08-14）

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `viewmodel/ControlViewModel.java` | 修改 | 加载代际计数（过期响应丢弃）、loading 计数管理、场景执行/创建防重、单设备控制本地即时更新状态 |
| `adapter/SceneAdapter.java` | 修改 | setOperating：执行中禁用按钮并显示"执行中..." |
| `adapter/ControlLogAdapter.java` | 新增 | 控制记录列表适配器（来源标签：手动/场景触发/预警联动） |
| `ui/control/ControlFragment.java` | 修改 | 观察 isOperating；新增"设备控制记录"入口 |
| `ui/control/ControlLogActivity.java` | 新增 | 设备控制记录页（分页、按来源筛选、加载更多） |
| `data/model/ControlLogItem.java` | 新增 | 控制日志模型（含 source/sceneName/success） |
| `data/api/GreenhouseApiService.java` | 修改 | 新增 getControlLogs 分页接口 |
| `data/repository/ControlRepository.java` | 修改 | 新增 getControlLogs |
| `data/model/DeviceInfo.java` | 修改 | 新增 setStatus/setLastValue（本地即时更新用） |
| `res/layout/activity_control_log.xml` | 新增 | 控制记录页布局（含来源筛选 Chip） |
| `res/layout/item_control_log.xml` | 新增 | 记录项布局 |
| `res/layout/fragment_control.xml` | 修改 | 设备列表标题行新增"设备控制记录"入口 |
| `AndroidManifest.xml` | 修改 | 注册 ControlLogActivity |

> 验证：APP gradle 离线编译通过（EXIT=0）；运行时验证（防抖手感、控制记录展示）待重新安装 APK 后进行。

---

## 后续变更（R39：场景名映射 + 设备图标，2026-08-28）

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `SceneAdapter.java` | 修改 | 场景名编码映射（fnvcc → 水泵+风机联动，与 Web 端一致，不改数据库） |
| 设备图标 | 重建 | 旧 APK 未含 ic_device_* 图标资源，重新构建安装后正常显示 |

> 验证：场景卡片显示"水泵+风机联动"；设备行图标正常（绿色水滴/风机图标）。

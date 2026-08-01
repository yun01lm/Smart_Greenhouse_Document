---
id: TASK-FIX02
title: 第一轮功能补全（A1-A6）
type: fix
module: multi
tags: [五级地址, 员工更新, 天气预警, TTS, Web QA, 文档修正]
status: completed
created: 2026-08-01
completed: 2026-08-01
author: AI助手
---

# TASK-FIX02: 第一轮功能补全

## A3: 五级地址补全
- Greenhouse实体已有town/village，AdminOwnerController遗漏
- 修改 `AdminOwnerController.java`: 补全town/village字段

## A5: 员工更新端点
- 新建 `UpdateEmployeeRequest.java` DTO
- PermissionController 新增 PUT /{employeeId}
- PermissionService 新增 updateEmployee() 含手机号唯一性校验

## A4: 天气预报结合预警
- SmartGreenhouseApplication 添加 @EnableScheduling
- AlertRuleRepository 新增 findByRuleTypeAndEnabledTrue(WEATHER)
- AlertEngine: checkWeatherRule() + scheduledWeatherCheck() (30min定时)

## A1+A2: TTS + Web QA + 来源展示
- 新建 `web/src/api/qa.js`
- 新建 `web/src/views/qa/QaPage.vue` (聊天气泡+TTS+来源)
- 路由添加 /qa，侧边栏添加菜单

## A6: 文档修正
- sensor-api.md 追加 history/compare HTTP方法修正说明
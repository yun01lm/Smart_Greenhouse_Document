---
id: TASK-FIX05
title: 本地运行验证 + Web端修复 + 数据库修复
type: fix
module: web,backend,simulator
tags: [BOM, 路由, export-default, 中文乱码, utf8mb4, 限流, paho-mqtt]
status: completed
created: 2026-08-02
completed: 2026-08-02
author: AI助手
---

# TASK-FIX05: 本地运行验证 + Web端修复 + 数据库修复

## 背景
本地启动全栈验证（后端 8080 + Web 3000 + 模拟器），发现 Web 端白屏、
数据库中文乱码、登录被误限流等问题。

## 修复清单

### E1: Web 端 BOM 清理（4 文件）
- web/package.json / vite.config.js / src/main.js / src/api/qa.js

### E2: router/index.js 路由块错乱
- owner 和 qa 路由块搅乱，第 71 行多余左花括号 → 重新排列修复

### E3: request.js 缺失 export default
- axios 实例未导出 → 补 export default request

### E4: 模拟器环境
- 安装 paho-mqtt 2.1.0（MinGW Python 3.12）
- devices.json 移除 BOM

### E5: MySQL 中文乱码
- utf8mb4 重新导入 init_seed_data.sql
- 手动修复 users/greenhouses 中文字段
- 删除测试账号 test_user_082

### E6: RateLimitInterceptor 限流 bug
- 登录与普通 API 共用计数器导致登录误限流
- 改为分开计数（login| / api| 前缀）

## 验证
- ✅ 后端登录/大棚/设备/实时数据接口全部正常
- ✅ 中文正常显示（张棚主、一号番茄大棚）
- ✅ 8 设备在线，传感器数据实时更新
- ✅ 模拟器 → MQTT → 后端 → InfluxDB 通路验证通过

---
id: TASK-FIX06
title: Mosquitto 认证持久化 + 一键启动脚本
type: fix
module: infra,devops
tags: [mosquitto, docker, 一键启动, batch]
status: completed
created: 2026-08-03
completed: 2026-08-03
author: AI助手
---

# TASK-FIX06: Mosquitto 认证持久化 + 一键启动脚本

## 背景
电脑重启后 Mosquitto 容器因认证文件（passwd/acl）未持久化而反复重启；
同时为用户提供一键启动脚本，双击即可自动拉起全部服务并打开浏览器。

## 修复清单

### E1: Mosquitto 认证文件持久化
- mosquitto/ 新增 passwd、acl 源文件并纳入版本管理
- mosquitto.conf 认证路径指向数据卷 /mosquitto/data/
- docker-compose.yml 清理无效挂载，挂载 mosquitto_data 卷
- 认证文件复制进数据卷，容器重启后认证正常

### E2: 一键启动脚本 start_all.bat
- 环境检查：Maven / npm / Python 路径存在性
- Docker 引擎与容器检查，未运行则 docker compose up -d
- 后端 8080 / Web 3000 / 模拟器 端口或进程检测，未运行才启动
- Web 首次运行自动 npm install
- 等待就绪后自动打开浏览器 http://localhost:3000
- 修复批处理括号陷阱：if/else 块内 ASCII 括号截断代码块导致双分支执行
- 模拟器 PowerShell 进程检测移到代码块外

## 验证
- ✅ Mosquitto 容器重启后稳定运行，MQTT 连接正常
- ✅ start_all.bat 全服务运行态执行约 4 秒，退出码 0
- ✅ 各服务保持原进程，无重复启动

---
id: TASK-FIX04
title: 后端编译修复（Maven + JDK 21）
type: fix
module: backend
tags: [编译, javac崩溃, 括号修复, BOM, Maven]
status: completed
created: 2026-08-02
completed: 2026-08-02
author: AI助手
---

# TASK-FIX04: 后端编译修复

## 背景
IDEA 使用 JDK 21.0.11 编译 backend 时 javac 抛 AssertionError（VirtualParser.errPos），
疑似该 JDK 版本的编译器 bug。更换 Maven 3.9.16 + JDK 21.0.8 后编译失败，实际根因是
多个源文件存在语法错误（括号不匹配、错误引用、BOM）。

## 修复清单

### FIX-A: AlertEngine.java
- 括号结构错误：try 块闭合括号错位、checkWeatherRule/scheduledWeatherCheck 多余大括号
- 引用不存在的 WeatherService → 改为 QWeatherService
- getMinValue/getMaxValue → getMinThreshold/getMaxThreshold（与 UserAlertThreshold 实体字段一致）

### FIX-B: PermissionAspect.java
- case EXPERT -> {} 缺少闭合大括号（导致后续解析崩溃）
- 局部变量 auth 与 Authentication auth 重名 → 改名 dataAuth

### FIX-C: GlobalExceptionHandler.java
- 错误导入 org.springframework.bind.* → org.springframework.web.bind.*

### FIX-D: PermissionService.java
- 悬挂 @Transactional（无对应方法）导致 "不是可重复的批注接口"
- 删除悬挂注解，为 removeEmployee() 补 @Transactional

### FIX-E: BOM 清理
- 全量清理 backend/src/main/java 下 .java 文件的 UTF-8 BOM

## 验证
- ✅ mvn clean compile 成功（common 4 文件 + backend 206 文件）
- 📌 IDEA 配置：Maven home = F:/apache-maven-3.9.16-bin/apache-maven-3.9.16
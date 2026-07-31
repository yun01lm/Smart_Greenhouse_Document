---
id: TASK-SEC01
title: Android Token 加密存储加固
module: app
type: task
priority: critical
status: completed
created: 2026-08-01
started: 2026-08-01
completed: 2026-08-01
assignee: AI助手
related_docs: [CODE_REVIEW.md, AI开发规则文档_v2.0.md]
tags: [安全加固, Android, EncryptedSharedPreferences, Token]
---

# Android Token 加密存储加固

---

## 任务描述

### 背景
代码审查发现 Android 端 JWT Token 明文存储在 SharedPreferences 中。在 root 设备或恶意应用环境下，Token 可被轻易窃取，导致身份冒充。

### 目标
将 Token 存储从明文 SharedPreferences 替换为 EncryptedSharedPreferences（AES-256 加密，Android Keystore 硬件保护）。

### 范围
仅 Android 端 `TokenManager.java` 及依赖配置。

---

## 验收标准

1. [x] Token 写入时自动 AES-256 加密
2. [x] Token 读取时自动解密（对调用方透明）
3. [x] 现有 `GreenhouseApplication.onCreate()` 中的 `TokenManager.init(this)` 无需修改
4. [x] 编译通过

---

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `app/build.gradle` | 修改 | 新增 `androidx.security:security-crypto:1.1.0-alpha06` 依赖 |
| `app/.../data/local/TokenManager.java` | 重写 | SharedPreferences → EncryptedSharedPreferences；主密钥使用 Android Keystore (AES256-GCM) |

---

## 阻塞记录
无阻塞

---

## 备注
- 加密方案：主密钥 AES256-GCM (Android Keystore) + Key 加密 AES256-SIV + Value 加密 AES256-GCM
- `init()` 方法签名不变（仍接收 Context），调用方无需修改
- minSdk 26 完全支持 EncryptedSharedPreferences

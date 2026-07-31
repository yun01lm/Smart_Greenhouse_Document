---
id: TASK-REF01
title: Android Repository上帝对象拆分
module: app
type: task
priority: high
status: completed
created: 2026-08-01
started: 2026-08-01
completed: 2026-08-01
assignee: AI助手
related_docs: [CODE_REVIEW.md]
tags: [重构, Android, Repository, 单一职责]
---

# Android Repository 上帝对象拆分

## 任务描述

### 背景
`GreenhouseRepository.java` 单个类 900+ 行，封装了认证、传感器、预警、控制、诊断、健康、QA、长势、专家等全部 API 调用，违反单一职责原则。

### 目标
拆分为 1 个基类 + 9 个职责单一的 Repository。

### 范围
Android 端 `data/repository/` 目录 + 11 个 ViewModel。

## 验收标准

1. [x] 原 900 行文件拆分为 10 个文件，最大不超过 250 行
2. [x] 每个 Repository 只负责一个业务领域
3. [x] 公共逻辑抽取到 BaseRepository
4. [x] 所有 ViewModel 引用更新完毕
5. [x] 删除旧 GreenhouseRepository.java

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `app/.../repository/BaseRepository.java` | 新建 | 公共骨架（线程池、Handler、Callback、错误处理） |
| `app/.../repository/AuthRepository.java` | 新建 | 登录、获取当前用户 |
| `app/.../repository/SensorRepository.java` | 新建 | 实时数据、大棚列表、历史数据 |
| `app/.../repository/AlertRepository.java` | 新建 | 预警列表、已读标记、阈值管理 |
| `app/.../repository/ControlRepository.java` | 新建 | 执行器控制、场景管理 |
| `app/.../repository/DiagnosisRepository.java` | 新建 | 图像诊断、诊断历史 |
| `app/.../repository/HealthRepository.java` | 新建 | 健康评分、历史、详情 |
| `app/.../repository/QaRepository.java` | 新建 | 文字/语音问答、问答历史 |
| `app/.../repository/GrowthRepository.java` | 新建 | 长势评估、截帧、作物周期 |
| `app/.../repository/ExpertRepository.java` | 新建 | 专家、聊天、授权 |
| `app/.../repository/GreenhouseRepository.java` | 删除 | — |
| `app/.../viewmodel/*.java` (11个) | 修改 | 引用更新 |

## 阻塞记录
无阻塞

## 备注
- Callback 接口从 `GreenhouseRepository.Callback` 迁移到 `BaseRepository.Callback`
- 向后兼容：所有方法签名保持不变，仅引用路径变化

---
id: TASK-REF03
title: API限流
module: backend
type: task
priority: high
status: completed
created: 2026-08-01
started: 2026-08-01
completed: 2026-08-01
assignee: AI助手
related_docs: [CODE_REVIEW.md]
tags: [后端, 安全加固, 限流]
---

# API 限流

## 任务描述
新增基于内存滑动窗口的 API 限流，通用 API 60次/分钟/IP，登录接口 5次/分钟/IP。

## 验收标准
1. [x] 超限返回 HTTP 429 + 友好提示
2. [x] 登录接口单独限制（防暴力破解）
3. [x] 注册接口不限流
4. [x] 支持反向代理 X-Forwarded-For

## 关键修改
| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `backend/.../config/RateLimitInterceptor.java` | 新建 | 滑动窗口限流逻辑 |
| `backend/.../config/WebMvcConfig.java` | 新建 | 注册拦截器 |

## 阻塞记录
无阻塞

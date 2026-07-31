---
id: TASK-REF02
title: 后端全局异常处理器
module: backend
type: task
priority: high
status: completed
created: 2026-08-01
started: 2026-08-01
completed: 2026-08-01
assignee: AI助手
related_docs: [CODE_REVIEW.md]
tags: [后端, 异常处理, 安全加固]
---

# 后端全局异常处理器

## 任务描述
新增 @RestControllerAdvice 全局异常处理器，拦截所有未捕获异常，防止堆栈信息泄露到前端。

## 验收标准
1. [x] BusinessException 返回业务错误码和消息
2. [x] BadCredentialsException 返回 "用户名或密码错误"（不暴露原因）
3. [x] 参数校验失败返回字段级错误详情
4. [x] 未知异常兜底返回 "服务器内部错误"
5. [x] 文件大小超限返回友好提示

## 关键修改
| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `backend/.../config/GlobalExceptionHandler.java` | 新建 | 8 个异常处理方法 |

## 阻塞记录
无阻塞

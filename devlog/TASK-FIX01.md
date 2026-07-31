---
id: TASK-FIX01
title: 审计问题修复（专家权限+growth模块+AI引擎配置）
module: backend
type: task
priority: high
status: completed
created: 2026-08-01
started: 2026-08-01
completed: 2026-08-01
assignee: AI助手
related_docs: [AUDIT_REPORT.md]
tags: [修复, 专家, growth, AI引擎]
---

# 审计问题修复

## 任务描述
修复 AUDIT_REPORT.md 中发现的 3 个代码缺失问题，复核 1 个误判项。

## 验收标准
1. [x] 专家授权通过后可实际查看大棚数据
2. [x] growth 模块 3 个端点可用（latest/history/images）
3. [x] AI 引擎配置端点可用（config/status）
4. [x] AI Provider 策略模式复核：已存在

## 关键修改
| 文件 | 修改类型 | 说明 |
|------|----------|------|
| PermissionAspect.java | 修改 | EXPERT 权限校验从硬编码拒绝→查询授权记录 |
| DataAuthorizationRepository.java | 修改 | 新增授权查询方法 |
| GrowthController.java | 新建 | 3 个端点 |
| GrowthService.java | 新建 | 长势数据查询 |
| GrowthResponse.java | 新建 | DTO |
| GrowthImageResponse.java | 新建 | DTO |
| GrowthAssessmentRepository.java | 修改 | 新增查询方法 |
| AdminAiController.java | 新建 | AI 引擎配置端点 |
| AUDIT_REPORT.md | 追加 | 复审说明 |

## 阻塞记录
无阻塞

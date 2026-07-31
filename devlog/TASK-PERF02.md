---
id: TASK-PERF02
title: OkHttpClient单例化 + Web瘦身 + 连接池
module: app,web,backend
type: task
priority: medium
status: completed
created: 2026-08-01
started: 2026-08-01
completed: 2026-08-01
assignee: AI助手
related_docs: [CODE_REVIEW.md]
tags: [性能优化]
---

# OkHttpClient单例 + Web按需引入 + 连接池

## 任务描述
- Android: OkHttpClient 改为单例模式，复用连接池
- Web: Element Plus 从全量引入改为按需引入
- Backend: HikariCP 连接池显式调优

## 验收标准
1. [x] ApiClient.init() 多次调用不重复创建 OkHttpClient
2. [x] Web 首屏仅打包实际使用的组件
3. [x] 连接池最大 20 连接、最小 5 空闲

## 关键修改
| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `app/.../ApiClient.java` | 修改 | OkHttpClient 静态字段 + 判空 |
| `web/package.json` | 修改 | 新增按需引入插件 |
| `web/vite.config.js` | 修改 | AutoImport + Components 插件 |
| `web/src/main.js` | 修改 | 移除全量引入 |
| `backend/.../application-dev.yml` | 修改 | 连接池配置 |

## 阻塞记录
无阻塞

---
id: TASK-FIX03
title: 第二轮安全加固+性能优化（B1-B4 + C1-C5）
type: fix+perf
module: multi
tags: [JWT, refresh-token, CORS, 密码复杂度, 登录锁定, Async, WebSocket, RecyclerView, LRU缓存, API去重]
status: completed
created: 2026-08-01
completed: 2026-08-01
author: AI助手
---

# TASK-FIX03: 第二轮安全加固+性能优化

## B1: JWT Refresh Token
- JwtTokenProvider: generateRefreshToken(24h) + getUserIdFromExpiredToken()
- LoginResponse: +refreshToken
- AuthController: POST /auth/refresh
- Web request.js: 401自动刷新

## B2: CORS白名单
- SecurityConfig: cors()配置

## B3: 密码复杂度
- AuthService.register(): >=8位+字母+数字

## B4: 登录锁定
- User: +loginFailCount +lockedUntil
- AuthService.login(): 5次锁定30分钟

## C1: 知识库异步
- @EnableAsync + KnowledgeService @Async

## C2: WebSocket退避
- websocket.js: 指数退避1s→30s

## C3: RecyclerView优化
- 5个Fragment: setHasFixedSize(true)

## C4: Android缓存
- BaseRepository: LRU缓存50条/30s

## C5: API去重
- request.js: 300ms窗口去重
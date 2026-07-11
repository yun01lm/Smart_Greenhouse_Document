---
id: API-001
title: 认证模块 API
type: api
module: auth
tags: [认证, JWT, 注册, 登录, 用户信息]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

认证模块负责用户注册、登录和身份验证。系统使用 JWT Token 进行无状态认证，Token 通过 `Authorization: Bearer <token>` 请求头传递。APP端仅支持 OWNER（棚主）和 WORKER（员工）角色注册，ADMIN（管理员）角色由系统预设。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| POST | `/api/v1/auth/register` | 否 | 公开 | 用户注册（角色: OWNER/WORKER） |
| POST | `/api/v1/auth/login` | 否 | 公开 | 用户登录，返回JWT Token |
| GET | `/api/v1/auth/profile` | 是 | 已登录 | 获取当前用户信息（含角色+权限） |

## 请求/响应示例

### 1. 用户注册

**请求：**
```json
POST /api/v1/auth/register
Content-Type: application/json

{
  "username": "zhangfarmer",
  "password": "Farm2026!",
  "phone": "13800138001",
  "realName": "张三",
  "role": "OWNER"
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "username": "zhangfarmer",
    "phone": "13800138001",
    "role": "OWNER",
    "createdAt": "2026-07-11T10:00:00"
  }
}
```

### 2. 用户登录

**请求：**
```json
POST /api/v1/auth/login
Content-Type: application/json

{
  "username": "zhangfarmer",
  "password": "Farm2026!"
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ6aGFuZ2Zhcm1lciIsInJvbGUiOiJPV05FUiIsImlhdCI6MTc1MjIyNjAwMCwiZXhwIjoxNzUyMjMzMjAwfQ.xxx",
    "userId": 1,
    "username": "zhangfarmer",
    "role": "OWNER",
    "expiresIn": 7200
  }
}
```

### 3. 获取当前用户信息

**请求：**
```
GET /api/v1/auth/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "username": "zhangfarmer",
    "phone": "13800138001",
    "role": "OWNER",
    "realName": "张三",
    "greenhouseCount": 3,
    "permissions": {
      "canViewData": true,
      "canControlDevice": true,
      "canDiagnose": true,
      "canAskExpert": true,
      "canManageEmployee": true
    },
    "createdAt": "2026-07-11T10:00:00"
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 2003 | 409 | 用户名已存在 |
| 2004 | 409 | 手机号已注册 |
| 2005 | 401 | 用户名或密码错误 |
| 2001 | 401 | 未登录或Token已过期 |
| 2002 | 401 | Token无效 |
| 1001 | 400 | 参数错误（用户名/密码格式不符） |

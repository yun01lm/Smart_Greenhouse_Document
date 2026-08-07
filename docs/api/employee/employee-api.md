---
id: API-010
title: 员工管理 API
type: api
module: employee
tags: [员工, 权限分配, 棚主, 单归属]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - 智慧大棚AIoT系统架构方案
  - AI开发规则文档
---

## 概述

员工管理模块供棚主（OWNER）管理名下员工，包括创建员工账号、分配大棚访问权限和功能权限。员工只能归属一个棚主，不存在切换棚主的情况。权限模型按大棚（数据维度）和功能（操作维度）两个层面进行精细化控制。

**认证方式**：JWT Bearer Token | **响应格式**：`{"code":200,"message":"success","data":{...}}`

## 端点列表

### 棚主端 - 员工管理

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| POST | `/api/v1/owner/employees` | 是 | OWNER | 创建/邀请员工 |
| GET | `/api/v1/owner/employees` | 是 | OWNER | 员工列表 |
| PUT | `/api/v1/owner/employees/{id}` | 是 | OWNER | 更新员工信息 |
| DELETE | `/api/v1/owner/employees/{id}` | 是 | OWNER | 删除员工 |
| PUT | `/api/v1/owner/employees/{id}/permissions` | 是 | OWNER | 分配员工权限 |
| GET | `/api/v1/owner/employees/{id}/permissions` | 是 | OWNER | 查看员工权限 |

### 员工端 - 查看自身权限

| 方法 | 路径 | 认证 | 权限 | 说明 |
|------|------|------|------|------|
| GET | `/api/v1/worker/permissions` | 是 | WORKER | 查看自己的权限 |
| GET | `/api/v1/worker/greenhouses` | 是 | WORKER | 可访问的大棚列表 |

## 请求/响应示例

### 1. 创建员工

**请求：**
```json
POST /api/v1/owner/employees
Authorization: Bearer <token>
Content-Type: application/json

{
  "username": "worker_li",
  "password": "Worker2026!",
  "phone": "13800138002",
  "realName": "李四"
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 3,
    "username": "worker_li",
    "phone": "13800138002",
    "realName": "李四",
    "role": "WORKER",
    "ownerId": 1,
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

### 2. 获取员工列表

**请求：**
```
GET /api/v1/owner/employees?page=1&size=10
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "list": [
      {
        "id": 3,
        "username": "worker_li",
        "realName": "李四",
        "phone": "13800138002",
        "status": 1,
        "createdAt": "2026-07-11T10:30:00"
      }
    ],
    "total": 2,
    "page": 1,
    "size": 10
  }
}
```

### 3. 分配员工权限

**请求：**
```json
PUT /api/v1/owner/employees/3/permissions
Authorization: Bearer <token>
Content-Type: application/json

{
  "greenhousePermissions": [
    {
      "greenhouseId": 1,
      "canViewData": true,
      "canControlDevice": false,
      "canDiagnose": true,
      "canAskExpert": false,
      "canViewAlerts": true,
      "canViewHistory": true
    }
  ]
}
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "employeeId": 3,
    "permissions": [
      {
        "greenhouseId": 1,
        "greenhouseName": "1号番茄大棚",
        "canViewData": true,
        "canControlDevice": false,
        "canDiagnose": true,
        "canAskExpert": false,
        "canViewAlerts": true,
        "canViewHistory": true
      }
    ],
    "updatedAt": "2026-07-11T10:35:00"
  }
}
```

### 4. 员工查看自身权限

**请求：**
```
GET /api/v1/worker/permissions
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "employeeId": 3,
    "ownerId": 1,
    "ownerName": "张三",
    "greenhousePermissions": [
      {
        "greenhouseId": 1,
        "greenhouseName": "1号番茄大棚",
        "canViewData": true,
        "canControlDevice": false,
        "canDiagnose": true,
        "canAskExpert": false,
        "canViewAlerts": true,
        "canViewHistory": true
      }
    ]
  }
}
```

### 5. 员工可访问大棚列表

**请求：**
```
GET /api/v1/worker/greenhouses
Authorization: Bearer <token>
```

**响应（成功）：**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "list": [
      {
        "id": 1,
        "name": "1号番茄大棚",
        "cropType": "番茄",
        "permissions": {
          "canViewData": true,
          "canControlDevice": false,
          "canDiagnose": true,
          "canAskExpert": false,
          "canViewAlerts": true,
          "canViewHistory": true
        }
      }
    ]
  }
}
```

## 错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 3004 | 403 | 仅棚主可执行此操作 |
| 3005 | 403 | 员工数量已达上限 |
| 3002 | 403 | 无该大棚访问权限 |
| 2003 | 409 | 用户名已存在 |
| 2004 | 409 | 手机号已注册 |
| 1002 | 404 | 员工不存在 |
| 1001 | 400 | 参数错误 |

---

## 变更记录（追加，2026-08-07 · 步骤92：技术员角色 + 员工管理双模式 + 重置密码）

- `POST /api/v1/owner/employees` — 重构为双模式：创建模式（username/realName/phone/password + roleType=WORKER|TECHNICIAN + greenhouseId）；邀请模式（identifier 用户名/手机号 + greenhouseId）。权限字段可空，未传时按角色默认值：技术员全部开放；普通员工默认「看数据+控设备+看预警」
- `GET /api/v1/owner/employees` — 返回 WORKER+TECHNICIAN 两类员工，响应新增 `role` 字段（WORKER/TECHNICIAN）
- 新增 `PUT /api/v1/owner/employees/{employeeId}/password` — 棚主重置/初始化员工密码（新密码 >=8位且含字母和数字）
- 新增 `TECHNICIAN` 角色：可登录 Web+APP，默认权限全部开放但可被棚主收紧，后端按权限表强制校验
- 相关文件：`PermissionController` / `PermissionService` / `AddEmployeeRequest` / `ResetEmployeePasswordRequest` / `EmployeeResponse` / `User.Role`

---

## 变更记录（追加，2026-08-07 · 步骤94：Android 端棚主员工管理）

- Android 端实现员工管理全流程：棚主个人中心「员工管理」入口（仅棚主可见）
- 复用后端接口：`GET/POST /owner/employees`、`PUT /owner/employees/{id}/password`、`GET/PUT /owner/employees/{id}/permissions`、`DELETE /owner/employees/{id}`
- 新增文件：`EmployeeItem/AddEmployeeRequest/EmployeePermissionItem/UpdatePermissionRequest/ResetPasswordRequest`（模型）、`EmployeeRepository`（仓库）、`EmployeeViewModel`（VM）、`EmployeeAdapter`（列表）、`EmployeeManagementActivity`（页面）
- 功能：员工列表（类型标签）、新增员工（创建/邀请双模式 + 授权大棚）、权限设置（按大棚 6 项权限勾选）、重置密码、移除员工（二次确认）
- 技术员默认权限全开；普通员工默认「看数据+控设备+看预警」；权限变更由棚主在 APP/Web 端操作，后端权限表强制校验

---

## 变更记录（追加，2026-08-07 · 步骤95：员工大棚展示 + 权限全大棚分配）

- `GET /api/v1/owner/employees` — 响应新增 `greenhouseNames`（List<String>，员工被授权的大棚名称列表）
- `PUT /api/v1/owner/employees/{employeeId}/permissions` — 由「仅更新已有权限记录」改为 upsert：员工在该大棚无权限记录时，校验大棚归属棚主后按角色默认值新建权限记录（技术员默认全开；普通员工默认「看数据+控设备+看预警」）
- Web/Android 端权限设置界面：列出棚主全部大棚，与员工已有权限合并后由棚主逐大棚勾选 6 项权限；保存时全量提交（未勾选即取消对应权限）
- 相关文件：`PermissionService` / `EmployeeResponse` / Web `EmployeeManage.vue` / Android `EmployeeManagementActivity` 等
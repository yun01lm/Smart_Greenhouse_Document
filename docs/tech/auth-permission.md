---
id: TECH-003
title: 用户认证与权限技术设计文档
type: tech-design
module: auth-permission
tags:
  - JWT
  - RBAC
  - Spring Security
  - 权限管理
  - 多角色
  - 员工单归属
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - /document/docs/prd/smart-greenhouse-prd.md
  - /document/docs/api/api-design.md
---

## 概述

用户认证与权限模块是系统的安全基石，负责用户身份认证（JWT Token）、角色权限控制（RBAC）、大棚数据隔离（按大棚授权）和功能操作控制（按功能授权）。系统定义四种角色：管理员(ADMIN)、棚主(OWNER)、员工(WORKER)、专家(EXPERT)，通过"角色 + 大棚授权 + 功能授权"三维权限模型实现精细化访问控制。

**核心规则**：
- APP端只有棚主(OWNER)和员工(WORKER)两种角色，**禁止出现管理员**
- Web端有管理员(ADMIN)、棚主(OWNER)、专家(EXPERT)三种角色，**禁止出现员工**
- 员工只能归属**一个**棚主（单归属），不存在切换棚主的情况
- 权限模型 = RBAC角色 + 大棚授权(数据隔离) + 功能授权(操作粒度)

## 架构设计

### 角色与端的关系

```
┌─────────────────────────────────────────────────────────┐
│                      系统角色分布                         │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  ADMIN       │  │  OWNER       │  │  WORKER      │  │
│  │  管理员      │  │  棚主        │  │  员工        │  │
│  │              │  │              │  │              │  │
│  │  仅Web端 ✓   │  │  APP端 ✓     │  │  仅APP端 ✓   │  │
│  │  APP端 ✗     │  │  Web端 ✓     │  │  Web端 ✗     │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │  EXPERT 专家                                      │   │
│  │  仅Web端 ✓ | APP端 ✗                             │   │
│  │  被授权大棚(7天有效期) | 仅查看+对话功能            │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### 三维权限模型

```
权限校验 = RBAC角色 + 大棚授权 + 功能授权

┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  RBAC角色    │     │  大棚授权     │     │  功能授权     │
│              │     │              │     │              │
│  ADMIN       │     │ 数据隔离      │     │ 操作粒度      │
│  OWNER       │     │ 哪些大棚可见  │     │ 可查看数据    │
│  WORKER      │     │              │     │ 可控制设备    │
│  EXPERT      │     │              │     │ 可拍照诊断    │
│              │     │              │     │ 可求助专家    │
│              │     │              │     │ 可查看预警    │
│              │     │              │     │ 可查看历史    │
└──────────────┘     └──────────────┘     └──────────────┘
         │                   │                    │
         └───────────────────┼────────────────────┘
                             ▼
                  ┌─────────────────────┐
                  │   权限校验最终结果    │
                  │  允许 / 拒绝访问     │
                  └─────────────────────┘
```

### Spring Security + 自定义注解架构

```
请求到达
  │
  ▼
JwtAuthenticationFilter (OncePerRequestFilter)
  ├── 从 Authorization Header 提取 Token
  ├── 验证Token有效性 (JwtTokenProvider)
  ├── 解析用户ID + 角色
  └── 构建 Authentication 对象 → SecurityContext
  │
  ▼
方法级权限注解 (AOP拦截)
  ├── @RequireGreenhouseAccess  → 校验大棚授权
  │   └── 检查 employee_permissions 或 users.role
  │
  ├── @RequireFunction(function = "CAN_CONTROL_DEVICE") → 校验功能权限
  │   └── 检查 employee_permissions 对应功能字段
  │
  └── @PreAuthorize("hasRole('OWNER')") → 校验角色
  │
  ▼
Controller方法执行 / 拒绝返回403
```

### 自定义权限注解定义

```java
// 大棚授权校验注解
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface RequireGreenhouseAccess {
    // 从请求参数中提取greenhouse_id的表达式
    String value() default "#greenhouseId";
    // 允许的角色列表（默认OWNER和ADMIN无需额外授权检查）
    Role[] allowedRoles() default {Role.OWNER, Role.ADMIN};
}

// 功能权限校验注解
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface RequireFunction {
    EmployeeFunction value();  // 需要的功能
}

// 功能枚举
public enum EmployeeFunction {
    CAN_VIEW_DATA,
    CAN_CONTROL_DEVICE,
    CAN_DIAGNOSE,
    CAN_ASK_EXPERT,
    CAN_VIEW_ALERTS,
    CAN_VIEW_HISTORY
}
```

### 各角色权限矩阵

| 操作 | ADMIN | OWNER | WORKER | EXPERT |
|------|-------|-------|--------|--------|
| 注册账号 | ✓(任意) | ✓(APP) | ✗ | ✗ |
| 查看大棚数据 | 全部 | 自己的 | 被授权的 | 被授权(7天) |
| 创建/管理大棚 | ✓ | ✓ | ✗ | ✗ |
| 设备控制 | ✓ | ✓ | 被授权 | ✗ |
| 拍照诊断 | ✓ | ✓ | 被授权 | ✗ |
| AI问答 | ✓ | ✓ | 被授权 | ✗ |
| 求助专家 | ✓ | ✓ | 被授权 | ✗ |
| 查看预警 | ✓ | ✓ | 被授权 | ✗ |
| 查看历史 | ✓ | ✓ | 被授权 | 被授权(7天) |
| 管理员工 | ✗ | ✓(自己的) | ✗ | ✗ |
| 自定义阈值 | ✗ | ✓ | ✗(只读) | ✗ |
| 管理平台用户 | ✓ | ✗ | ✗ | ✗ |
| 知识库管理 | ✓ | ✗ | ✗ | ✗ |
| 聊天回复 | ✗ | ✗ | ✗ | ✓ |

## 数据流

### 用户注册流

```
APP端注册请求 → POST /api/v1/auth/register
  → 校验: role只能是OWNER或WORKER (禁止ADMIN)
  → 如果role=WORKER: 必须提供owner_id (邀请码)
  → 检查: owner_id是否存在且角色为OWNER
  → 检查: 手机号/用户名唯一性
  → 密码BCrypt加密
  → 创建users记录
    - role=OWNER: 独立创建，无owner_id
    - role=WORKER: owner_id指向邀请的棚主，单归属
  → 返回注册成功
```

### 用户登录与JWT生成流

```
登录请求 → POST /api/v1/auth/login
  → 验证用户名/密码 (BCrypt匹配)
  → 检查账号状态 (status=1 正常)
  → 生成JWT Token:
      claims:
        sub: userId
        role: OWNER | WORKER | ADMIN | EXPERT
        exp: now + 2小时
  → 返回: { token, expiresAt, user: { id, username, role } }
```

### 请求权限校验流

```
API请求 (携带Authorization: Bearer <token>)
  → JwtAuthenticationFilter 提取Token
  → 验证Token签名和有效期
  → 解析userId和role → 构建Authentication
  → Spring Security上下文设置
  → 到达Controller方法:
      ├── 检查 @PreAuthorize 角色注解
      ├── 检查 @RequireGreenhouseAccess 大棚授权
      │   ├── ADMIN: 直接通过
      │   ├── OWNER: 检查 greenhouses.user_id == userId
      │   ├── WORKER: 检查 employee_permissions 中该大棚+功能
      │   └── EXPERT: 检查 data_authorizations 有效授权
      └── 检查 @RequireFunction 功能授权 (仅WORKER)
          └── 检查 employee_permissions 对应功能开关
  → 通过: 执行业务逻辑
  → 拒绝: 返回 ErrorCode.GREENHOUSE_ACCESS_DENIED / FUNCTION_DENIED
```

### 员工权限分配流

```
棚主分配员工权限 → PUT /api/v1/owner/employees/{id}/permissions
  → 校验: 操作者是该员工的owner_id (棚主身份)
  → 请求体:
      {
        "permissions": [
          {
            "greenhouse_id": 1,
            "can_view_data": true,
            "can_control_device": false,
            "can_diagnose": true,
            "can_ask_expert": true,
            "can_view_alerts": true,
            "can_view_history": true
          }
        ]
      }
  → 更新 employee_permissions 表
  → 返回更新后的权限列表
```

## 接口设计概要

### 认证接口

| 方法 | 端点 | 说明 |
|------|------|------|
| POST | /api/v1/auth/register | 注册 (role仅限OWNER/WORKER) |
| POST | /api/v1/auth/login | 登录 → 返回JWT |
| GET | /api/v1/auth/profile | 当前用户信息 (含角色+权限) |
| POST | /api/v1/auth/refresh | 刷新Token |

### 员工管理接口（棚主端）

| 方法 | 端点 | 说明 |
|------|------|------|
| POST | /api/v1/owner/employees | 创建/邀请员工 |
| GET | /api/v1/owner/employees | 员工列表 |
| PUT | /api/v1/owner/employees/{id} | 更新员工信息 |
| DELETE | /api/v1/owner/employees/{id} | 删除员工 |
| PUT | /api/v1/owner/employees/{id}/permissions | 分配员工权限 |
| GET | /api/v1/owner/employees/{id}/permissions | 查看员工权限 |

### 员工端接口

| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/worker/permissions | 查看自己的权限 |
| GET | /api/v1/worker/greenhouses | 可访问的大棚列表 |

### 管理员接口

| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/admin/users | 用户列表 (含角色/地区筛选) |
| PUT | /api/v1/admin/users/{id} | 更新用户 |
| GET | /api/v1/admin/users/regions | 用户地区分布统计 |
| GET | /api/v1/admin/roles | 角色列表 |

### 注册请求DTO

```json
{
  "username": "farmer_zhang",
  "password": "********",
  "phone": "138****1234",
  "role": "OWNER",
  "real_name": "张三",
  "owner_id": null,
  "address": {
    "province": "河北省",
    "city": "石家庄市",
    "district": "藁城区",
    "town": "南营镇",
    "village": "马庄村",
    "detail": "村东头1号大棚"
  }
}
```

### JWT Token结构

```
Header:
{
  "alg": "HS256",
  "typ": "JWT"
}

Payload:
{
  "sub": "1",           // userId
  "role": "OWNER",      // 角色
  "iat": 1690000000,    // 签发时间
  "exp": 1690007200     // 过期时间 (2小时)
}

Signature:
HMAC-SHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

## 关键算法/逻辑

### 权限校验拦截器逻辑

```java
@Aspect
@Component
public class GreenhouseAccessAspect {
    
    @Around("@annotation(requireAccess)")
    public Object checkGreenhouseAccess(ProceedingJoinPoint pjp, 
                                         RequireGreenhouseAccess requireAccess) {
        // 1. 获取当前登录用户
        User currentUser = SecurityUtils.getCurrentUser();
        
        // 2. 从方法参数中提取greenhouseId
        Long greenhouseId = extractGreenhouseId(pjp, requireAccess.value());
        
        // 3. ADMIN直接通过
        if (currentUser.getRole() == Role.ADMIN) {
            return pjp.proceed();
        }
        
        // 4. OWNER检查大棚归属
        if (currentUser.getRole() == Role.OWNER) {
            Greenhouse gh = greenhouseRepository.findById(greenhouseId)
                .orElseThrow(() -> new BusinessException(ErrorCode.GREENHOUSE_NOT_FOUND));
            if (!gh.getUserId().equals(currentUser.getId())) {
                throw new BusinessException(ErrorCode.GREENHOUSE_ACCESS_DENIED);
            }
            return pjp.proceed();
        }
        
        // 5. WORKER检查员工权限
        if (currentUser.getRole() == Role.WORKER) {
            EmployeePermission perm = permissionRepository
                .findByEmployeeIdAndGreenhouseId(currentUser.getId(), greenhouseId)
                .orElseThrow(() -> new BusinessException(ErrorCode.GREENHOUSE_ACCESS_DENIED));
            return pjp.proceed();
        }
        
        // 6. EXPERT检查授权状态
        if (currentUser.getRole() == Role.EXPERT) {
            DataAuthorization auth = authorizationRepository
                .findActiveByExpertAndGreenhouse(currentUser.getId(), greenhouseId)
                .orElseThrow(() -> new BusinessException(ErrorCode.AUTHORIZATION_EXPIRED));
            return pjp.proceed();
        }
        
        throw new BusinessException(ErrorCode.ACCESS_DENIED);
    }
}
```

### 员工单归属校验

```java
// 创建员工时强制绑定棚主
public User createEmployee(Long ownerId, CreateEmployeeDTO dto) {
    // 验证ownerId对应的用户是OWNER角色
    User owner = userRepository.findById(ownerId)
        .orElseThrow(() -> new BusinessException(ErrorCode.RESOURCE_NOT_FOUND));
    if (owner.getRole() != Role.OWNER) {
        throw new BusinessException(ErrorCode.NOT_OWNER);
    }
    
    // 创建员工，owner_id锁定
    User employee = new User();
    employee.setUsername(dto.getUsername());
    employee.setPassword(passwordEncoder.encode(dto.getPassword()));
    employee.setRole(Role.WORKER);
    employee.setOwnerId(ownerId);  // 单归属，不可更改
    employee.setStatus(1);
    
    return userRepository.save(employee);
}
```

## 技术选型理由

| 决策项 | 选择 | 理由 |
|--------|------|------|
| JWT认证 | jjwt 0.12.x | 无状态Token，适合APP+Web双端；jjwt是Java生态最成熟的JWT库 |
| Spring Security | 6.x (Spring Boot 3.x内置) | 行业标准安全框架，支持方法级注解和过滤器链 |
| BCrypt加密 | Spring Security BCryptPasswordEncoder | 不可逆哈希，抗彩虹表攻击 |
| 自定义注解 | @RequireGreenhouseAccess + @RequireFunction | 声明式权限校验，代码清晰，AOP统一拦截 |
| 员工单归属 | users.owner_id单字段 | 比多对多关系表简单，满足"不可切换"的业务约束 |
| Token有效期 | 2小时 | 平衡安全性和用户体验，支持刷新Token续期 |

## 注意事项

1. **注册角色限制**：注册接口必须严格校验role字段，只允许OWNER和WORKER。禁止通过API直接注册ADMIN或EXPERT账号（由管理员在Web端创建）。

2. **员工单归属不可变**：users.owner_id字段一旦设置不可修改。如需更换棚主，必须原棚主删除员工账号，新棚主重新创建。

3. **APP端无管理员**：Android APP的登录和注册界面不应出现管理员选项。后端API也需校验：如果请求来自APP端注册ADMIN角色，直接拒绝。

4. **Token安全存储**：APP端Token存储在SharedPreferences（加密），Web端存储在HttpOnly Cookie或localStorage。Token过期后前端自动跳转登录页。

5. **密码强度要求**：注册时密码至少8位，包含字母和数字。密码传输使用HTTPS加密。

6. **JWT密钥管理**：JWT签名密钥通过环境变量 JWT_SECRET 注入，至少256位随机字符串。禁止硬编码，禁止提交到Git。

7. **权限缓存**：用户权限信息可在登录时一次性加载并缓存在内存中（通过SecurityContext），避免每次请求都查询数据库。权限变更时清除缓存。

8. **越权防护**：所有API必须经过权限校验，不能依赖前端隐藏按钮来实现安全。每个Controller方法必须添加对应的权限注解。

---
id: STD-001
title: 开发规范
type: standards
module: standards
tags: [coding, git, documentation]
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
---

# 开发规范

> 本文档定义智慧大棚AIoT系统的编码规范、Git分支策略、文档编写规则和代码审查标准。
> 所有开发人员必须严格遵守本文档。

---

## 编码规范

### Java编码风格

| 项目 | 规范 | 示例 |
|------|------|------|
| 类名 | PascalCase | `GreenhouseController` |
| 方法名/变量名 | camelCase | `getGreenhouseById` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 包名 | 全小写 | `com.greenhouse.module.auth` |
| 缩进 | 4个空格，禁止Tab | - |
| 行宽 | 不超过120字符 | - |
| Lombok | 必须使用Lombok | `@Data`, `@Slf4j`, `@Builder` |

**依赖注入**：推荐构造器注入，禁止 `@Autowired` 字段注入。

**分层约束**：

| 层 | 允许 | 禁止 |
|----|------|------|
| Controller | 参数校验、调用Service、返回DTO | 编写业务逻辑、直接返回Entity |
| Service | 业务逻辑、事务管理 | 直接操作HttpServletRequest/Response |
| Repository | 数据访问、自定义查询 | 拼接SQL字符串 |

**异常处理**：

- 全局异常处理器 `@RestControllerAdvice` 统一拦截
- 业务异常统一抛出 `BusinessException(ErrorCode)`
- 禁止吞掉异常（catch后必须记录日志或重新抛出）
- 外部API调用（百度/讯飞/DeepSeek）必须有超时设置和重试机制
- 禁止向客户端暴露堆栈信息

**日志规范**：

| 级别 | 使用场景 |
|------|----------|
| ERROR | 系统异常、外部API调用失败 |
| WARN | 业务异常、权限拒绝、数据异常 |
| INFO | 关键业务流程（登录、设备上线、预警触发、AI调用） |
| DEBUG | 开发调试信息（生产环境关闭） |

- 日志格式：`[时间] [级别] [线程] [类名] - 消息`
- 框架：SLF4J + Logback
- 禁止使用 `System.out.println`

**安全要求**：

- 密码必须使用 BCrypt 加密存储，禁止明文
- 必须防范SQL注入：使用JPA参数化查询，禁止拼接SQL字符串
- API Key等敏感数据通过环境变量注入，禁止硬编码
- JWT Token有效期建议2小时，支持刷新
- 文件上传必须校验类型和大小

**注释规范**：

- 所有公开API方法必须有JavaDoc注释
- 复杂业务逻辑必须有行内注释
- AI抽象层接口必须有接口说明注释
- 禁止无意义注释（如 `// 获取用户` 在 `getUser()` 上面）

---

### Vue 3 编码风格

| 项目 | 规范 |
|------|------|
| 组件名 | PascalCase（如 `GreenhouseDashboard.vue`） |
| 组合式API | 优先使用 `<script setup>` |
| 状态管理 | Pinia |
| CSS | scoped样式，使用Element Plus变量体系 |
| 路由命名 | kebab-case（如 `/greenhouse-list`） |
| API封装 | 统一在 `api/` 目录，按模块拆分文件 |
| 代码格式化 | ESLint + Prettier |

**组件规范**：

- 单文件组件不超过500行
- 复杂页面拆分为多个子组件
- Props必须定义类型和默认值
- 禁止在模板中编写复杂逻辑（使用computed）

---

### Android Java 编码风格

| 项目 | 规范 |
|------|------|
| 架构 | MVVM + Jetpack |
| 类名 | PascalCase |
| 方法名/变量名 | camelCase |
| 常量 | UPPER_SNAKE_CASE |
| 缩进 | 4个空格，禁止Tab |
| 网络请求 | Retrofit + OkHttp |
| 本地存储 | Room |
| 异步 | Kotlin Coroutines / RxJava |
| 图片加载 | Glide |
| WebSocket | STOMP over OkHttp |

**Android特有规范**：

- Activity/Fragment不写业务逻辑（放到ViewModel）
- 所有网络请求必须在子线程执行
- 禁止在主线程进行IO操作
- 权限申请使用AndroidX Activity Result API
- target SDK 34，最低支持API 26 (Android 8.0)

---

### ESP32 C/C++ 编码风格

| 项目 | 规范 |
|------|------|
| 开发框架 | Arduino-ESP32 2.x |
| 开发环境 | PlatformIO (VS Code插件) |
| 类名 | PascalCase |
| 函数名/变量名 | camelCase |
| 常量/宏 | UPPER_SNAKE_CASE |
| 缩进 | 4个空格 |

**传感器驱动接口**（每个传感器必须实现）：

```cpp
class SensorDriver {
public:
    virtual bool begin() = 0;          // 初始化传感器
    virtual bool isAvailable() = 0;    // 检测传感器是否在线
    virtual float readValue() = 0;     // 读取传感器数值
    virtual String getSensorType() = 0;// 返回传感器类型标识
};
```

**嵌入式特有规范**：

- 禁止硬编码WiFi密码和MQTT地址（必须通过配置或动态获取）
- 禁止使用阻塞式延时（必须使用非阻塞方式）
- 必须实现看门狗机制，断线自动重连（指数退避重试）
- MQTT消息必须携带 `group_id` 字段
- 传感器读取失败时必须处理异常值

---

## Git分支与提交规范

### 分支策略

| 分支 | 用途 | 说明 |
|------|------|------|
| `main` | 稳定发布分支 | 只通过PR合并，禁止直接push |
| `develop` | 日常开发分支 | 所有feature/fix分支合并到此 |
| `feature/{模块名}` | 功能开发分支 | 从develop切出，完成后合并回develop |
| `fix/{问题描述}` | Bug修复分支 | 从develop切出，修复后合并回develop |
| `release/{版本号}` | 发布准备分支 | 从develop切出，测试通过后合并到main |

**分支命名示例**：

```
feature/user-auth          # 用户认证模块
feature/mqtt-consumer      # MQTT消费模块
feature/expert-chat        # 专家聊天模块
fix/login-token-expire     # 修复登录Token过期问题
release/v1.0.0             # v1.0.0发布
```

### Conventional Commits

格式：`<type>(<scope>): <subject>`

**type类型**：

| Type | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(device): 新增ESP32设备注册接口` |
| `fix` | Bug修复 | `fix(mqtt): 修复MQTT断线重连后重复订阅问题` |
| `docs` | 文档更新 | `docs(api): 更新API端点文档` |
| `style` | 代码格式 | `style: 统一缩进为4空格` |
| `refactor` | 重构 | `refactor(alert): 提取预警规则校验逻辑` |
| `test` | 测试 | `test(greenhouse): 补充大棚Service单元测试` |
| `chore` | 构建/工具/依赖 | `chore(deps): 升级Spring Boot到3.3.2` |
| `perf` | 性能优化 | `perf(sensor): 优化时序数据批量写入` |

**scope范围**：

| Scope | 对应模块 |
|-------|----------|
| `auth` | C1 用户认证 |
| `device` | C2 设备管理 / C19 设备分组 |
| `greenhouse` | C3 大棚管理 |
| `mqtt` | C4 MQTT消费 |
| `sensor` | C5 时序数据 |
| `alert` | C6 预警 / C21 自定义阈值 |
| `control` | C7 设备控制 / C12 场景联动 |
| `diagnosis` | C8 图片诊断 |
| `qa` | C9 RAG问答 |
| `speech` | C10 语音识别 |
| `websocket` | C11 WebSocket / C16 聊天 |
| `weather` | C14 天气 |
| `fusion` | C15 多模态融合 |
| `chat` | C16 实时聊天 |
| `authorization` | C17 专家授权 |
| `permission` | C18 权限 |
| `crop` | C22 生长周期 |
| `app` | Android APP端 |
| `web` | Vue Web管理端 |
| `esp32` | ESP32固件 |
| `docker` | Docker/部署相关 |

### 提交示例

```bash
git commit -m "feat(device): 新增ESP32设备注册接口

支持设备编码唯一校验和自动绑定大棚
- 新增DeviceController.register()接口
- 新增DeviceService.registerDevice()业务逻辑
- 新增DeviceCodeExistsException异常

Closes #12"

git commit -m "fix(mqtt): 修复MQTT断线重连后重复订阅问题"
```

### Git禁止事项

- **禁止**直接push到 `main` 分支
- **禁止**force push到共享分支（`main`、`develop`）
- **禁止**提交包含API Key/密码的代码
- **禁止**提交未通过编译的代码
- **禁止**提交超过500行的大提交（应拆分为多个有意义的提交）
- **禁止**提交 `TODO` 注释而不创建对应的Issue跟踪

---

## 文档编写规范

### YAML Front Matter 格式

所有项目文档必须在文件开头使用YAML头部：

```yaml
---
id: DOC-001                    # 文档唯一标识
title: 文档标题                  # 文档标题
type: standards                 # 文档类型
module: module-name             # 所属模块
tags: [tag1, tag2]             # 标签
status: draft                   # 状态：draft / review / final / active
created: 2026-07-11             # 创建日期
last_modified: 2026-07-11       # 最后修改日期
author: 作者名                   # 作者
---
```

### 文件命名规则

- 所有文档文件名使用**小写英文+连字符**（kebab-case）
- 禁止使用中文文件名、空格、大写字母
- 示例：`development-standards.md`、`test-case-template.md`、`api-design.md`

### 文档类型及对应目录

| 文档类型 | 目录 | 命名格式 | 示例 |
|----------|------|----------|------|
| PRD（产品需求文档） | `document/docs/prd/` | `prd-{模块名}.md` | `prd-device-management.md` |
| 架构设计 | `document/docs/architecture/` | `arch-{主题}.md` | `arch-system-overview.md` |
| API文档 | `document/docs/api/` | `api-{模块名}.md` | `api-greenhouse.md` |
| 开发规范 | `document/docs/standards/` | `{规范名}.md` | `development-standards.md` |
| 测试文档 | `document/docs/test/` | `test-{模块名}.md` | `test-auth-module.md` |
| 部署文档 | `document/docs/deployment/` | `deploy-guide.md` | `deploy-guide.md` |
| 开发日志 | `document/devlog/` | `TASK-{编号}.md` | `TASK-C01.md` |

### 文档模板

#### PRD文档模板

```markdown
---
id: PRD-{编号}
title: {功能名称}产品需求文档
type: prd
module: {模块名}
tags: [prd, {模块标签}]
status: draft
created: YYYY-MM-DD
last_modified: YYYY-MM-DD
author: 作者名
---

# {功能名称}

## 1. 功能概述

## 2. 用户故事

## 3. 功能详细说明

## 4. 界面原型

## 5. 验收标准

## 6. 非功能需求
```

#### API文档模板

```markdown
---
id: API-{编号}
title: {模块名} API文档
type: api
module: {模块名}
tags: [api, rest]
status: draft
created: YYYY-MM-DD
last_modified: YYYY-MM-DD
author: 作者名
---

# {模块名} API

## 接口清单

| 方法 | 端点 | 说明 |
|------|------|------|

## 接口详情

### {接口名称}

**请求**：
**响应**：
**错误码**：
```

#### 技术方案文档模板

```markdown
---
id: TECH-{编号}
title: {主题}技术方案
type: tech-design
module: {模块名}
tags: [tech, design]
status: draft
created: YYYY-MM-DD
last_modified: YYYY-MM-DD
author: 作者名
---

# {主题}

## 1. 背景与目标

## 2. 技术选型

## 3. 详细设计

## 4. 接口定义

## 5. 数据模型

## 6. 风险与对策
```

---

## 代码审查清单

每次Pull Request合并前，审查者必须逐项检查以下15项：

1. **[ ] 编译通过**：代码可以成功编译，无编译错误
2. **[ ] 测试通过**：所有现有单元测试和集成测试通过，新增功能有对应测试
3. **[ ] 代码风格**：符合Java/Vue/Android/ESP32各自编码规范（缩进、命名、行宽）
4. **[ ] 分层正确**：Controller无业务逻辑，Service无直接数据库操作，无Entity暴露到前端
5. **[ ] 异常处理**：异常被正确捕获和处理，使用BusinessException+ErrorCode，无吞异常
6. **[ ] 日志规范**：关键操作有INFO日志，异常有ERROR日志，日志信息清晰
7. **[ ] 安全校验**：API有JWT认证校验，有权限校验（大棚授权+功能授权），密码使用BCrypt
8. **[ ] 参数校验**：所有用户输入有校验（@Valid/@NotNull/长度/范围），文件上传有类型和大小校验
9. **[ ] SQL安全**：无SQL拼接，使用JPA参数化查询，无N+1查询问题
10. **[ ] API规范**：URL使用kebab-case复数形式，带/api/v1/前缀，响应使用统一ApiResponse格式
11. **[ ] 无硬编码**：配置值通过application.yml或环境变量注入，无硬编码IP/端口/密钥
12. **[ ] 数据库规范**：新增/修改表结构有对应迁移脚本，外键使用逻辑删除（禁止CASCADE）
13. **[ ] 外部API调用**：有超时设置（连接5s/读取10s），有重试机制（最多3次，指数退避），有降级处理
14. **[ ] 文档更新**：新增/修改API时同步更新API文档，复杂逻辑有注释
15. **[ ] 无敏感信息**：无API Key、密码等提交到代码仓库，.env文件已在.gitignore中

---

## 数据库操作规范

### 通用规范

- 表名使用**小写字母+下划线**，复数形式（如 `users`、`greenhouses`、`devices`）
- 字段名使用**小写字母+下划线**（如 `user_id`、`created_at`）
- 主键统一命名 `id`（BIGINT, AUTO_INCREMENT）
- 外键命名格式：`{关联表单数}_id`（如 `greenhouse_id`）
- 时间字段：`created_at`、`updated_at`（DATETIME类型）
- 状态字段：`status`（TINYINT类型，0=禁用/离线/关，1=正常/在线/开）
- 禁止使用MySQL保留字

### 外键约束

- 所有外键必须在建表时明确定义
- **禁止级联删除（CASCADE）**，必须使用逻辑删除或手动处理关联数据
- 删除关联数据前必须检查依赖关系

### 迁移脚本

- **禁止**直接删除表或字段，必须通过迁移脚本（Flyway/Liquibase）
- **禁止**在代码中直接执行DDL语句
- **禁止**修改已有字段类型或名称（只能通过迁移脚本新增字段并废弃旧字段）
- **禁止**跳过外键约束直接插入数据
- **禁止**在生产环境手动修改数据库数据

### 查询优化

- **禁止**在循环中执行数据库查询（N+1问题）
- 复杂查询使用JPQL或Native Query
- 大数据量查询必须分页
- 合理使用索引，关注慢查询

### 多数据库规范

| 数据库 | 用途 | 关键规范 |
|--------|------|----------|
| MySQL 8.0 | 业务数据（25张表） | 所有关系型数据，JPA实体映射 |
| InfluxDB 2.7 | 传感器时序数据 | measurement: `sensor_data`，tags含greenhouse_id/device_id/group_id/sensor_type |
| Chroma | 知识库向量存储 | collection: `agricultural_knowledge`，每片500字，1024维bge-m3向量 |

---

## 附录：快速参考卡

### 命名速查

| 场景 | 规范 | 示例 |
|------|------|------|
| Java类 | PascalCase | `GreenhouseController` |
| Java方法/变量 | camelCase | `findByGreenhouseId` |
| Java常量 | UPPER_SNAKE_CASE | `DEFAULT_PAGE_SIZE` |
| Java包 | 全小写 | `com.greenhouse.module.auth` |
| Vue组件 | PascalCase | `GreenhouseDashboard.vue` |
| Vue路由 | kebab-case | `/greenhouse-list` |
| 数据库表 | snake_case, 复数 | `greenhouses` |
| 数据库字段 | snake_case | `user_id`, `created_at` |
| API路径 | kebab-case, 复数, /api/v1/ | `/api/v1/greenhouses` |
| 文档文件 | kebab-case | `development-standards.md` |
| Git分支 | kebab-case | `feature/user-auth` |

### 禁止事项速查

| 类别 | 禁止事项 |
|------|----------|
| 安全 | 明文密码、硬编码API Key、跳过JWT、SQL拼接、暴露堆栈 |
| 编码 | Controller写业务逻辑、直接返回Entity、System.out.println、N+1查询 |
| 数据库 | 级联删除、直接DDL、修改字段类型、手动改生产数据 |
| Git | push到main、force push、提交密钥、提交编译失败代码 |
| 架构 | 引入Python后端、微服务、绕过AI抽象层、替换技术栈组件 |

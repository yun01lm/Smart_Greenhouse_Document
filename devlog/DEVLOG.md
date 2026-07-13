# 智慧大棚AIoT系统 — 开发总日志

> **规则**：只追加，不修改，不删除。每条记录永久保留。

---

## 2026-07-12

### 步骤1：创建项目骨架 | ✅ 完成

- **时间**：02:55
- **操作**：
  - 创建 Maven 多模块项目结构（父项目 smart-greenhouse + common + backend）
  - 配置 Spring Boot 3.3.5 + Java 21
  - 创建 common 模块：ApiResponse（统一响应）、PageResult（分页）、BusinessException（业务异常）、ErrorCode（8大类40+错误码枚举）
  - 创建 backend 模块：启动类 SmartGreenhouseApplication、application.yml
  - 配置所有 Maven 依赖（Spring Boot、Spring AI 1.0.9、Spring Security、JPA、WebSocket、MySQL、InfluxDB、MQTT、JWT、OkHttp、Lombok）
  - 创建 .gitignore
  - Git 初始化并首次 commit（10个文件，499行）
- **结果**：项目骨架就绪，可以开始写业务代码
- **文件清单**：
  - `pom.xml`（父POM）
  - `common/pom.xml`
  - `common/src/main/java/com/greenhouse/common/ApiResponse.java`
  - `common/src/main/java/com/greenhouse/common/PageResult.java`
  - `common/src/main/java/com/greenhouse/common/BusinessException.java`
  - `common/src/main/java/com/greenhouse/common/ErrorCode.java`
  - `backend/pom.xml`
  - `backend/src/main/java/com/greenhouse/SmartGreenhouseApplication.java`
  - `backend/src/main/resources/application.yml`
  - `.gitignore`

### 步骤2：Docker 开发环境 | ✅ 完成

- **时间**：03:10
- **操作**：
  - 创建 `docker-compose.yml`：4个服务（MySQL 8.0 / InfluxDB 2.7 / Mosquitto 2.x / Chroma），含健康检查和数据卷持久化
  - 创建 `mosquitto.conf`：开发阶段允许匿名连接，含 WebSocket MQTT（端口9001）
  - 创建 `.env.example`：环境变量模板，含所有第三方API占位符
  - 创建 `application-dev.yml`：Spring Boot 完整开发环境配置（MySQL/InfluxDB/Chroma/MQTT/JWT/所有第三方API）
- **结果**：一条 `docker-compose up -d` 命令即可启动全部基础设施
- **用户确认**：MQTT 匿名连接后期改认证只需改配置+重启，不影响代码
- **文件清单**：
  - `docker-compose.yml`
  - `mosquitto.conf`
  - `.env.example`
  - `backend/src/main/resources/application-dev.yml`

### 步骤3：Spring Boot 骨架（安全+JWT+异常处理） | ✅ 完成

- **时间**：03:20
- **操作**：
  - `SecurityConfig.java`：无状态JWT认证，关闭CSRF/Session，白名单路径（注册/登录/WebSocket/健康检查/Swagger/OPTIONS），其余全部需认证
  - `JwtTokenProvider.java`：JWT 生成（含userId/username/role）、解析、验证，密钥从配置读取
  - `JwtAuthenticationFilter.java`：从 Authorization: Bearer <token> 提取Token，验证后设置SecurityContext
  - `UserDetailsServiceImpl.java`：临时内存用户实现，预置admin账号，步骤4将改为数据库查询
  - `GlobalExceptionHandler.java`：统一拦截 BusinessException / @Valid校验失败 / BadCredentialsException / AccessDeniedException / 未知异常，全部返回ApiResponse格式
  - `WebSocketConfig.java`：STOMP基础配置，端点 /ws/connect，后续步骤9完善
  - `@RequireGreenhouseAccess` / `@RequireFunction`：自定义权限注解（空壳），步骤7实现AOP切面
- **结果**：Spring Boot 安全骨架就绪，JWT认证链路完整
- **文件清单**：
  - `backend/.../config/SecurityConfig.java`
  - `backend/.../config/WebSocketConfig.java`
  - `backend/.../security/JwtTokenProvider.java`
  - `backend/.../security/JwtAuthenticationFilter.java`
  - `backend/.../security/UserDetailsServiceImpl.java`
  - `backend/.../exception/GlobalExceptionHandler.java`
  - `backend/.../security/annotations/RequireGreenhouseAccess.java`
  - `backend/.../security/annotations/RequireFunction.java`

### 步骤4：C1 用户认证模块 | ✅ 完成

- **时间**：03:30
- **操作**：
  - `User.java`：JPA实体，四种角色(ADMIN/OWNER/WORKER/EXPERT)，BCrypt密码，员工单归属(ownerId)，专家在线状态(expertStatus)
  - `UserRepository.java`：JPA Repository，支持用户名/手机号唯一性校验，按角色/棚主/在线状态查询
  - `AuthController.java`：3个API端点 — POST /api/v1/auth/register（注册）、POST /api/v1/auth/login（登录）、GET /api/v1/auth/profile（获取个人信息）
  - `AuthService.java`：注册（校验角色/用户名/手机号唯一性、员工必须指定棚主、BCrypt加密、注册成功直接返回Token）、登录（Spring Security认证+生成JWT+检查账号状态）
  - 4个DTO：RegisterRequest（含@Valid校验）、LoginRequest、LoginResponse（含Token+用户信息）、UserProfileResponse
  - `UserDetailsServiceImpl.java`：**已从临时内存版本改为数据库版本**，通过UserRepository加载用户
- **结果**：用户认证链路完整（注册→BCrypt加密→数据库存储→登录→JWT生成→过滤器验证→SecurityContext）
- **文件清单**：
  - `backend/.../entity/User.java`
  - `backend/.../repository/UserRepository.java`
  - `backend/.../module/auth/controller/AuthController.java`
  - `backend/.../module/auth/service/AuthService.java`
  - `backend/.../module/auth/dto/RegisterRequest.java`
  - `backend/.../module/auth/dto/LoginRequest.java`
  - `backend/.../module/auth/dto/LoginResponse.java`
  - `backend/.../module/auth/dto/UserProfileResponse.java`
  - `backend/.../security/UserDetailsServiceImpl.java`（修改）

### 步骤5：C3 大棚管理 + C20 地区管理 | ✅ 完成

- **时间**：03:45
- **操作**：
  - `Greenhouse.java`：JPA实体，字段包含名称、位置、作物类型、棚主ID(ownerId)、五级地址（省/市/区/镇/村）、创建时间
  - `GreenhouseRepository.java`：JPA Repository，支持按棚主查询、按棚主计数、地区统计查询（按省/市/区/镇/村分组统计大棚数量）
  - `GreenhouseController.java`：6个API端点 — GET /api/v1/greenhouses（列表，按角色过滤）、GET /api/v1/greenhouses/{id}（详情）、POST /api/v1/greenhouses（创建）、PUT /api/v1/greenhouses/{id}（更新）、DELETE /api/v1/greenhouses/{id}（删除）、GET /api/v1/greenhouses/regions（地区统计）
  - `GreenhouseService.java`：核心业务 — 按角色过滤大棚列表（ADMIN看全部、OWNER看自己的、WORKER看所属OWNER的）、棚主大棚数量限制校验、所有权校验、五级地区统计
  - 3个DTO：GreenhouseRequest（@Valid校验，名称/位置必填）、GreenhouseResponse（含作物类型和地区信息）、RegionStatsResponse（地区统计数据结构）
- **结果**：大棚管理完整闭环，支持五级地区钻取统计，权限分级过滤
- **文件清单**：
  - `backend/.../entity/Greenhouse.java`
  - `backend/.../repository/GreenhouseRepository.java`
  - `backend/.../module/greenhouse/controller/GreenhouseController.java`
  - `backend/.../module/greenhouse/service/GreenhouseService.java`
  - `backend/.../module/greenhouse/dto/GreenhouseRequest.java`
  - `backend/.../module/greenhouse/dto/GreenhouseResponse.java`
  - `backend/.../module/greenhouse/dto/RegionStatsResponse.java`

### 步骤6：C2 设备管理 + C19 设备分组 | ✅ 完成

- **时间**：04:10
- **操作**：
  - `Device.java`：JPA实体，字段包含名称、设备编号(deviceSn)、设备类型(SENSOR/CONTROLLER)、传感器子类型(8种枚举)、状态(ONLINE/OFFLINE/ALARM)、大棚归属、MQTT topic自动生成、安装位置、上次数据时间/值
  - `DeviceGroup.java`：JPA实体，使用 @ElementCollection 存储组内设备ID列表（device_group_members关联表），支持设备多分组归属
  - `DeviceRepository.java`：支持按大棚/设备类型/状态筛选查询、设备统计（总数/按类型分组统计/按状态统计）、批量查询
  - `DeviceGroupRepository.java`：按大棚查询分组、名称唯一性校验
  - `DeviceController.java`：5个API端点 — GET（列表，可选type/status参数筛选）、GET/{id}（详情）、POST（创建）、PUT/{id}（更新）、DELETE/{id}（删除）
  - `DeviceGroupController.java`：7个API端点 — GET（列表）、GET/{id}（详情）、POST（创建）、PUT/{id}（更新）、POST/{groupId}/devices/{deviceId}（添加设备）、DELETE/{groupId}/devices/{deviceId}（移除设备）、DELETE/{groupId}（删除分组）
  - `DeviceService.java`：核心业务 — 设备创建（大棚归属校验、数量上限50、编号/名称唯一性、传感器类型必填校验）、列表查询（按角色过滤+按类型/状态筛选）、更新（deviceSn和deviceType创建后不可修改）、删除
  - `DeviceGroupService.java`：核心业务 — 分组创建（上限20、名称唯一性、设备合法性校验防跨大棚）、更新、设备添加/移除、删除
  - 4个DTO：DeviceRequest（@Valid校验）、DeviceResponse（含MQTT topic和状态信息）、DeviceGroupRequest（@Size(max=50)）、DeviceGroupResponse（含deviceCount计算字段）
- **结果**：设备管理和分组功能完整，API路径遵循 RESTful 风格（`/api/v1/greenhouses/{greenhouseId}/devices` 和 `/api/v1/greenhouses/{greenhouseId}/device-groups`），设备归属严格校验，防跨大棚操作
- **文件清单**：
  - `backend/.../entity/Device.java`
  - `backend/.../entity/DeviceGroup.java`
  - `backend/.../repository/DeviceRepository.java`
  - `backend/.../repository/DeviceGroupRepository.java`
  - `backend/.../module/device/controller/DeviceController.java`
  - `backend/.../module/device/controller/DeviceGroupController.java`
  - `backend/.../module/device/service/DeviceService.java`
  - `backend/.../module/device/service/DeviceGroupService.java`
  - `backend/.../module/device/dto/DeviceRequest.java`
  - `backend/.../module/device/dto/DeviceResponse.java`
  - `backend/.../module/device/dto/DeviceGroupRequest.java`
  - `backend/.../module/device/dto/DeviceGroupResponse.java`

### 步骤7：C18 多角色权限模块 + AOP切面 | ✅ 完成

- **时间**：04:40
- **操作**：
  - `EmployeePermission.java`：JPA实体，对应 DB 第8号表，6个功能权限布尔字段（canViewData/canControlDevice/canDiagnose/canAskExpert/canViewAlerts/canViewHistory）
  - `EmployeePermissionRepository.java`：支持按员工ID/棚主ID/大棚ID查询权限，按员工+大棚组合删除
  - `PermissionController.java`：棚主端 5个API端点 — POST /api/v1/owner/employees（添加/邀请员工）、GET（员工列表）、GET/{id}/permissions（查看权限）、PUT/{id}/permissions（更新权限）、DELETE/{id}（删除员工）
  - `WorkerPermissionController.java`：员工端 2个API端点 — GET /api/v1/worker/permissions（查看自己权限）、GET /api/v1/worker/greenhouses（可访问大棚列表）
  - `PermissionService.java`：核心业务 — 添加员工（用户名/手机号查找、角色校验、单归属校验、首次绑定ownerId、权限记录创建）、权限更新（6个字段按非空更新）、删除员工（删除权限+解除归属）、员工端查询
  - `PermissionAspect.java`：AOP切面实现 — `@RequireGreenhouseAccess` 校验大棚访问权限（ADMIN放行/OWNER检查归属/WORKER查employee_permissions/EXPERT暂拒绝）、`@RequireFunction` 校验功能权限（ADMIN+OWNER放行/WORKER按6个功能标识逐一检查）、greenhouseId自动提取（支持@PathVariable/@RequestParam）
  - `UserRepository.java`：新增 `findByRoleAndOwnerId` 和 `countByRoleAndOwnerId` 方法
  - `GreenhouseService.java`：WORKER角色从返回空列表改为从 employee_permissions 查询被授权的大棚
  - `DeviceService.java`：WORKER角色从拒绝改为检查 employee_permissions 表
  - `pom.xml`：显式添加 `spring-boot-starter-aop` 依赖确保切面正常工作
- **结果**：多角色权限体系完整闭环 — 棚主可邀请员工并精细分配6项功能权限，员工端可查看自己被授权的大棚和权限，AOP切面自动拦截校验，之前步骤中 WORKER 返回空列表/拒绝的问题全部修复
- **文件清单**：
  - `backend/.../entity/EmployeePermission.java`
  - `backend/.../repository/EmployeePermissionRepository.java`
  - `backend/.../module/permission/controller/PermissionController.java`
  - `backend/.../module/permission/controller/WorkerPermissionController.java`
  - `backend/.../module/permission/service/PermissionService.java`
  - `backend/.../module/permission/dto/AddEmployeeRequest.java`
  - `backend/.../module/permission/dto/EmployeeResponse.java`
  - `backend/.../module/permission/dto/UpdatePermissionRequest.java`
  - `backend/.../module/permission/dto/PermissionResponse.java`
  - `backend/.../security/aop/PermissionAspect.java`
  - `backend/.../repository/UserRepository.java`（修改）
  - `backend/.../module/greenhouse/service/GreenhouseService.java`（修改）
  - `backend/.../module/device/service/DeviceService.java`（修改）
  - `backend/pom.xml`（修改）

### 步骤8：C4 MQTT消费者 + C5 时序数据 | ✅ 完成

- **时间**：05:10
- **操作**：
  - `MqttConfig.java`：MQTT 客户端配置，支持用户名/密码认证（可选）、自动重连、cleanSession，从 application.yml 读取 broker/clientId
  - `InfluxDbConfig.java`：InfluxDB 2.x 客户端配置，创建 InfluxDBClient/WriteApiBlocking/QueryApi 三个 Bean，从 application.yml 读取 url/token/org/bucket
  - `MqttSubscriber.java`：MQTT 消息订阅器 — @PostConstruct 启动时订阅 `greenhouse/+/device/+` 通配符主题，接收 ESP32 JSON 数据（greenhouseId/deviceId/sensorType/value/timestamp），调用 SensorDataService 写入 InfluxDB + 更新 Device 状态为 ONLINE
  - `SensorDataService.java`：时序数据核心服务 — writeData（写入 InfluxDB point）、updateDeviceStatus（更新设备在线状态和最后读数）、getRealtimeData（Flux 查询最近5分钟 last()）、getHistoryData（时间范围 + aggregateWindow 均值聚合）、getCompareData（多设备同时段对比）、getAggregateData（mean/max/min/last/count 五维统计）、exportCsv（生成 CSV 字符串）
  - `InfluxDbConfigHelper.java`：辅助组件，@Value 注入 influxdb.org/bucket，避免与 InfluxDbConfig 循环依赖
  - `SensorController.java`：5个API端点 — GET /api/v1/sensors/realtime（实时数据）、POST /api/v1/sensors/history（历史数据+聚合）、POST /api/v1/sensors/compare（多组对比）、GET /api/v1/sensors/aggregate（聚合统计）、GET /api/v1/sensors/export（CSV导出）
  - 6个DTO：SensorDataPoint（通用数据点）、SensorRealtimeResponse（按类型分组）、SensorHistoryRequest（@Valid校验）、SensorCompareResponse（含DeviceSeries子类）、SensorAggregateResponse（五维统计）、CompareRequest
- **结果**：感知→存储→查询整条链路打通 — ESP32 通过 MQTT 上报数据 → InfluxDB 存储 → API 查询（实时/历史/对比/统计/导出），设备在线状态自动更新
- **文件清单**：
  - `backend/.../config/MqttConfig.java`
  - `backend/.../config/InfluxDbConfig.java`
  - `backend/.../module/mqtt/MqttSubscriber.java`
  - `backend/.../module/sensor/service/SensorDataService.java`
  - `backend/.../module/sensor/service/InfluxDbConfigHelper.java`
  - `backend/.../module/sensor/controller/SensorController.java`
  - `backend/.../module/sensor/dto/SensorDataPoint.java`
  - `backend/.../module/sensor/dto/SensorRealtimeResponse.java`
  - `backend/.../module/sensor/dto/SensorHistoryRequest.java`
  - `backend/.../module/sensor/dto/SensorCompareResponse.java`
  - `backend/.../module/sensor/dto/SensorAggregateResponse.java`
  - `backend/.../module/sensor/dto/CompareRequest.java`

### 步骤9：C11 WebSocket 实时推送 | ✅ 完成

- **时间**：05:35
- **操作**：
  - `StompAuthInterceptor.java`：STOMP CONNECT 阶段拦截器 — 从 Header 提取 `Authorization: Bearer <token>`，调用 JwtTokenProvider 校验，解析 userId/username/role 存入 StompPrincipal，校验失败抛出异常拒绝连接
  - `WebSocketConfig.java`：重写 — 注入 StompAuthInterceptor，注册到 `configureClientInboundChannel`；连接端点 `/ws/connect` 允许跨域；消息代理 `/topic` + `/queue`；应用前缀 `/app`；用户前缀 `/user`
  - `RealtimePushService.java`：统一推送服务 — `pushSensorData()`（传感器数据 → `/topic/greenhouse/{id}/realtime`）、`pushDeviceStatus()`（设备状态变更 → `/topic/device/{id}/status`）、`pushAlert()`（预警 → `/topic/greenhouse/{id}/alerts`），全部通过 SimpMessagingTemplate.convertAndSend()
  - `MqttSubscriber.java`：修改 — MQTT 收到数据后，在写入 InfluxDB + 更新设备状态之后，调用 `pushService.pushSensorData()` 实时推送到前端
  - `SensorDataService.java`：修改 — `updateDeviceStatus()` 改为返回设备名称（String），供推送消息使用
  - 3个消息 DTO：RealtimeMessage（SENSOR_DATA）、DeviceStatusMessage（DEVICE_STATUS）、AlertPushMessage（ALERT，为步骤10准备）
- **结果**：WebSocket 实时推送链路完整 — 前端 STOMP 连接 `/ws/connect` → JWT 认证 → 订阅主题 → MQTT 数据到达后自动推送到前端，无需轮询
- **文件清单**：
  - `backend/.../module/websocket/handler/StompAuthInterceptor.java`
  - `backend/.../module/websocket/service/RealtimePushService.java`
  - `backend/.../module/websocket/dto/RealtimeMessage.java`
  - `backend/.../module/websocket/dto/DeviceStatusMessage.java`
  - `backend/.../module/websocket/dto/AlertPushMessage.java`
  - `backend/.../config/WebSocketConfig.java`（修改）
  - `backend/.../module/mqtt/MqttSubscriber.java`（修改）
  - `backend/.../module/sensor/service/SensorDataService.java`（修改）

### 步骤10：C7 设备控制 + C12 场景联动 | ✅ 完成

- **时间**：06:05
- **操作**：
  - `Scene.java`：JPA实体，对应 DB 第11号表 — 场景名称、描述、触发条件JSON（Phase 2用）、动作列表JSON、所属大棚、启用状态
  - `ControlLog.java`：JPA实体，对应 DB 第14号表 — 操作人ID、设备ID、动作(ON/OFF)、来源(MANUAL/SCENE/ALERT)、场景ID、成功/失败原因
  - `SceneRepository.java`：按大棚查询/按启用状态查询/名称唯一性校验/数量统计
  - `ControlLogRepository.java`：按设备/用户/场景查询日志，支持分页
  - `ControlService.java`：设备控制核心 — 校验设备类型(CONTROLLER)/在线状态/权限（OWNER/WORKER需canControlDevice）、MQTT下发指令到 `greenhouse/{ghId}/device/{deviceSn}/command`、自动记录控制日志、更新设备状态
  - `SceneService.java`：场景联动核心 — CRUD + 手动执行（逐一调用ControlService控制每个设备，标记source=SCENE）、设备归属校验（禁止跨大棚）、控制器类型校验
  - `ControlController.java`：2个端点 — POST /api/v1/control/actuator（控制设备）、GET /api/v1/control/logs（查询日志）
  - `SceneController.java`：5个端点 — GET /api/v1/control/scenes（列表）、POST（创建）、PUT/{id}（更新）、DELETE/{id}（删除）、POST/{id}/execute（执行）
  - 4个DTO：ControlRequest（@Valid校验）、ControlLogResponse（含username/deviceName）、SceneRequest（含SceneAction子类，@NotEmpty校验）、SceneResponse（含SceneActionInfo子类）
- **结果**：设备控制链路完整 — 用户API → MQTT下发 → ESP32执行 → 日志记录。场景联动支持一键批量控制多个设备。Phase 1 全部 10 步业务代码完成！
- **文件清单**：
  - `backend/.../entity/Scene.java`
  - `backend/.../entity/ControlLog.java`
  - `backend/.../repository/SceneRepository.java`
  - `backend/.../repository/ControlLogRepository.java`
  - `backend/.../module/control/service/ControlService.java`
  - `backend/.../module/control/service/SceneService.java`
  - `backend/.../module/control/controller/ControlController.java`
  - `backend/.../module/control/controller/SceneController.java`
  - `backend/.../module/control/dto/ControlRequest.java`
  - `backend/.../module/control/dto/ControlLogResponse.java`
  - `backend/.../module/control/dto/SceneRequest.java`
  - `backend/.../module/control/dto/SceneResponse.java`

### 步骤11：集成验证 + 推送 GitHub | ✅ 完成

- **时间**：06:30
- **操作**：
  - 整体文件清单审查：73 个 Java 文件，包结构符合 AI 开发规则文档第 3.1 节规范
  - API 路径一致性检查：所有 35+ 端点均使用 `/api/v1/` 前缀 + kebab-case + RESTful 风格，与 API 端点清单一致
  - ErrorCode 完整性检查：所有引用的错误码均在 ErrorCode 枚举中定义，无遗漏
  - Git log 审查：18 次提交，每步 feat + docs 配对，历史清晰可追溯
  - 推送到 GitHub：`yun01lm/Smart-Greenhouse`，main 分支，18 次提交全部推送成功
- **结果**：Phase 1（后端骨架 + 核心业务模块）全部完成并推送 GitHub
- **Phase 1 总结**：
  - Java 文件：73 个（含 4 个 common 模块）
  - API 端点：35+
  - 实体类：7 个（User/Greenhouse/Device/DeviceGroup/EmployeePermission/Scene/ControlLog）
  - 配置类：4 个（Security/WebSocket/MQTT/InfluxDB）
  - 安全模块：JWT + AOP 切面 + STOMP 认证
  - 数据链路：MQTT → InfluxDB 存储 + WebSocket 推送
  - 控制链路：API → MQTT 下发 → ESP32 执行 → 日志记录
- **仓库地址**：https://github.com/yun01lm/Smart-Greenhouse

### 步骤12：C6 预警引擎 + C21 自定义阈值 | ✅ 完成

- **时间**：07:30
- **操作**：
  - `AlertRule.java`：JPA实体，对应 DB 第9号表 — 4种规则类型(THRESHOLD/TREND/COMPOSITE/WEATHER)、3级告警(INFO/WARNING/CRITICAL)、条件JSON、关联场景
  - `Alert.java`：JPA实体，对应 DB 第10号表 — 告警级别/标题/内容/传感器数值/已读状态
  - `UserAlertThreshold.java`：JPA实体，对应 DB 第25号表 — 用户自定义min/max阈值、传感器类型
  - 3个Repository：AlertRuleRepository（按大棚+传感器类型查询启用规则）、AlertRepository（分页+未读统计）、UserAlertThresholdRepository（按用户+大棚+传感器查询）
  - `AlertEngine.java`：预警引擎核心 — MQTT 数据到达后调用 `check()`，比对系统规则（THRESHOLD 类型解析 JSON 中的 min/max）和用户自定义阈值，超限则生成 Alert 记录并通过 `pushService.pushAlert()` 推送 WebSocket
  - `AlertRuleService.java`：规则管理 — 创建/列表/更新/删除，权限校验（OWNER 只能操作自己大棚的规则），数量上限 50 条
  - `AlertThresholdService.java`：自定义阈值管理 — 设置（创建或更新同传感器类型的阈值）/列表/删除
  - `AlertController.java`：9个端点 — GET /api/v1/alerts（告警列表分页，可选 level 筛选）、PUT /api/v1/alerts/{id}/read（标记已读）、GET/POST/PUT/DELETE /api/v1/alerts/rules（规则 CRUD）、GET/POST/PUT/DELETE /api/v1/alerts/thresholds（阈值 CRUD）
  - 5个DTO：AlertRuleRequest（@Valid校验）、AlertRuleResponse（fromEntity转换）、AlertResponse（含greenhouseName）、ThresholdRequest、ThresholdResponse
  - `MqttSubscriber.java`：修改 — 在 WebSocket 推送之后新增 `alertEngine.check()` 调用
- **结果**：预警检测闭环完成 — MQTT 数据 → AlertEngine 比对规则/阈值 → 超限生成 Alert → WebSocket 实时推送告警。系统从"被动查看数据"升级为"主动告警通知"
- **文件清单**：
  - `backend/.../entity/AlertRule.java`
  - `backend/.../entity/Alert.java`
  - `backend/.../entity/UserAlertThreshold.java`
  - `backend/.../repository/AlertRuleRepository.java`
  - `backend/.../repository/AlertRepository.java`
  - `backend/.../repository/UserAlertThresholdRepository.java`
  - `backend/.../module/alert/service/AlertEngine.java`
  - `backend/.../module/alert/service/AlertRuleService.java`
  - `backend/.../module/alert/service/AlertThresholdService.java`
  - `backend/.../module/alert/controller/AlertController.java`
  - `backend/.../module/alert/dto/AlertRuleRequest.java`
  - `backend/.../module/alert/dto/AlertRuleResponse.java`
  - `backend/.../module/alert/dto/AlertResponse.java`
  - `backend/.../module/alert/dto/ThresholdRequest.java`
  - `backend/.../module/alert/dto/ThresholdResponse.java`
  - `backend/.../module/mqtt/MqttSubscriber.java`（修改）

### 补充：更新任务日志索引 | ✅ 完成

- **时间**：07:45
- **操作**：
  - 更新 `/workspace/Smart_Greenhouse/document/devlog/INDEX.md`：将全部 14 个已完成任务状态从 `planned` 更新为 `✅ completed`，添加完成时间和 DEVLOG 关联
  - 新增"已完成任务详情"表格：列出每个任务的完成日期、Java 文件数、API 端点数和 DEVLOG 步骤
  - 更新 Phase 1/2/3 状态标识：Phase 1 ✅、Phase 2 ✅、Phase 3 🔄
  - 更新统计概览：已完成 14、已计划 32
- **结果**：任务日志索引与实际开发进度同步

### 步骤13：AI 抽象层 + C8 病虫害图片诊断 | ✅ 完成

- **时间**：08:30
- **操作**：
  - `DiseaseRecognitionProvider.java`：AI 策略接口 — 定义 `recognize(byte[])` 方法和 `RecognitionResult` record（diseaseName/confidence/treatment/engineName/needExpertConsultation），Spring Boot 自动注入唯一实现类
  - `BaiduRecognitionProvider.java`：百度 AI 植物识别实现 — 通过 API Key + Secret Key 获取 OAuth Access Token（缓存+过期前60秒自动刷新），图片 Base64 编码后调用植物识别 API（baike_num=1），解析返回最高置信度结果和百科描述
  - `ResNetRecognitionProvider.java`：ResNet 本地模型占位实现，Phase 3 阶段将替换为真正的 PyTorch 模型推理
  - `DiagnosticRecord.java`：JPA实体，对应 DB 第12号表 — 用户ID/大棚ID/图片路径/病害名称/置信度/防治方案/识别引擎/是否已咨询专家
  - `DiagnosticRecordRepository.java`：按用户ID分页查询诊断历史，按时间倒序
  - `FileService.java`：文件上传服务 — 校验图片类型（jpg/jpeg/png/gif/webp）、大小限制（10MB）、保存到 `uploads/diagnosis/yyyy/MM/dd/uuid.ext` 日期分目录
  - `DiagnosisService.java`：诊断核心流程 — ①接收图片→保存文件 ②调用 AI 识别 ③保存诊断记录 ④返回结果。AI 识别失败时保存失败记录并抛出 BusinessException
  - `DiagnosisController.java`：2个API端点 — POST /api/v1/diagnosis/recognize（上传图片+可选greenhouseId，返回诊断结果含needExpert标识）、GET /api/v1/diagnosis/records（分页查询诊断历史）
  - `DiagnosisResponse.java`：响应 DTO — 含 `needExpert` 字段（confidence < 0.70 为 true），fromEntity 方法自动计算
  - `application-dev.yml`：新增 `spring.servlet.multipart` 配置（max-file-size: 10MB）
- **结果**：AI 病虫害诊断链路完整 — 用户拍照上传 → 百度 AI 植物识别 → 诊断记录持久化 → 低置信度引导专家咨询。策略模式设计支持未来切换 ResNet 本地模型（改配置即可）
- **文件清单**：
  - `backend/.../ai/DiseaseRecognitionProvider.java`
  - `backend/.../ai/baidu/BaiduRecognitionProvider.java`
  - `backend/.../ai/resnet/ResNetRecognitionProvider.java`
  - `backend/.../entity/DiagnosticRecord.java`
  - `backend/.../repository/DiagnosticRecordRepository.java`
  - `backend/.../module/file/service/FileService.java`
  - `backend/.../module/diagnosis/dto/DiagnosisResponse.java`
  - `backend/.../module/diagnosis/service/DiagnosisService.java`
  - `backend/.../module/diagnosis/controller/DiagnosisController.java`
  - `backend/.../resources/application-dev.yml`（修改）

### 步骤14：C9 RAG 问答 + C10 语音识别 | ✅ 完成

- **时间**：09:45
- **操作**：
  - `SpeechRecognitionProvider.java`：语音识别策略接口 — `recognize(byte[])` 方法和 `SpeechRecognitionResult` record（text/rawDialectText/confidence/dialect/engineName/durationMs），通过 `ai.voice.provider` 配置切换实现
  - `XunfeiSpeechProvider.java`：讯飞 ASR 实现 — HMAC-SHA256 签名鉴权（host+date+request-line 签名）、支持河北话方言（accent=hebei）、Base64 音频上传、拼接所有识别片段并计算平均置信度
  - `WhisperSpeechProvider.java`：本地 Whisper 占位 — Phase 3 替换为 DJL + ONNX 模型推理
  - `QaRecord.java`：JPA实体，对应 DB 第13号表 — 用户ID/问题/回答/输入类型(TEXT/VOICE)/ASR引擎/引用来源JSON/创建时间
  - `QaRecordRepository.java`：按用户ID分页查询历史（时间倒序）
  - `EmbeddingService.java`：SiliconFlow bge-m3 向量化服务 — 调用 `/v1/embeddings` API，返回 1024 维向量，支持单条和批量向量化
  - `ChromaRetrievalService.java`：Chroma 向量检索服务 — 调用 `/api/v1/collections/{name}/query` REST API，返回 Top-K 结果（含文档内容、元数据、距离→相似度转换），Chroma 不可用时返回空列表不阻塞
  - `RagQaService.java`：RAG 核心服务 — ① 向量化问题 ② Chroma top-5 检索 ③ 组装 system prompt（知识库上下文） ④ DeepSeek chat/completions 生成 ⑤ 保存 qa_records。提供 `generateAnswerOnly()` 供 VoiceQaService 复用（不重复保存记录）。知识库无内容时自动降级为通用知识并标注
  - `VoiceQaService.java`：语音问答服务 — 保存音频 → ASR 识别 → RAG 生成 → 保存 VOICE 类型记录。ASR 失败抛 AI_SPEECH_FAILED，LLM 失败仍保存识别结果
  - `QaController.java`：3个API端点 — POST /api/v1/qa/ask（文字问答）、POST /api/v1/qa/ask/voice（语音问答，multipart/form-data）、GET /api/v1/qa/records（问答历史分页）
  - 3个DTO：QaAskRequest（@Valid校验）、QaResponse（含 SourceInfo 子类和 fromEntity/fromVoiceEntity）、QaHistoryItem（问题截取前50字）
  - `FileService.java`：新增 `saveAudioFile()` 方法 — 校验音频类型（wav/mp3/amr/webm）、30MB上限、按日期分目录
  - `PageResult.java`：新增 `of(Page<T>)` 工厂方法，自动处理 Spring Data 分页偏移
  - `application-dev.yml`：新增 chroma.collection 配置、deepseek 独立配置（api-key/base-url/model）
- **结果**：AI 问答能力完整 — 文字问答走 RAG 全链路（向量化→检索→LLM→保存），语音问答先经讯飞 ASR 转写再走 RAG。降级策略完备：Chroma 不可用时用 DeepSeek 通用知识、LLM 不可用时保存错误信息。策略模式支持后期切换 Whisper 本地模型
- **文件清单**：
  - `backend/.../ai/SpeechRecognitionProvider.java`
  - `backend/.../ai/xunfei/XunfeiSpeechProvider.java`
  - `backend/.../ai/whisper/WhisperSpeechProvider.java`
  - `backend/.../entity/QaRecord.java`
  - `backend/.../repository/QaRecordRepository.java`
  - `backend/.../module/qa/service/EmbeddingService.java`
  - `backend/.../module/qa/service/ChromaRetrievalService.java`
  - `backend/.../module/qa/service/RagQaService.java`
  - `backend/.../module/qa/service/VoiceQaService.java`
  - `backend/.../module/qa/controller/QaController.java`
  - `backend/.../module/qa/dto/QaAskRequest.java`
  - `backend/.../module/qa/dto/QaResponse.java`
  - `backend/.../module/qa/dto/QaHistoryItem.java`
  - `backend/.../module/file/service/FileService.java`（修改）
  - `common/.../common/PageResult.java`（修改）
  - `backend/.../resources/application-dev.yml`（修改）

### 步骤15：C14 天气API + C22 作物生长周期 | ✅ 完成

- **时间**：10:30
- **操作**：
  - `WeatherCache.java`：JPA实体，对应 DB 第19号表 — 位置标识/温度/湿度/天气代码/风速/预报JSON/更新时间，3小时缓存策略
  - `WeatherCacheRepository.java`：按位置查询最新缓存记录
  - `QWeatherService.java`：和风天气 API 服务 — 缓存优先模式（3小时内直接返回缓存，过期调用API刷新）、getCurrentWeather()（当前天气：温度/体感温度/湿度/风速/风向/气压/能见度）、getForecast()（3天/7天预报：逐日最高最低温/天气/湿度/风速/降水量）、缓存保存到 weather_cache 表、API失败时抛出 INTERNAL_ERROR
  - `WeatherController.java`：2个端点 — GET /api/v1/weather/current（当前天气）、GET /api/v1/weather/forecast（天气预报，days参数控制3天/7天）
  - 2个DTO：WeatherCurrentResponse（含体感温度/风向/气压/能见度）、WeatherForecastResponse（含 DayForecast 子类）
  - `CropCycle.java`：JPA实体，对应 DB 第20号表 — 大棚ID/作物名称/品种/种植日期/预计收获日期/实际收获日期/当前阶段(5阶段)/阶段来源(AUTO/MANUAL)/状态(ACTIVE/COMPLETED/CANCELLED)。内置 estimateStage() 静态方法（育苗期0-20天→生长期21-40天→开花期41-55天→结果期56-80天→收获期81天+）、autoUpdateStage() 自动更新阶段、getDaysSincePlanting() 计算种植天数
  - `CropCycleRepository.java`：按大棚+状态分页查询、按大棚全量查询、检查进行中周期（防重复创建）
  - `CropCycleService.java`：核心业务 — create（校验无重复进行中周期→创建→自动估算阶段）、list（自动更新阶段后返回）、getById、update（仅ACTIVE状态可更新）、complete（设置实际收获日期+标记COMPLETED）、setStage（手动覆盖阶段→MANUAL模式）、getTimeline（生成种植→阶段变更→收获事件时间线）
  - `CropCycleController.java`：7个端点 — GET /api/v1/crop-cycles（列表，可选status筛选）、POST（创建）、GET /{id}（详情）、PUT /{id}（更新）、PATCH /{id}/stage（手动设阶段）、PATCH /{id}/complete（标记完成）、GET /{id}/timeline（时间线）
  - 3个DTO：CropCycleRequest（@Valid校验）、CropCycleResponse（含 daysSincePlanting 计算字段）、CropTimelineResponse（含 TimelineEvent 列表）
- **结果**：天气模块提供当前天气+预报查询，缓存策略减少API调用。作物生长周期模块支持完整的种植→生长→收获生命周期管理，阶段自动估算+手动修正双模式，时间线可视化追溯
- **文件清单**：
  - `backend/.../entity/WeatherCache.java`
  - `backend/.../entity/CropCycle.java`
  - `backend/.../repository/WeatherCacheRepository.java`
  - `backend/.../repository/CropCycleRepository.java`
  - `backend/.../module/weather/service/QWeatherService.java`
  - `backend/.../module/weather/controller/WeatherController.java`
  - `backend/.../module/weather/dto/WeatherCurrentResponse.java`
  - `backend/.../module/weather/dto/WeatherForecastResponse.java`
  - `backend/.../module/crop/service/CropCycleService.java`
  - `backend/.../module/crop/controller/CropCycleController.java`
  - `backend/.../module/crop/dto/CropCycleRequest.java`
  - `backend/.../module/crop/dto/CropCycleResponse.java`
  - `backend/.../module/crop/dto/CropTimelineResponse.java`

### 步骤16：C16 实时聊天 + C17 专家授权 | ✅ 完成

- **时间**：11:45
- **操作**：
  - `ChatConversation.java`：JPA实体，对应 DB 第21号表 — 用户ID/专家ID/大棚ID/咨询主题/状态(WAITING/ACTIVE/CLOSED)/关联诊断ID，专家首次回复自动从WAITING变为ACTIVE
  - `ChatMessage.java`：JPA实体，对应 DB 第22号表 — 发送者ID/发送者身份(USER/EXPERT)/消息类型(TEXT/IMAGE/VIDEO/ENV_SNAPSHOT)/文字内容/文件路径/快照JSON/已读状态
  - `ChatConversationRepository.java`：按用户/专家/状态多维度查询
  - `ChatMessageRepository.java`：按对话分页查询（时间正序）、未读统计（按用户/专家/对话三维度）、markAsRead批量标记已读
  - `DataAuthorization.java`：JPA实体，对应 DB 第23号表 — 5种状态(PENDING/APPROVED/REJECTED/EXPIRED/REVOKED)、7天有效期（approved_at+7天）、撤销人/撤销时间
  - `ExpertAvailability.java`：JPA实体，对应 DB 第24号表 — 在线状态(0/1)、最后活跃时间、最大并发数
  - `DataAuthorizationRepository.java`：多维度查询（专家+用户+大棚+状态、按状态+过期时间）
  - `ExpertAvailabilityRepository.java`：按专家ID查询
  - `ChatService.java`：核心业务 — createConversation（校验专家角色+创建对话+首条消息）、getConversations（按角色区分用户端/专家端，含未读数和最后消息预览）、getMessages（参与者校验+分页）、sendMessage（对话状态校验+WAITING→ACTIVE自动转换）、sendSnapshot（调用SensorDataService获取实时数据生成快照JSON）、closeConversation、getUnreadCount
  - `ChatController.java`：7个端点 — POST /api/v1/chat/conversations（创建）、GET /conversations（列表）、GET /conversations/{id}/messages（历史）、POST /messages（发送）、POST /snapshot（快照）、PUT /conversations/{id}/close（关闭）、GET /unread（未读数）
  - 4个DTO：ConversationRequest（@Valid）、ConversationResponse（含unreadCount/lastMessage）、MessageResponse（fromEntity转换）、SendMessageRequest
  - `ExpertService.java`：核心业务 — getExpertList（专家列表含在线状态）、requestAuthorization（防重复有效授权）、getPendingAuthorizations、approveAuthorization（设置7天过期）、rejectAuthorization、revokeAuthorization、getActiveAuthorizations、updateOnlineStatus
  - `AuthorizationController.java`：8个端点 — POST /api/v1/expert/authorize/request、GET /pending、PUT /{id}/approve、PUT /{id}/reject、PUT /{id}/revoke、GET /active、GET /history、PUT /status
  - `ExpertController.java`：1个端点 — GET /api/v1/experts（含specialty/onlineOnly筛选）
  - 1个DTO：AuthorizationResponse（含remainingDays计算字段）
- **结果**：专家咨询体系完整 — 用户发起求助→创建对话→实时消息收发→环境快照共享。双轨制授权：快照一次性使用+授权7天持续查看，用户可随时撤销。专家在线状态管理+并发控制
- **文件清单**：
  - `backend/.../entity/ChatConversation.java`
  - `backend/.../entity/ChatMessage.java`
  - `backend/.../entity/DataAuthorization.java`
  - `backend/.../entity/ExpertAvailability.java`
  - `backend/.../repository/ChatConversationRepository.java`
  - `backend/.../repository/ChatMessageRepository.java`
  - `backend/.../repository/DataAuthorizationRepository.java`
  - `backend/.../repository/ExpertAvailabilityRepository.java`
  - `backend/.../module/chat/service/ChatService.java`
  - `backend/.../module/chat/controller/ChatController.java`
  - `backend/.../module/chat/dto/ConversationRequest.java`
  - `backend/.../module/chat/dto/ConversationResponse.java`
  - `backend/.../module/chat/dto/MessageResponse.java`
  - `backend/.../module/chat/dto/SendMessageRequest.java`
  - `backend/.../module/expert/service/ExpertService.java`
  - `backend/.../module/expert/controller/AuthorizationController.java`
  - `backend/.../module/expert/controller/ExpertController.java`
  - `backend/.../module/expert/dto/AuthorizationResponse.java`

### 步骤17：C15 多模态融合分析 | ✅ 完成

- **时间**：02:20
- **操作**：
  - `HealthAssessment.java`：JPA实体，对应 DB 第18号表 — 环境健康分/视觉健康分/天气风险/修正因子/综合评分(0-100)/分析JSON/建议措施，内嵌 ScoreLevel 枚举（5级：健康/良好/关注/警告/危险）
  - `HealthAssessmentRepository.java`：大棚最新评估查询、时间范围分页查询、24小时计数
  - `EnvironmentHealthCalculator.java`：环境健康分计算器（权重60%）— 合规率(50%)：11种传感器参数×各组比对用户自定义阈值或系统默认阈值，偏离度公式 max(0, value/min) 或 max(0, max/value)；趋势稳定性(30%)：InfluxDB 查询30分钟数据 stddev → 方差归一化 stability=1/(1+variance/range²)；组间一致性(20%)：多组同参数标准差 → consistency=1/(1+stdDev/range×0.3)
  - `VisualHealthCalculator.java`：视觉健康分计算器（权重40%）— 病害评分(60%)：查询最近诊断记录，NORMAL→1.0，有病害→1.0-(1-confidence)×(1-severityFactor)，6种病害类别(FUNGAL/BACTERIAL/VIRAL/PEST/NUTRIENT/NORMAL)各有严重性因子；长势评分(40%)：预留 Phase 4 GrowthAssessment 接口，当前默认0.8
  - `WeatherRiskCalculator.java`：天气风险修正因子 — 极端天气(暴雨/暴雪/大风/冰雹/极端温度)→0.7，连续3天高温(>35°C)→0.75，轻微不利(多云/小雨/阴)→0.9，适宜→1.0，无数据→1.0。解析 forecastJson 检测连续高温天数
  - `HealthAssessmentService.java`：核心融合引擎 — calculateAndSave() 执行6步流程：计算环境分→视觉分→天气修正→加权融合 overall=(env×0.6+visual×0.4)×weatherFactor→存储→WebSocket推送→低分预警(score<40→CRITICAL, <60→WARNING)。getCurrentScore() 30分钟缓存优先。建议生成：按环境/视觉/天气维度分别生成中文建议。天气位置解析：city > province > location > "北京"
  - `HealthController.java`：3个端点 — GET /api/v1/health/score?greenhouseId=（30分钟缓存优先+实时计算回退）、GET /api/v1/health/history（分页+默认7天）、GET /api/v1/health/detail/{id}
  - 2个DTO：HealthScoreResponse（含level/levelColor/analysis JSON）、HealthHistoryResponse
- **结果**：后端全部22个模块完成！多模态融合引擎将环境时序数据(60%)+视觉诊断数据(40%)+天气修正因子进行跨模态加权融合，生成0-100分综合健康评分。5级评分体系（绿/蓝/黄/橙/红）直观展示大棚健康状态。低分自动触发CRITICAL/WARNING预警。WebSocket实时推送评分变更。VisualHealthCalculator 预留 Phase 4 长势评估接口。
- **文件清单**：
  - `backend/.../entity/HealthAssessment.java`
  - `backend/.../repository/HealthAssessmentRepository.java`
  - `backend/.../module/health/service/EnvironmentHealthCalculator.java`
  - `backend/.../module/health/service/VisualHealthCalculator.java`
  - `backend/.../module/health/service/WeatherRiskCalculator.java`
  - `backend/.../module/health/service/HealthAssessmentService.java`
  - `backend/.../module/health/controller/HealthController.java`
  - `backend/.../module/health/dto/HealthScoreResponse.java`
  - `backend/.../module/health/dto/HealthHistoryResponse.java`

### 步骤18：数据模拟脚本 + F01实时数据看板 + F09个人中心 | ✅ 完成

- **时间**：03:00
- **操作**：
  - `tools/sensor_simulator.py`：Python ESP32 数据模拟器 — 实时模式：通过 MQTT 协议（paho-mqtt）连接 Mosquitto Broker，模拟 3 组设备 × 11 种传感器参数定时上报，数据完全走真实链路 MQTT→Spring Boot→InfluxDB+WebSocket+预警。历史模式：直接写入 InfluxDB，含昼夜温度曲线模拟。用法：`python3 sensor_simulator.py --mode both --days 7`
  - Android 项目骨架：Gradle 8.5 + AGP 8.2，target SDK 34，min SDK 26
  - `TokenManager.java`：SharedPreferences 存储 JWT Token + 用户信息（userId/username/role/realName）
  - `ApiClient.java`：Retrofit 单例封装，OkHttp 拦截器自动注入 Bearer Token，ExecutorService 线程池（4线程）
  - `GreenhouseApiService.java`：Retrofit 接口定义，覆盖 C1 认证/C3 大棚/C5 时序/C6 预警/C15 健康评分/C22 生长周期
  - `StompClient.java`：STOMP over OkHttp WebSocket 实现 — 手动构建 STOMP 1.2 帧（CONNECT/SUBSCRIBE/MESSAGE/DISCONNECT），10 秒心跳，符合规范要求
  - 数据模型 10 个：ApiResponse/PageResult/LoginRequest/LoginResponse/UserInfo/Greenhouse/SensorRealtimeData/SensorDataPoint/AlertItem/HealthScoreData/CropCycleData
  - `GreenhouseRepository.java`：数据仓库层 — ExecutorService 后台执行 Retrofit 同步请求，Handler 回调主线程更新 UI
  - `LoginViewModel.java`：登录业务逻辑 — 输入校验 + API 调用 + Token 保存，Activity 只负责 UI
  - `DashboardViewModel.java`：看板业务逻辑 — 大棚列表加载、传感器数据获取、WebSocket 订阅、健康评分
  - `LoginActivity.java`：登录页面（MVVM：观察 ViewModel 的 LiveData，不写业务逻辑）
  - `MainActivity.java`：底部 Tab 导航（看板/预警/诊断/问答/我的），Fragment 切换
  - `DashboardFragment.java`：实时数据看板 — 大棚 Spinner 选择 + RecyclerView 展示 11 种传感器卡片 + 健康评分圆形指示器
  - `SensorAdapter.java`：传感器数据 RecyclerView 适配器 — 中文名映射、单位显示、异常值红色标记
  - `ProfileFragment.java`：个人中心 — 显示用户名/角色，退出登录（清除 Token 跳转登录页）
  - 资源文件：6 个 Vector 图标、5 个布局 XML、3 个样式文件、底部菜单、自适应启动图标
- **结果**：**BUILD SUCCESSFUL** — APK 编译通过（app-debug.apk, 7.2MB）。APP 端具备登录认证、大棚选择、11 种传感器实时数据展示、健康评分展示、个人中心等核心功能。数据模拟脚本可独立运行，为后续 APP 开发和测试提供真实数据流。MVVM 架构完整：View(Activity/Fragment)→ViewModel→Repository→ApiService。
- **文件清单**：
  - `tools/sensor_simulator.py`
  - `tools/README.md`
  - `build.gradle`（项目级）
  - `settings.gradle`
  - `gradle.properties`
  - `gradle/wrapper/gradle-wrapper.properties`
  - `gradle/wrapper/gradle-wrapper.jar`
  - `gradlew`
  - `app/build.gradle`
  - `app/proguard-rules.pro`
  - `app/src/main/AndroidManifest.xml`
  - `app/.../GreenhouseApplication.java`
  - `app/.../data/local/TokenManager.java`
  - `app/.../data/api/ApiClient.java`
  - `app/.../data/api/GreenhouseApiService.java`
  - `app/.../data/model/` (10 个模型文件)
  - `app/.../data/repository/GreenhouseRepository.java`
  - `app/.../websocket/StompClient.java`
  - `app/.../viewmodel/LoginViewModel.java`
  - `app/.../viewmodel/DashboardViewModel.java`
  - `app/.../ui/login/LoginActivity.java`
  - `app/.../ui/common/MainActivity.java`
  - `app/.../ui/dashboard/DashboardFragment.java`
  - `app/.../ui/profile/ProfileFragment.java`
  - `app/.../adapter/SensorAdapter.java`
  - `app/src/main/res/` (20 个资源文件)

---

### 步骤19：F02 环境预警中心 | ✅ 完成

- **时间**：04:30
- **操作**：
  - `AlertViewModel.java`：预警业务逻辑 — 分页加载预警列表、三级筛选（INFO/WARNING/CRITICAL）、标记已读、自定义阈值 CRUD
  - `AlertAdapter.java`：预警列表适配器 — 级别颜色编码（红=CRITICAL/橙=WARNING/蓝=INFO）、未读圆点标记、相对时间格式化
  - `AlertFragment.java`：预警列表页面 — ChipGroup 三级筛选（全部/警告/严重）、点击跳转详情、阈值设置按钮入口
  - `AlertDetailActivity.java`：预警详情页 — 接收 Intent 传递数据，显示级别标签/传感器信息/内容详情
  - `ThresholdSettingsActivity.java`：阈值编辑页 — 合并默认传感器类型与已保存阈值、逐项编辑 min/max、批量保存
  - `ThresholdAdapter.java`：阈值编辑适配器 — EditText 焦点监听自动更新模型、8 种传感器类型中文名映射
  - 布局文件：fragment_alert.xml（ChipGroup 筛选栏+RecyclerView+空状态/加载）、item_alert_card.xml（级别标签+标题+内容+传感器信息+时间+未读点）、activity_alert_detail.xml（详情布局）、activity_threshold_settings.xml（阈值编辑）、item_threshold_edit.xml（阈值编辑卡片）
  - 修改文件：GreenhouseApiService.java（新增阈值 CRUD+级别筛选接口）、GreenhouseRepository.java（新增预警标记已读+阈值方法）、MainActivity.java（AlertFragment 替换占位）、AndroidManifest.xml（注册 AlertDetailActivity + ThresholdSettingsActivity）
- **结果**：**BUILD SUCCESSFUL** — APK 编译通过。F02 预警中心功能完整：三级筛选、分页加载、详情查看、标记已读、自定义阈值编辑保存。修复了 `android:chipBackgroundColor` → `app:chipBackgroundColor` 的 Material Chip 属性前缀错误。
- **文件清单**：
  - `app/.../viewmodel/AlertViewModel.java`
  - `app/.../adapter/AlertAdapter.java`
  - `app/.../adapter/ThresholdAdapter.java`
  - `app/.../ui/alert/AlertFragment.java`
  - `app/.../ui/alert/AlertDetailActivity.java`
  - `app/.../ui/alert/ThresholdSettingsActivity.java`
  - `app/src/main/res/layout/fragment_alert.xml`
  - `app/src/main/res/layout/item_alert_card.xml`
  - `app/src/main/res/layout/activity_alert_detail.xml`
  - `app/src/main/res/layout/activity_threshold_settings.xml`
  - `app/src/main/res/layout/item_threshold_edit.xml`
  - `app/src/main/res/drawable/ic_alert_level_info.xml`
  - `app/src/main/res/drawable/ic_alert_level_critical.xml`
  - `app/.../data/api/GreenhouseApiService.java`（修改）
  - `app/.../data/repository/GreenhouseRepository.java`（修改）
  - `app/.../ui/common/MainActivity.java`（修改）
  - `app/src/main/AndroidManifest.xml`（修改）

---

### 步骤20：F03 病虫害拍照诊断 | ✅ 完成

- **时间**：05:30
- **操作**：
  - `DiagnosisResponse.java`：诊断结果模型 — 含 getConfidenceText()（百分比格式化）、getConfidenceLevel()（绿≥80%/黄70-80%/红<70% 三级颜色）
  - `DiagnosisHistoryItem.java`：历史列表项模型 — 含 fromResponse() 转换方法、置信度颜色等级
  - `DiagnosisViewModel.java`：诊断业务逻辑 — 图片上传诊断、历史记录分页加载、低置信度判断。符合规范：不持有 Context/ContentResolver 等 Android 组件
  - `DiagnosisHistoryAdapter.java`：历史列表适配器 — Glide 缩略图加载、置信度颜色圆点、点击跳转详情
  - `DiagnosisFragment.java`：诊断主页 — 拍照按钮 + 相册按钮（AndroidX Activity Result API）、RecyclerView 历史列表、图片 Uri→File 压缩（子线程 IO）→ ViewModel 上传。符合规范：Fragment 只负责 UI 和导航
  - `DiagnosisResultActivity.java`：诊断结果页 — 图片展示（Glide）、病害名称 + 置信度（进度条+颜色）、防治方案、低置信度求助专家按钮（预埋跳转，F10 对接）
  - 布局文件：fragment_diagnosis.xml（操作区+历史列表+空状态）、activity_diagnosis_result.xml（图片+结果卡片+防治方案卡片+求助按钮）、item_diagnosis_history.xml（缩略图+病害名+置信度+时间）
  - 资源文件：ic_camera.xml、ic_gallery.xml、ic_alert_level_critical.xml、bg_thumbnail_placeholder.xml、bg_confidence_dot.xml、file_paths.xml（FileProvider）、colors.xml（新增 confidence_high/medium/low）
  - 修改文件：GreenhouseApiService.java（新增 @Multipart diagnose + getDiagnosisHistory）、GreenhouseRepository.java（新增 diagnose() + getDiagnosisHistory()）、MainActivity.java（诊断 Tab 切换到 DiagnosisFragment）、AndroidManifest.xml（注册 DiagnosisResultActivity + FileProvider）
- **结果**：**BUILD SUCCESSFUL** — APK 编译通过。F03 诊断功能完整：拍照/相册选取→图片压缩（1024px/JPEG 80%）→上传 API→置信度展示（三级颜色）+ 防治方案。低置信度（<70%）显示求助专家按钮（预埋跳转）。历史记录分页列表 + 空状态。分享导出功能标记不开发。
- **文件清单**：
  - `app/.../data/model/DiagnosisResponse.java`
  - `app/.../data/model/DiagnosisHistoryItem.java`
  - `app/.../viewmodel/DiagnosisViewModel.java`
  - `app/.../adapter/DiagnosisHistoryAdapter.java`
  - `app/.../ui/diagnosis/DiagnosisFragment.java`
  - `app/.../ui/diagnosis/DiagnosisResultActivity.java`
  - `app/src/main/res/layout/fragment_diagnosis.xml`
  - `app/src/main/res/layout/activity_diagnosis_result.xml`
  - `app/src/main/res/layout/item_diagnosis_history.xml`
  - `app/src/main/res/drawable/ic_camera.xml`
  - `app/src/main/res/drawable/ic_gallery.xml`
  - `app/src/main/res/drawable/ic_alert_level_critical.xml`
  - `app/src/main/res/drawable/bg_thumbnail_placeholder.xml`
  - `app/src/main/res/drawable/bg_confidence_dot.xml`
  - `app/src/main/res/xml/file_paths.xml`
  - `app/src/main/res/values/colors.xml`（修改）
  - `app/.../data/api/GreenhouseApiService.java`（修改）
  - `app/.../data/repository/GreenhouseRepository.java`（修改）
  - `app/.../ui/common/MainActivity.java`（修改）
  - `app/src/main/AndroidManifest.xml`（修改）

---

## Step 21: F04 AI智能问答模块开发 (2026-07-13)

**状态**: ✅ 完成

### 功能概述
开发APP端AI智能问答模块，支持文字提问和语音输入两种方式，集成RAG知识检索与TTS语音播报。

### 实现内容

#### 1. 数据模型层
- `app/.../data/model/QaRequest.java` — 文字问答请求模型（question + greenhouseId）
- `app/.../data/model/QaResponse.java` — 问答响应模型，含SourceInfo内部类（title/category来源引用）、isVoiceInput()辅助方法
- `app/.../data/model/QaHistoryItem.java` — 历史记录模型，含isVoiceInput()标识

#### 2. ViewModel层（纯业务逻辑，无Context依赖）
- `app/.../viewmodel/QaViewModel.java` — 核心逻辑：
  - `askQuestion(String)` — 文字问答，自动追加用户消息+AI回复到聊天列表
  - `askVoice(File)` — 语音问答，Multipart上传音频文件
  - `loadHistory()` / `loadMoreHistory()` — 分页加载历史记录
  - `speakAnswer(String)` — TTS语音播报（通过TextToSpeech，由Fragment注入）
  - ChatMessage内部类：TYPE_USER（用户绿色气泡）、TYPE_AI（AI白色气泡）、TYPE_ERROR（红色错误提示）

#### 3. RecyclerView适配器
- `app/.../adapter/ChatAdapter.java` — 多ViewType聊天适配器：
  - UserViewHolder：绿色气泡右对齐，支持语音输入提示
  - AiViewHolder：白色气泡左对齐，支持来源引用展开+TTS播放按钮
  - ErrorViewHolder：居中红色错误文本

#### 4. UI层（纯展示，业务逻辑在ViewModel）
- `app/.../ui/qa/QaFragment.java` — 智能问答主页面：
  - EditText输入框 + 发送按钮
  - MediaRecorder语音录制（AAC格式），AndroidX ActivityResultContracts.RequestPermission权限申请
  - TextToSpeech初始化（中文Locale），注入ViewModel
  - RecyclerView自动滚动到最新消息
- 布局文件：`fragment_qa.xml`、`item_chat_bubble_user.xml`、`item_chat_bubble_ai.xml`、`item_chat_bubble_error.xml`

#### 5. 资源文件
- `ic_mic.xml` — 麦克风图标
- `ic_send.xml` — 发送图标
- `ic_volume_up.xml` — 音量/TTS图标
- `bg_chat_bubble_user.xml` — 用户气泡背景（绿色圆角）
- `bg_chat_bubble_ai.xml` — AI气泡背景（白色圆角）
- `bg_input_qa.xml` — 输入区域背景

#### 6. 网络层
- `GreenhouseApiService.java`（修改）— 新增接口：
  - `POST /api/qa/ask` — 文字问答
  - `POST /api/qa/ask/voice` (@Multipart) — 语音问答
  - `GET /api/qa/history` — 问答历史（分页）
- `GreenhouseRepository.java`（修改）— 新增 ask/askVoice/getQaHistory 仓库方法

#### 7. 导航集成
- `MainActivity.java`（修改）— QA Tab导航到QaFragment

### 开发规范遵循
- ✅ ViewModel不含Context/ContentResolver，TTS由Fragment注入
- ✅ 网络请求在ExecutorService后台线程（Repository层）
- ✅ Activity/Fragment纯展示，业务逻辑在ViewModel
- ✅ 权限申请使用AndroidX ActivityResultContracts API
- ✅ 语音录制使用系统MediaRecorder（AAC格式），TTS使用系统TextToSpeech
- ✅ RecyclerView多ViewType模式，支持聊天消息差异化展示

### 变更文件清单
- `app/.../data/model/QaRequest.java`（新增）
- `app/.../data/model/QaResponse.java`（新增）
- `app/.../data/model/QaHistoryItem.java`（新增）
- `app/.../viewmodel/QaViewModel.java`（新增）
- `app/.../adapter/ChatAdapter.java`（新增）
- `app/.../ui/qa/QaFragment.java`（新增）
- `app/src/main/res/layout/fragment_qa.xml`（新增）
- `app/src/main/res/layout/item_chat_bubble_user.xml`（新增）
- `app/src/main/res/layout/item_chat_bubble_ai.xml`（新增）
- `app/src/main/res/layout/item_chat_bubble_error.xml`（新增）
- `app/src/main/res/drawable/ic_mic.xml`（新增）
- `app/src/main/res/drawable/ic_send.xml`（新增）
- `app/src/main/res/drawable/ic_volume_up.xml`（新增）
- `app/src/main/res/drawable/bg_chat_bubble_user.xml`（新增）
- `app/src/main/res/drawable/bg_chat_bubble_ai.xml`（新增）
- `app/src/main/res/drawable/bg_input_qa.xml`（新增）
- `app/.../data/api/GreenhouseApiService.java`（修改）
- `app/.../data/repository/GreenhouseRepository.java`（修改）
- `app/.../ui/common/MainActivity.java`（修改）

---

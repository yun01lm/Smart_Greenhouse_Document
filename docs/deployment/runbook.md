---
id: DEPLOY-002
title: 运维手册
type: deployment
module: deployment
tags: [ops, runbook]
status: draft
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
---

# 运维手册

> 本文档描述智慧大棚AIoT系统的日常运维操作、故障处理和监控方案。
> 当前为框架文档，具体内容后续补充。

---

## 日常巡检

### 每日检查清单

| 检查项 | 检查方法 | 正常标准 | 异常处理 |
|--------|----------|----------|----------|
| 容器运行状态 | `docker compose ps` | 所有服务状态为 `Up` | 重启异常容器 |
| 磁盘使用率 | `df -h` | 使用率 < 80% | 清理日志或扩容 |
| 内存使用率 | `free -h` | 可用内存 > 20% | 检查内存泄漏 |
| API服务可用性 | `curl localhost:8080/actuator/health` | HTTP 200 | 查看后端日志 |
| MQTT连接数 | Mosquitto日志 | 与设备数一致 | 检查离线设备 |
| 外部API调用量 | 系统监控面板 | 在日配额内 | 关注超额风险 |

### 每周检查清单

| 检查项 | 检查方法 | 正常标准 | 异常处理 |
|--------|----------|----------|----------|
| 数据库备份 | 检查备份文件 | 最近7天有完整备份 | 手动执行备份 |
| 日志文件大小 | `du -sh logs/` | 单文件 < 500MB | 配置日志轮转 |
| InfluxDB数据量 | InfluxDB UI | 数据连续无断点 | 检查MQTT消费 |
| Chroma向量库状态 | API调用 | 知识库查询正常 | 重建索引 |
| Docker镜像更新 | `docker compose pull --dry-run` | 无安全漏洞告警 | 计划更新 |

### 每月检查清单

| 检查项 | 检查方法 | 正常标准 | 异常处理 |
|--------|----------|----------|----------|
| 系统安全更新 | `apt list --upgradable` | 无高危漏洞 | 计划更新 |
| API Key有效性 | 各平台控制台 | 所有Key有效 | 及时续期 |
| 传感器精度 | 与标准仪器对比 | 偏差在允许范围 | 校准或更换 |
| 性能基线 | 与上月对比 | 无明显下降 | 分析瓶颈 |

> [待补充] 自动化巡检脚本

---

## 日志查看

### 日志文件位置

| 服务 | 日志路径 | 说明 |
|------|----------|------|
| Spring Boot | `logs/spring-boot.log` | 后端应用日志 |
| MySQL | Docker logs | 数据库日志 |
| InfluxDB | Docker logs | 时序数据库日志 |
| Mosquitto | `logs/mosquitto.log` | MQTT Broker日志 |
| Nginx | `logs/nginx/` | Web访问日志和错误日志 |
| frp | `logs/frp.log` | 内网穿透日志 |

### 常用日志查看命令

```bash
# 查看所有Docker服务日志（实时）
docker compose logs -f

# 查看指定服务日志
docker compose logs -f spring-boot
docker compose logs -f mosquitto

# 查看Spring Boot应用日志
tail -f logs/spring-boot.log

# 按级别过滤日志
grep "ERROR" logs/spring-boot.log
grep "WARN" logs/spring-boot.log

# 按时间范围查看
grep "2026-07-11" logs/spring-boot.log

# 查看Nginx访问日志
tail -f logs/nginx/access.log

# 统计API调用次数
grep "/api/v1/" logs/nginx/access.log | wc -l
```

### 关键日志监控关键词

| 关键词 | 含义 | 处理 |
|--------|------|------|
| `ERROR` | 系统异常 | 立即排查 |
| `BusinessException` | 业务异常 | 分析频率和原因 |
| `MQTT connection lost` | MQTT断连 | 检查Broker和网络 |
| `AI_RECOGNITION_FAILED` | AI识别失败 | 检查API配额和网络 |
| `OutOfMemoryError` | 内存溢出 | 重启服务并分析原因 |
| `Connection refused` | 连接拒绝 | 检查目标服务状态 |
| `Token expired` | Token过期 | 正常现象，关注频率 |

> [待补充] 日志聚合方案（ELK/Loki）和告警规则配置

---

## 数据备份

### 备份策略

| 数据 | 备份方式 | 频率 | 保留期限 |
|------|----------|------|----------|
| MySQL | `mysqldump` | 每日凌晨2点 | 30天 |
| InfluxDB | `influx backup` | 每日凌晨3点 | 30天 |
| Chroma | 文件系统备份 | 每周日 | 4周 |
| 上传文件 | `rsync` | 每日 | 30天 |
| 配置文件 | Git + 文件备份 | 每次变更 | 永久 |

### MySQL备份脚本

```bash
#!/bin/bash
# backup-mysql.sh
BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="smart_greenhouse"

mkdir -p $BACKUP_DIR

docker compose exec -T mysql mysqldump \
  -u root -p${MYSQL_PASSWORD} \
  --single-transaction \
  --routines \
  --triggers \
  $DB_NAME | gzip > $BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz

# 删除30天前的备份
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete
```

### InfluxDB备份脚本

```bash
#!/bin/bash
# backup-influxdb.sh
BACKUP_DIR="/backup/influxdb"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

docker compose exec -T influxdb influx backup \
  $BACKUP_DIR/${DATE} \
  -t ${INFLUX_TOKEN}

# 删除30天前的备份
find $BACKUP_DIR -mindepth 1 -maxdepth 1 -type d -mtime +30 -exec rm -rf {} \;
```

### Chroma备份脚本

```bash
#!/bin/bash
# backup-chroma.sh
BACKUP_DIR="/backup/chroma"
DATE=$(date +%Y%m%d)

mkdir -p $BACKUP_DIR

tar -czf $BACKUP_DIR/chroma_${DATE}.tar.gz \
  docker/volumes/chroma/

# 保留最近4周
find $BACKUP_DIR -name "chroma_*.tar.gz" -mtime +28 -delete
```

### 备份验证

```bash
# 定期验证备份文件完整性
gunzip -t /backup/mysql/smart_greenhouse_latest.sql.gz
tar -tzf /backup/chroma/chroma_latest.tar.gz > /dev/null
```

> [待补充] 自动备份定时任务配置（crontab）和备份恢复测试流程

---

## 故障恢复

### 常见故障及处理

#### 1. 服务容器异常退出

**症状**：`docker compose ps` 显示某服务状态为 `Exit` 或 `Restarting`

**处理步骤**：

```bash
# 1. 查看异常容器日志
docker compose logs --tail=100 <service-name>

# 2. 尝试重启
docker compose restart <service-name>

# 3. 如果重启失败，重建容器
docker compose up -d --force-recreate <service-name>

# 4. 如果是依赖服务问题，重启全部
docker compose down && docker compose up -d
```

#### 2. MySQL连接失败

**症状**：Spring Boot日志出现 `CommunicationsException` 或 `Connection refused`

**处理步骤**：

```bash
# 1. 检查MySQL容器状态
docker compose ps mysql

# 2. 检查MySQL是否可连接
docker compose exec mysql mysqladmin -u root -p${MYSQL_PASSWORD} ping

# 3. 查看MySQL错误日志
docker compose logs mysql --tail=50

# 4. 如果磁盘满，清理binlog
docker compose exec mysql mysql -u root -p${MYSQL_PASSWORD} \
  -e "PURGE BINARY LOGS BEFORE NOW() - INTERVAL 7 DAY;"

# 5. 重启MySQL
docker compose restart mysql
```

#### 3. MQTT消息丢失

**症状**：传感器数据在InfluxDB中不连续，有缺失时段

**处理步骤**：

```bash
# 1. 检查Mosquitto运行状态
docker compose ps mosquitto

# 2. 检查订阅者连接
docker compose exec mosquitto mosquitto_sub -t '$SYS/broker/clients/connected' -C 1

# 3. 检查Spring Boot MQTT连接日志
grep "MQTT" logs/spring-boot.log | tail -20

# 4. 重启MQTT消费模块（重启Spring Boot）
docker compose restart spring-boot

# 5. 检查ESP32设备连接状态
# 在系统监控面板查看设备在线列表
```

#### 4. 磁盘空间不足

**症状**：日志中出现 `No space left on device`，服务异常

**处理步骤**：

```bash
# 1. 检查磁盘使用
df -h

# 2. 查找大文件
du -sh /* 2>/dev/null | sort -rh | head -20

# 3. 清理Docker资源
docker system prune -a --volumes -f

# 4. 清理旧日志
find logs/ -name "*.log" -mtime +7 -delete

# 5. 清理旧备份（保留最近7天）
find /backup/ -mtime +7 -delete

# 6. 检查InfluxDB数据保留策略
docker compose exec influxdb influx bucket list
```

#### 5. 外部API调用失败

**症状**：AI诊断/问答/语音功能不可用

**处理步骤**：

```bash
# 1. 检查网络连通性
curl -I https://api.deepseek.com
curl -I https://api.siliconflow.cn

# 2. 检查API Key是否有效
# 在管理端 AI引擎管理页面查看各引擎状态

# 3. 检查API配额
# 登录对应平台控制台查看调用量和余额

# 4. 如果是超时问题，检查网络延迟
ping api.deepseek.com

# 5. 临时切换备用引擎（如已实现）
# 在管理端切换AI引擎配置
```

#### 6. frp内网穿透断连

**症状**：公网无法访问系统

**处理步骤**：

```bash
# 1. 检查frp客户端状态
ps aux | grep frpc

# 2. 查看frp日志
tail -f logs/frp.log

# 3. 重启frp客户端
kill $(pgrep frpc)
./frpc -c frpc.toml &

# 4. 检查frp服务端状态（需要服务端权限）
# 联系服务器管理员
```

### 紧急恢复流程

```
故障发生
    │
    ▼
1. 确认故障范围（哪个服务/功能异常）
    │
    ▼
2. 查看对应服务日志，定位原因
    │
    ├── 已知故障 → 按上述流程处理
    │
    ├── 未知故障 → 执行全量重启
    │       docker compose down
    │       docker compose up -d
    │
    ▼
3. 验证恢复
    │
    ├── 恢复成功 → 记录故障原因和解决方案
    │
    └── 恢复失败 → 从备份恢复数据
            ├── 恢复MySQL备份
            ├── 恢复InfluxDB备份
            └── 重新部署
```

> [待补充] 详细故障处理流程、数据恢复步骤和应急联系人

---

## 性能监控

### 监控指标

| 类别 | 指标 | 告警阈值 | 说明 |
|------|------|----------|------|
| 系统 | CPU使用率 | > 80% 持续5分钟 | Docker宿主机 |
| 系统 | 内存使用率 | > 85% | Docker宿主机 |
| 系统 | 磁盘使用率 | > 80% | Docker宿主机 |
| 应用 | API响应时间(P99) | > 3000ms | Spring Boot |
| 应用 | API错误率 | > 5% | 5分钟内 |
| 应用 | JVM堆内存使用率 | > 80% | Spring Boot |
| 数据库 | MySQL连接数 | > 80%最大连接数 | MySQL |
| 数据库 | 慢查询 | > 1000ms | MySQL |
| MQTT | 离线设备数 | > 30%总设备数 | Mosquitto |
| MQTT | 消息积压 | > 1000条 | 消费延迟 |
| AI | API调用失败率 | > 10% | 5分钟内 |
| AI | API调用配额 | > 80%日配额 | 各平台 |

### 监控工具

| 工具 | 用途 | 配置 |
|------|------|------|
| Spring Boot Actuator | 应用健康检查和指标 | 已内置 |
| Docker Stats | 容器资源使用 | `docker stats` |
| Prometheus + Grafana | 全面监控（推荐） | [待补充] |
| MySQL慢查询日志 | 数据库性能 | 已开启 |

### 性能优化建议

| 场景 | 优化方案 |
|------|----------|
| API响应慢 | 增加数据库索引、使用缓存、优化查询 |
| 时序数据查询慢 | 调整InfluxDB保留策略、使用聚合查询 |
| 图片上传慢 | 压缩图片、使用CDN、异步处理 |
| MQTT消息积压 | 增加消费者线程、优化写入批处理 |
| 内存不足 | 调整JVM参数、排查内存泄漏 |

> [待补充] Prometheus + Grafana 监控方案配置、告警规则定义、性能基线数据

---

## 附录

### 常用运维命令速查

```bash
# 服务管理
docker compose up -d              # 启动所有服务
docker compose down               # 停止所有服务
docker compose restart <service>  # 重启指定服务
docker compose logs -f <service>  # 查看服务日志
docker compose ps                 # 查看服务状态
docker compose exec <service> sh  # 进入容器Shell

# 系统检查
df -h                             # 磁盘使用
free -h                           # 内存使用
top                               # CPU和进程
uptime                            # 系统运行时间
docker stats                      # 容器资源使用

# 网络检查
curl localhost:8080/actuator/health  # API健康检查
ping api.deepseek.com                # 外网连通性
netstat -tlnp                        # 端口监听

# 日志
tail -f logs/spring-boot.log         # 应用日志
docker compose logs -f               # 所有容器日志
journalctl -u docker -f              # Docker系统日志
```

### 应急联系方式

| 角色 | 联系方式 | 职责 |
|------|----------|------|
| 系统管理员 | [待补充] | 服务器运维、网络管理 |
| 后端开发 | [待补充] | Spring Boot应用维护 |
| 硬件维护 | [待补充] | ESP32设备维护 |
| AI平台对接 | [待补充] | 百度AI/讯飞/DeepSeek账号管理 |

> [待补充] 完整的应急联系人和升级流程

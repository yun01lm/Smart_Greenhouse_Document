---
id: TASK-G01
title: 数据总览大屏开发
module: G1 Web总览
type: task
priority: high
status: completed
created: 2026-07-14
started: 2026-07-14
completed: 2026-07-14
assignee: AI助手
related_docs:
  - DEVLOG-步骤32
tags: [web, vue3, dashboard, echarts, element-plus]
---

# 数据总览大屏开发

---

## 任务描述

### 背景

G01 是 Web 管理端的第一个模块，搭建 Vue 3 项目框架并实现大棚数据总览大屏。作为 Web 端"地基"，后续 G02-G10 都在此框架上扩展。

### 目标

搭建 Vue 3 + Element Plus 项目，实现数据总览大屏：传感器实时数据卡片、趋势曲线图、健康评分、预警列表、天气信息。

### 范围

Web 前端（Vue 3），独立于后端和 APP 端。

---

## 验收标准

1. [x] Vue 3 + Vite + Element Plus + ECharts + Axios + Vue Router + Pinia 项目骨架
2. [x] 登录页面（用户名/密码，对接后端 /auth/login）
3. [x] Token 管理（Axios 拦截器自动附加 + 401 跳转登录）
4. [x] 主布局（顶部导航 + 大棚选择器 + 左侧菜单 + 内容区）
5. [x] 8 个传感器指标卡片（温度/湿度/CO₂/光照/土壤温度/湿度/pH/风速）
6. [x] ECharts 双轴趋势曲线图（温度+湿度，近24h）
7. [x] 健康评分展示（总体评分 + 环境/视觉进度条）
8. [x] 预警列表（最新5条，三级颜色区分）
9. [x] 天气卡片
10. [x] WebSocket 实时推送
11. [x] 构建成功（vite build）

---

## 关键修改

| 文件路径 | 修改类型 | 变更说明 |
|----------|----------|----------|
| `web/package.json` | 新建 | 项目配置 + 依赖声明 |
| `web/vite.config.js` | 新建 | Vite 配置 + API/WebSocket 代理 |
| `web/index.html` | 新建 | HTML 入口 |
| `web/src/main.js` | 新建 | Vue 应用入口，注册 Pinia/Router/ElementPlus |
| `web/src/App.vue` | 新建 | 根组件 |
| `web/src/assets/main.css` | 新建 | 全局样式（暗色大屏主题） |
| `web/src/router/index.js` | 新建 | 路由配置 + 登录守卫 |
| `web/src/stores/auth.js` | 新建 | Pinia 认证状态管理 |
| `web/src/utils/request.js` | 新建 | Axios 封装（拦截器 + 错误处理） |
| `web/src/utils/websocket.js` | 新建 | WebSocket STOMP 客户端 |
| `web/src/api/greenhouse.js` | 新建 | 大棚 API |
| `web/src/api/sensor.js` | 新建 | 传感器 API |
| `web/src/api/health.js` | 新建 | 健康评分 API |
| `web/src/api/alert.js` | 新建 | 预警 API |
| `web/src/api/weather.js` | 新建 | 天气 API |
| `web/src/views/Login.vue` | 新建 | 登录页 |
| `web/src/layouts/MainLayout.vue` | 新建 | 主布局（导航+菜单+内容） |
| `web/src/views/dashboard/DashboardPage.vue` | 新建 | 大屏主页（组装所有组件） |
| `web/src/views/dashboard/SensorCards.vue` | 新建 | 传感器卡片组件 |
| `web/src/views/dashboard/TrendChart.vue` | 新建 | ECharts 趋势图组件 |
| `web/src/views/dashboard/HealthScore.vue` | 新建 | 健康评分组件 |
| `web/src/views/dashboard/AlertList.vue` | 新建 | 预警列表组件 |

---

## 技术架构

```
web/src/
├── api/                    # API 封装（按模块分文件）
│   ├── greenhouse.js
│   ├── sensor.js
│   ├── health.js
│   ├── alert.js
│   └── weather.js
├── assets/
│   └── main.css            # 全局样式
├── layouts/
│   └── MainLayout.vue      # 主布局
├── router/
│   └── index.js            # 路由 + 守卫
├── stores/
│   └── auth.js             # Pinia 认证
├── utils/
│   ├── request.js          # Axios 封装
│   └── websocket.js        # WebSocket 客户端
├── views/
│   ├── Login.vue           # 登录页
│   └── dashboard/
│       ├── DashboardPage.vue  # 大屏主页
│       ├── SensorCards.vue    # 传感器卡片
│       ├── TrendChart.vue     # 趋势图
│       ├── HealthScore.vue    # 健康评分
│       └── AlertList.vue      # 预警列表
├── App.vue
└── main.js
```

### 数据流

```
REST API (并发)                WebSocket STOMP
    │                               │
    ├── GET /sensor/realtime ───┐   ├── /topic/greenhouse/{id}/realtime
    ├── GET /health/score ─────┤   │
    ├── GET /alerts ───────────┤   │
    ├── GET /alerts/unread ────┤   │
    └── GET /weather/current ──┘   │
        │                           │
        ▼                           ▼
   Promise.allSettled          realtimeClient.onMessage
        │                           │
        ▼                           ▼
    reactive data ──────────→  DashboardPage ─→ 子组件
                                     │
                             30s 轮询兜底
```

---

## 完成结果

Web 端 Vue 3 项目搭建完成，数据总览大屏包含 8 传感器卡片 + 趋势图 + 健康评分 + 预警列表 + 天气卡片。双通道数据获取（REST 并发 + WebSocket 实时 + 轮询兜底），暗色大屏风格，构建成功。

---

## 阻塞记录

无阻塞。

---

## 备注

- 完成日期：2026-07-14
- 关联DEVLOG：步骤32
- 左侧菜单 G02-G10 已预设但 disabled，后续逐步启用
- dist 产物约 2.3MB（gzip 约 540KB）

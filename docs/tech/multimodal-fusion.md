---
id: TECH-008
title: 多模态融合分析技术设计文档
type: tech-design
module: multimodal-fusion
tags:
  - 多模态融合
  - 健康评分
  - 环境分析
  - 图像分析
  - 天气预测
  - 加权评分
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - /document/docs/prd/health-assessment.md
  - /document/docs/prd/growth-assessment.md
  - /document/docs/api/health/health-api.md
  - /document/docs/api/growth/growth-api.md
---

## 概述

多模态融合分析模块是系统创新点三的核心实现，将环境时序数据、作物图像长势数据和天气预报数据进行跨模态融合，生成综合健康评分（0-100分）。该评分综合反映大棚作物的整体健康状态，为农户提供直观的决策参考，并在评分低于阈值时触发预警。

**创新点依据**（申报书原文）：
> "将环境时序特征与视觉长势特征进行融合分析"

**评分维度与权重**：
- **环境健康分 (60%)**：基于11种传感器参数的阈值合规率和趋势稳定性
- **视觉健康分 (40%)**：基于病虫害诊断结果和作物长势评估
- **天气风险因子**：基于未来天气预报的辅助修正系数

## 架构设计

### 融合引擎架构

```
┌─────────────────────────────────────────────────────────────────┐
│                    多模态融合分析引擎 (C15)                        │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     数据采集层                             │   │
│  │                                                           │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │   │
│  │  │ C5 时序数据   │  │ C8 诊断结果  │  │ C14 天气数据  │    │   │
│  │  │ (InfluxDB)   │  │ (MySQL)      │  │ (MySQL)       │    │   │
│  │  │              │  │              │  │               │    │   │
│  │  │ 11种环境参数  │  │ 病害置信度   │  │ 温度/湿度/    │    │   │
│  │  │ + 组维度数据  │  │ 防治方案     │  │ 风力/天气码   │    │   │
│  │  │ + 历史趋势    │  │ 长势评估     │  │ 预报数据      │    │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘    │   │
│  │         │                 │                  │             │   │
│  └─────────┼─────────────────┼──────────────────┼─────────────┘   │
│            │                 │                  │                  │
│            ▼                 ▼                  ▼                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     分析计算层                             │   │
│  │                                                           │   │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────┐  │   │
│  │  │ 环境健康评分器  │  │ 视觉健康评分器  │  │ 天气修正器  │  │   │
│  │  │                │  │                │  │            │  │   │
│  │  │ • 阈值合规率   │  │ • 病害检测结果  │  │ • 极端天气  │  │   │
│  │  │ • 趋势稳定性   │  │ • 长势评级     │  │   预警降分  │  │   │
│  │  │ • 参数加权    │  │ • 叶色分析     │  │ • 适宜天气  │  │   │
│  │  │ • 组间一致性   │  │ • 株高/叶面积  │  │   维持评分  │  │   │
│  │  └───────┬────────┘  └───────┬────────┘  └──────┬─────┘  │   │
│  │          │                   │                   │        │   │
│  │          └───────────────────┼───────────────────┘        │   │
│  │                              ▼                            │   │
│  │                  ┌──────────────────┐                     │   │
│  │                  │   加权融合器      │                     │   │
│  │                  │                  │                     │   │
│  │                  │ overall_score =  │                     │   │
│  │                  │ env_score × 0.6  │                     │   │
│  │                  │ + visual_score   │                     │   │
│  │                  │   × 0.4          │                     │   │
│  │                  │ × weather_factor │                     │   │
│  │                  └────────┬─────────┘                     │   │
│  └───────────────────────────┼───────────────────────────────┘   │
│                              │                                    │
│                              ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     输出层                                 │   │
│  │                                                           │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │   │
│  │  │ MySQL存储    │  │ WebSocket    │  │ 预警触发     │    │   │
│  │  │ health_      │  │ 实时推送     │  │ score < 40   │    │   │
│  │  │ assessments  │  │ 评分+建议    │  │ → 触发预警   │    │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘    │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 评分等级定义

| 评分范围 | 等级 | 颜色 | 含义 | 建议动作 |
|----------|------|------|------|----------|
| 80-100 | 健康 | 绿色 | 作物生长状态良好 | 继续保持当前管理 |
| 60-79 | 良好 | 蓝色 | 基本健康，轻微问题 | 关注异常指标 |
| 40-59 | 关注 | 黄色 | 存在明显问题 | 检查具体问题，调整管理 |
| 20-39 | 警告 | 橙色 | 严重问题 | 立即采取措施，考虑求助专家 |
| 0-19 | 危险 | 红色 | 紧急状况 | 紧急处理，立即求助专家 |

### 触发时机

```
评分计算触发时机:
  a) 每次病虫害诊断完成后 → 更新视觉健康分 → 重新计算综合评分
  b) 每次长势评估后 → 更新视觉健康分 → 重新计算综合评分
  c) 定时任务 (每30分钟) → 更新环境健康分 → 重新计算综合评分
  d) 天气数据更新后 (每3小时) → 更新天气修正因子 → 重新计算综合评分
  e) 用户手动请求 → GET /api/v1/health/score → 实时计算
```

## 数据流

### 综合健康评分计算流

```
触发 (定时/事件/手动)
  │
  ├── 步骤1: 计算环境健康分 (权重60%)
  │   ├── 查询InfluxDB: 各组最新传感器数据
  │   ├── 查询预警阈值: 用户自定义 > 系统默认
  │   ├── 计算各参数合规率:
  │   │   for each sensor_type:
  │   │     if min_threshold <= value <= max_threshold → 合规
  │   │     else → 计算偏离度 (0~1)
  │   │   avg_compliance = sum(compliance) / 11  (11种参数)
  │   │
  │   ├── 计算趋势稳定性:
  │   │   查询过去30分钟数据 → 计算各参数方差
  │   │   方差越小越稳定 → stability_score (0~1)
  │   │
  │   ├── 计算组间一致性:
  │   │   各组同参数的标准差 → 标准差越大越不稳定
  │   │   consistency_score (0~1)
  │   │
  │   └── env_score = (compliance_rate × 0.5 + stability_score × 0.3 
  │                    + consistency_score × 0.2) × 100
  │
  ├── 步骤2: 计算视觉健康分 (权重40%)
  │   ├── 查询最新诊断记录: diagnostic_records (最近24小时)
  │   ├── 查询最新长势评估: growth_assessments (最近7天)
  │   ├── 病害评分:
  │   │   if 有病害检测:
  │   │     if diseaseCategory == NORMAL → disease_score = 1.0
  │   │     else → disease_score = 1.0 - (1 - confidence) × severity_factor
  │   │   if 无检测记录 → disease_score = 0.8 (默认假设基本健康)
  │   │
  │   ├── 长势评分:
  │   │   if 有长势评估:
  │   │     growth_score = health_score / 100  (来自growth_assessments)
  │   │   if 无评估 → growth_score = 0.8
  │   │
  │   └── visual_score = (disease_score × 0.6 + growth_score × 0.4) × 100
  │
  ├── 步骤3: 计算天气修正因子
  │   ├── 查询 weather_cache: 未来24小时预报
  │   ├── 极端天气检测:
  │   │   if 暴雨/大风/极端高温/寒潮 → weather_factor = 0.8
  │   │   elif 小雨/阴天/微风 → weather_factor = 0.95
  │   │   else 晴天/适宜 → weather_factor = 1.0
  │   └── 特殊场景:
  │       if 连续3天高温预警 → weather_factor = 0.7
  │
  ├── 步骤4: 融合计算
  │   overall_score = (env_score × 0.6 + visual_score × 0.4) × weather_factor
  │   overall_score = clamp(overall_score, 0, 100)
  │
  └── 步骤5: 输出
      ├── 存储 health_assessments:
      │   { greenhouse_id, env_score, visual_score, weather_risk,
      │     overall_score, analysis_json, recommendations }
      ├── WebSocket推送: /topic/greenhouse/{id}/health
      └── 预警检查:
          if overall_score < 40 → 触发预警 (CRITICAL)
          if overall_score < 60 → 触发预警 (WARNING)
```

### 环境健康分详细计算

```
参数合规率 (compliance_rate):
  对每个传感器组:
    for each sensor_type in [TEMP, HUMIDITY, LIGHT, CO2, O2, 
                              SOIL_TEMP, SOIL_HUMIDITY, EC, N, P, K]:
      threshold = getEffectiveThreshold(sensor_type)
      if threshold == null → skip (该参数未配置阈值)
      
      if value in [min, max]:
        compliance_i = 1.0
      else:
        deviation = max(0, value - max) / max  or  max(0, min - value) / min
        compliance_i = max(0, 1.0 - deviation)
    
    group_compliance = avg(compliance_i)
  
  overall_compliance = avg(all groups' compliance)

趋势稳定性 (stability_score):
  查询过去30分钟数据 → 对每个参数:
    variance = calculate_variance(values)
    stability_i = 1.0 / (1.0 + variance / threshold)
    // 方差越小，稳定性越高
  
  overall_stability = avg(stability_i)

组间一致性 (consistency_score):
  对各组同参数:
    std_dev = calculate_std_dev(group_values)
    consistency_i = 1.0 / (1.0 + std_dev / expected_range)
    // 组间标准差越小，一致性越高
  
  overall_consistency = avg(consistency_i)

env_score = (compliance × 0.5 + stability × 0.3 + consistency × 0.2) × 100
```

### 视觉健康分详细计算

```
病害评分 (disease_score):
  查询最近24小时诊断记录:
    if records.isEmpty():
      disease_score = 0.8  // 无诊断记录，默认基本健康
    
    else:
      // 取最新的诊断结果
      latest = records[0]
      
      if latest.diseaseCategory == NORMAL:
        disease_score = 1.0
      else:
        // 严重性因子: 不同病害类别严重程度不同
        severity_factors = {
          FUNGAL: 0.8,    // 真菌性: 可控制
          BACTERIAL: 0.6, // 细菌性: 较严重
          VIRAL: 0.4,     // 病毒性: 很严重
          PEST: 0.7,      // 虫害: 中等
          NUTRIENT: 0.9   // 营养问题: 较易纠正
        }
        factor = severity_factors[latest.diseaseCategory] || 0.5
        disease_score = 1.0 - (1.0 - latest.confidence) * (1.0 - factor)

长势评分 (growth_score):
  查询最近7天长势评估:
    if assessments.isEmpty():
      growth_score = 0.8
    
    else:
      latest = assessments[0]
      growth_score = latest.healthScore / 100

visual_score = (disease_score × 0.6 + growth_score × 0.4) × 100
```

### 天气修正因子

```
查询最近天气缓存:
  if no weather data → weather_factor = 1.0 (无影响)
  
  forecast = weather_cache.latest()
  
  // 检测极端天气
  extreme_conditions = {
    temperature > 40°C → 极端高温
    temperature < 0°C  → 寒潮
    weather_code in [暴雨, 暴雪, 大风, 冰雹] → 极端天气
    wind_speed > 20m/s → 大风
  }
  
  if any extreme_condition:
    weather_factor = 0.7  // 极端天气显著降低评分
  elif 连续3天高温 (>35°C):
    weather_factor = 0.75
  elif 小雨/阴天/降温:
    weather_factor = 0.9
  else:
    weather_factor = 1.0  // 适宜天气
```

## 接口设计概要

### 健康评分接口

| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/health/score | 当前综合健康评分 |
| GET | /api/v1/health/history | 健康评分历史 |
| GET | /api/v1/health/detail/{id} | 详细评估报告 |

### 健康评分响应

```json
{
  "code": 200,
  "data": {
    "id": 1,
    "greenhouseId": 3,
    "greenhouseName": "1号大棚",
    "overallScore": 72.5,
    "level": "良好",
    "levelColor": "blue",
    "envScore": 78.3,
    "visualScore": 65.0,
    "weatherRisk": "无极端天气",
    "weatherFactor": 1.0,
    "analysis": {
      "complianceRate": 0.82,
      "stabilityScore": 0.75,
      "consistencyScore": 0.88,
      "diseaseScore": 0.60,
      "growthScore": 0.70,
      "anomalies": [
        {
          "sensorType": "TEMP",
          "zoneLabel": "西侧",
          "currentValue": 35.2,
          "threshold": 32.0,
          "deviation": 3.2
        }
      ]
    },
    "recommendations": [
      "西侧区域温度偏高，建议开启通风设备",
      "作物长势评分偏低，建议检查水肥管理",
      "无病害检测，继续保持病虫害监测"
    ],
    "createdAt": "2026-07-11T10:30:00"
  }
}
```

### 历史评分响应

```json
{
  "code": 200,
  "data": {
    "list": [
      { "overallScore": 72.5, "createdAt": "2026-07-11T10:30:00" },
      { "overallScore": 68.2, "createdAt": "2026-07-11T10:00:00" },
      { "overallScore": 75.1, "createdAt": "2026-07-11T09:30:00" }
    ],
    "total": 48,
    "trend": "STABLE"
  }
}
```

### WebSocket推送格式

```
/topic/greenhouse/{id}/health:
{
  "greenhouseId": 3,
  "overallScore": 72.5,
  "level": "良好",
  "levelColor": "blue",
  "envScore": 78.3,
  "visualScore": 65.0,
  "weatherRisk": "无极端天气",
  "recommendations": ["..."],
  "updatedAt": "2026-07-11T10:30:00"
}
```

## 关键算法/逻辑

### 加权融合算法

```java
@Service
public class MultimodalFusionService {
    
    public HealthAssessment calculateHealthScore(Long greenhouseId) {
        // 1. 计算各维度分数
        double envScore = environmentHealthCalculator.calculate(greenhouseId);
        double visualScore = visualHealthCalculator.calculate(greenhouseId);
        double weatherFactor = weatherRiskCalculator.calculate(greenhouseId);
        
        // 2. 加权融合
        double overallScore = (envScore * 0.6 + visualScore * 0.4) * weatherFactor;
        overallScore = Math.max(0, Math.min(100, overallScore)); // clamp
        
        // 3. 生成建议
        List<String> recommendations = generateRecommendations(
            greenhouseId, envScore, visualScore, weatherFactor);
        
        // 4. 存储评估记录
        HealthAssessment assessment = new HealthAssessment();
        assessment.setGreenhouseId(greenhouseId);
        assessment.setEnvScore(BigDecimal.valueOf(envScore));
        assessment.setVisualScore(BigDecimal.valueOf(visualScore));
        assessment.setWeatherRisk(determineWeatherRisk(weatherFactor));
        assessment.setOverallScore(BigDecimal.valueOf(overallScore));
        assessment.setAnalysisJson(buildAnalysisJson());
        assessment.setRecommendations(String.join("; ", recommendations));
        assessment.setCreatedAt(Instant.now());
        assessment = healthAssessmentRepository.save(assessment);
        
        // 5. WebSocket推送
        HealthScoreVO vo = convertToVO(assessment);
        messagingTemplate.convertAndSend(
            "/topic/greenhouse/" + greenhouseId + "/health", vo);
        
        // 6. 低分预警
        if (overallScore < 40) {
            triggerLowScoreAlert(greenhouseId, assessment);
        }
        
        return assessment;
    }
}
```

### 环境健康分计算器

```java
@Component
public class EnvironmentHealthCalculator {
    
    public double calculate(Long greenhouseId) {
        // 获取各组最新传感器数据
        Map<String, Map<String, Double>> groupData = 
            sensorDataRepository.queryLatestByGreenhouse(greenhouseId);
        
        // 1. 合规率计算 (权重50%)
        double complianceRate = calculateComplianceRate(greenhouseId, groupData);
        
        // 2. 趋势稳定性 (权重30%)
        double stabilityScore = calculateStability(greenhouseId);
        
        // 3. 组间一致性 (权重20%)
        double consistencyScore = calculateConsistency(groupData);
        
        return (complianceRate * 0.5 + stabilityScore * 0.3 + consistencyScore * 0.2) * 100;
    }
    
    private double calculateComplianceRate(Long greenhouseId, 
                                            Map<String, Map<String, Double>> groupData) {
        double totalCompliance = 0;
        int count = 0;
        
        for (Map.Entry<String, Map<String, Double>> group : groupData.entrySet()) {
            String groupId = group.getKey();
            for (Map.Entry<String, Double> sensor : group.getValue().entrySet()) {
                String sensorType = sensor.getKey();
                double value = sensor.getValue();
                
                // 获取有效阈值
                Threshold threshold = thresholdService.getEffectiveThreshold(
                    greenhouseId, groupId, sensorType);
                if (threshold == null) continue;
                
                double compliance;
                if (threshold.getMinThreshold() != null && value < threshold.getMinThreshold()) {
                    compliance = Math.max(0, value / threshold.getMinThreshold());
                } else if (threshold.getMaxThreshold() != null && value > threshold.getMaxThreshold()) {
                    compliance = Math.max(0, threshold.getMaxThreshold() / value);
                } else {
                    compliance = 1.0;
                }
                
                totalCompliance += compliance;
                count++;
            }
        }
        
        return count > 0 ? totalCompliance / count : 0.8;
    }
}
```

### 建议生成逻辑

```java
private List<String> generateRecommendations(Long greenhouseId, 
                                              double envScore, double visualScore, 
                                              double weatherFactor) {
    List<String> recommendations = new ArrayList<>();
    
    // 环境问题建议
    if (envScore < 60) {
        // 查找具体超限参数
        List<SensorAnomaly> anomalies = findAnomalies(greenhouseId);
        for (SensorAnomaly anomaly : anomalies) {
            switch (anomaly.getSensorType()) {
                case "TEMP":
                    recommendations.add(anomaly.getZoneLabel() + "区域温度" + 
                        (anomaly.isOverMax() ? "偏高" : "偏低") + 
                        "，建议调整通风/加热设备");
                    break;
                case "HUMIDITY":
                    recommendations.add("湿度异常，建议检查灌溉和通风系统");
                    break;
                case "CO2":
                    recommendations.add("CO2浓度异常，建议检查通风情况");
                    break;
                // ... 其他参数
            }
        }
    }
    
    // 视觉健康建议
    if (visualScore < 60) {
        DiagnosticRecord latestDiagnosis = diagnosisRepository
            .findLatestByGreenhouseId(greenhouseId);
        if (latestDiagnosis != null && latestDiagnosis.getDiseaseName() != null) {
            recommendations.add("检测到" + latestDiagnosis.getDiseaseName() + 
                "，建议按防治方案处理");
        }
    }
    
    // 天气风险建议
    if (weatherFactor < 0.9) {
        recommendations.add("未来24小时有恶劣天气风险，建议提前做好防护措施");
    }
    
    // 综合建议
    if (recommendations.isEmpty()) {
        recommendations.add("作物生长状态良好，继续保持当前管理措施");
    }
    
    return recommendations;
}
```

## 技术选型理由

| 决策项 | 选择 | 理由 |
|--------|------|------|
| 评分模型 | 加权融合 | 申报书创新点三要求"环境时序特征+视觉长势特征融合"，加权方式直观可解释 |
| 权重分配 | 环境60% + 视觉40% | 环境数据实时性强（30秒更新），视觉数据更新频率低（截帧30分钟/次，诊断按需） |
| 天气因子 | 乘法修正 | 天气作为外部不可控因素，对健康状态的影响是全局性的，乘法修正比加法更合理 |
| 评估频率 | 30分钟定时 + 事件触发 | 平衡计算开销和评分时效性 |
| 评分范围 | 0-100整数 | 直观易懂，适合APP端仪表盘展示 |
| 组间一致性 | 作为环境评分子因子 | 多组传感器是大棚的特点，组间差异反映环境均匀性 |

## 注意事项

1. **权重可配置**：环境健康分(60%)和视觉健康分(40%)的权重应通过配置文件管理，而非硬编码。后续可根据实际运行数据调整权重比例。

2. **数据缺失处理**：如果某维度数据缺失（如无诊断记录、无长势评估、无天气数据），该维度使用默认值（0.8）而非报错。确保评分计算不因部分数据缺失而失败。

3. **评分历史保留**：health_assessments表需要保留所有历史评分记录，用于趋势分析和模型优化。不建议定期清理。

4. **低分预警联动**：综合评分 < 40 时自动触发CRITICAL预警， < 60 时触发WARNING预警。预警内容应包含具体的扣分原因和建议。

5. **评分计算幂等性**：同一时间点的重复计算应返回相同结果。评分依赖的数据（传感器值、诊断结果）在短时间内不变，计算结果应一致。

6. **组维度的重要性**：环境健康分计算必须考虑多组传感器的差异。如果只有某一组参数异常，全棚平均可能掩盖问题。组间一致性因子可捕获此类情况。

7. **严重性因子表**：不同病害类别的严重性因子（severity_factor）需要定期校准。初期使用经验值，后期根据专家反馈和实际防治效果调整。

8. **性能优化**：环境健康分计算需要查询InfluxDB（各组最新值+30分钟趋势），视觉健康分需要查询MySQL（诊断记录+长势评估），天气修正需要查询MySQL（天气缓存）。建议使用缓存减少重复查询，缓存有效期=评分计算周期（30分钟）。

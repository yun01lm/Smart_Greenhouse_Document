---
id: TECH-005
title: AI能力层技术设计文档
type: tech-design
module: ai-layer
tags:
  - AI抽象层
  - 策略模式
  - 图像识别
  - 语音识别
  - RAG
  - 知识图谱
  - Spring AI
status: final
created: 2026-07-11
last_modified: 2026-07-11
author: AI助手
related_docs:
  - /document/docs/prd/smart-greenhouse-prd.md
  - /document/docs/api/api-design.md
---

## 概述

AI能力层是系统的智能核心，通过策略模式设计统一的AI抽象层，实现外部API与自训练模型的无缝切换。覆盖病虫害图像识别、方言语音识别、RAG知识问答和轻量级知识图谱四大能力。初期使用百度AI、讯飞、DeepSeek等外部API快速上线，后期通过Colab训练的ResNet、Whisper模型替换，业务代码无需修改。

**设计原则**：
- 接口统一：所有AI能力通过Provider接口暴露，实现类可热切换
- API先行：初期调用外部API，快速验证业务流程
- 预留自训练：接口设计兼容本地模型推理，后期替换零改动
- 配置驱动：通过 application.yml 切换Provider实现，无需改代码

## 架构设计

### AI抽象层整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        AI 抽象层                                 │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    Provider 接口层                         │   │
│  │                                                           │   │
│  │  DiseaseRecognitionProvider    SpeechRecognitionProvider   │   │
│  │  ├── recognize(byte[])        ├── recognize(byte[])       │   │
│  │  └── getEngineName()          ├── getEngineName()         │   │
│  │                               └── getSupportedDialects()  │   │
│  └──────────────────────┬───────────────────────────────────┘   │
│                         │                                        │
│  ┌──────────────────────┼───────────────────────────────────┐   │
│  │               Provider 实现层 (策略模式)                    │   │
│  │                                                           │   │
│  │  初期 (API)              后期 (自训练模型)                  │   │
│  │  ┌──────────────────┐   ┌──────────────────┐              │   │
│  │  │ BaiduRecognition │   │ ResNetRecognition │              │   │
│  │  │ Provider         │   │ Provider         │              │   │
│  │  │ (百度API)        │   │ (本地ResNet)     │              │   │
│  │  └──────────────────┘   └──────────────────┘              │   │
│  │  ┌──────────────────┐   ┌──────────────────┐              │   │
│  │  │ XunfeiSpeech     │   │ WhisperSpeech    │              │   │
│  │  │ Provider         │   │ Provider         │              │   │
│  │  │ (讯飞API)        │   │ (本地Whisper)    │              │   │
│  │  └──────────────────┘   └──────────────────┘              │   │
│  │                                                           │   │
│  │  Spring AI RAG (不变)    轻量级知识图谱 (Chroma检索)       │   │
│  │  ┌──────────────────┐   ┌──────────────────┐              │   │
│  │  │ DeepSeek Chat    │   │ KnowledgeGraph   │              │   │
│  │  │ + Chroma检索      │   │ Service          │              │   │
│  │  │ + bge-m3向量化    │   │ (病害→防治方案)   │              │   │
│  │  └──────────────────┘   └──────────────────┘              │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   配置切换 (application.yml)                │   │
│  │  ai.image.provider: baidu | resnet                        │   │
│  │  ai.voice.provider: xunfei | whisper                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Provider接口定义

**图像识别接口**：
```java
public interface DiseaseRecognitionProvider {
    /**
     * 识别病虫害
     * @param imageData 图片字节数据
     * @return 识别结果，包含病害名称、置信度、防治方案
     * @throws BusinessException(ErrorCode.AI_RECOGNITION_FAILED)
     */
    DiseaseRecognitionResult recognize(byte[] imageData);
    
    /**
     * 获取当前使用的识别引擎名称
     */
    String getEngineName();
}
```

**语音识别接口**：
```java
public interface SpeechRecognitionProvider {
    /**
     * 识别语音（支持方言）
     * @param audioData 音频字节数据 (wav/mp3/amr)
     * @return 识别结果，包含文本和置信度
     * @throws BusinessException(ErrorCode.AI_SPEECH_FAILED)
     */
    SpeechRecognitionResult recognize(byte[] audioData);
    
    String getEngineName();
    List<String> getSupportedDialects();
}
```

### Spring AI RAG架构

```
┌─────────────────────────────────────────────────────────────┐
│                      RAG 问答管线                             │
│                                                              │
│  用户提问 (文字/语音转文字)                                    │
│      │                                                       │
│      ▼                                                       │
│  ┌──────────────┐                                            │
│  │ 问题向量化    │ ← SiliconFlow bge-m3 (1024维)              │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                            │
│  │ Chroma检索   │ ← 相似度检索 top-5 相关文档片段             │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                            │
│  │ Prompt组装   │ ← 系统提示词 + 检索上下文 + 用户问题        │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                            │
│  │ DeepSeek生成 │ ← deepseek-chat 模型                       │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  返回: {答案 + 引用来源}                                      │
└─────────────────────────────────────────────────────────────┘
```

### 轻量级知识图谱架构

采用向量检索 + 结构化分类体系代替传统图数据库：

```
病害类型分类体系 (层级关系):
  
  真菌性病害
  ├── 霜霉病 → 防治方案: 农业防治 + 化学防治(百菌清) + ...
  ├── 白粉病 → 防治方案: 生物防治 + 化学防治(三唑酮) + ...
  └── 灰霉病 → 防治方案: ...
  
  细菌性病害
  ├── 青枯病 → 防治方案: ...
  └── 软腐病 → 防治方案: ...
  
  虫害
  ├── 蚜虫 → 防治方案: ...
  ├── 白粉虱 → 防治方案: ...
  └── 红蜘蛛 → 防治方案: ...

诊断流程:
  视觉AI识别 → 病害类型 + 置信度
    → 以病害类型为节点，Chroma检索关联知识
    → 组装结构化防治方案:
        TreatmentDetail {
          agricultural: "清除病残体，轮作..."
          physical: "黄板诱杀..."
          biological: "释放天敌..."
          chemical: "百菌清800倍液..."
          precautions: ["安全间隔期7天", "勿与碱性农药混用"]
        }
```

## 数据流

### 病虫害诊断流

```
APP拍照上传 → POST /api/v1/diagnosis/recognize (multipart)
  → 图片存储 (本地文件系统)
  → AI抽象层调用:
      DiseaseRecognitionProvider.recognize(imageData)
        ├── 初期: BaiduRecognitionProvider → 百度植物识别API
        │   └── 返回: { diseaseName, confidence, description }
        └── 后期: ResNetRecognitionProvider → 本地模型推理
            └── 返回: { diseaseName, confidence, description }
  → 轻量级知识图谱检索:
      KnowledgeGraphService.getTreatment(diseaseName, diseaseCategory)
        → Chroma检索: {category: diseaseCategory, disease: diseaseName}
        → 组装结构化 TreatmentDetail
  → 组装最终结果:
      DiseaseRecognitionResult {
        diseaseName, confidence, description,
        treatment, treatmentDetail,
        recognitionEngine: "baidu" | "resnet"
      }
  → 存储 diagnostic_records
  → 判断置信度:
      ├── confidence >= 0.7 → 正常返回
      └── confidence < 0.7 → 结果附加"建议求助专家"标记
  → 触发多模态融合模块更新健康评分
```

### RAG问答流

```
APP输入 (文字或语音转文字后) → POST /api/v1/qa/ask
  → 问题向量化:
      EmbeddingModel.embed(question)
        → SiliconFlow bge-m3 API → 1024维向量
  → Chroma向量检索:
      collection.query(query_embeddings, n_results=5)
        → 返回top-5相关文档片段
  → Prompt组装:
      """
      你是一个专业的农业助手，请根据以下知识库内容回答问题。
      
      知识库参考内容:
      {retrieved_context}
      
      用户问题: {question}
      
      请用简洁的中文回答，并标注信息来源。
      """
  → DeepSeek API调用:
      ChatClient.call(prompt)
        → 返回生成的答案
  → 存储 qa_records
  → 返回: { answer, sources: [{title, category}] }
```

### 语音识别流

```
APP录音上传 → POST /api/v1/qa/ask/voice (multipart, audio/wav)
  → 音频存储 (本地文件系统)
  → AI抽象层调用:
      SpeechRecognitionProvider.recognize(audioData)
        ├── 初期: XunfeiSpeechProvider → 讯飞方言ASR API
        │   └── 返回: { text, rawDialectText, confidence, dialect: "hebei" }
        └── 后期: WhisperSpeechProvider → 本地Whisper推理
            └── 返回: { text, confidence, dialect: "hebei" }
  → 识别文本作为问题输入
  → 进入RAG问答流 (流2)
  → 返回: { question: 识别文本, answer: RAG答案, asrEngine, asrConfidence }
```

### AI引擎切换流

```
管理员Web端切换AI引擎 → PUT /api/v1/admin/ai/config
  → 请求体: {"image_provider": "resnet", "voice_provider": "whisper"}
  → 更新系统配置 (或通过环境变量+重启)
  → Spring Boot重新加载配置:
      @ConditionalOnProperty(name="ai.image.provider", havingValue="resnet")
      → ResNetRecognitionProvider Bean 激活
      → BaiduRecognitionProvider Bean 失效
  → 无需重启 (热切换) 或 重启生效
```

## 接口设计概要

### 病虫害诊断接口

| 方法 | 端点 | 说明 |
|------|------|------|
| POST | /api/v1/diagnosis/recognize | 上传图片识别 (multipart) |
| GET | /api/v1/diagnosis/records | 诊断历史 |

### AI问答接口

| 方法 | 端点 | 说明 |
|------|------|------|
| POST | /api/v1/qa/ask | 文字问答 |
| POST | /api/v1/qa/ask/voice | 语音问答 (multipart) |
| GET | /api/v1/qa/records | 问答历史 |

### AI引擎管理接口（Web管理端）

| 方法 | 端点 | 说明 |
|------|------|------|
| GET | /api/v1/admin/ai/config | 当前AI引擎配置 |
| PUT | /api/v1/admin/ai/config | 切换AI引擎 |
| GET | /api/v1/admin/ai/status | 各引擎状态和调用量 |

### 返回值DTO

**DiseaseRecognitionResult**：
```java
@Data
public class DiseaseRecognitionResult {
    private String diseaseName;         // 病害名称
    private String diseaseCategory;     // 分类: FUNGAL/BACTERIAL/VIRAL/PEST/NUTRIENT/NORMAL
    private Double confidence;          // 置信度 0.0-1.0
    private String description;         // 病害描述
    private String treatment;           // 综合防治方案
    private TreatmentDetail treatmentDetail; // 结构化方案
    private String recognitionEngine;   // 引擎: baidu/resnet
    private Boolean needExpert;         // 是否需要求助专家 (confidence<0.7)
    
    @Data
    public static class TreatmentDetail {
        private String agricultural;    // 农业防治
        private String physical;        // 物理防治
        private String biological;      // 生物防治
        private String chemical;        // 化学防治
        private List<String> precautions; // 注意事项
    }
}
```

**SpeechRecognitionResult**：
```java
@Data
public class SpeechRecognitionResult {
    private String text;                // 识别文本（普通话）
    private String rawDialectText;      // 方言原文（可空）
    private Double confidence;          // 置信度
    private String dialect;             // 方言类型: hebei
    private String recognitionEngine;   // 引擎: xunfei/whisper
    private Integer durationMs;         // 音频时长
}
```

## 关键算法/逻辑

### AI Provider条件装配

```java
// 图像识别 - 百度API实现 (初期)
@Component
@ConditionalOnProperty(name = "ai.image.provider", havingValue = "baidu", matchIfMissing = true)
public class BaiduRecognitionProvider implements DiseaseRecognitionProvider {
    @Override
    public DiseaseRecognitionResult recognize(byte[] imageData) {
        // 1. 获取百度Access Token
        String token = getBaiduAccessToken();
        // 2. 调用植物识别API
        // 3. 解析响应 → DiseaseRecognitionResult
        // 4. 失败 → throw BusinessException(ErrorCode.AI_RECOGNITION_FAILED)
    }
}

// 图像识别 - ResNet自训练 (后期替换)
@Component
@ConditionalOnProperty(name = "ai.image.provider", havingValue = "resnet")
public class ResNetRecognitionProvider implements DiseaseRecognitionProvider {
    @Override
    public DiseaseRecognitionResult recognize(byte[] imageData) {
        // 1. 图片预处理 (resize 224x224, normalize)
        // 2. 本地模型推理 (DJL/ONNX Runtime)
        // 3. 后处理 → DiseaseRecognitionResult
    }
}
```

### 外部API调用重试策略

```java
@Component
public class AiApiRetryHandler {
    
    @Retryable(
        retryFor = {AiServiceException.class},
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000, multiplier = 2)
    )
    public <T> T callWithRetry(Supplier<T> apiCall, String serviceName) {
        try {
            return apiCall.get();
        } catch (Exception e) {
            log.error("{} API调用失败，将重试: {}", serviceName, e.getMessage());
            throw new AiServiceException(serviceName + " 服务异常", e);
        }
    }
    
    @Recover
    public <T> T recover(AiServiceException e, String serviceName) {
        log.error("{} API重试3次后仍失败", serviceName);
        if (serviceName.contains("百度")) {
            throw new BusinessException(ErrorCode.AI_RECOGNITION_FAILED);
        } else if (serviceName.contains("讯飞")) {
            throw new BusinessException(ErrorCode.AI_SPEECH_FAILED);
        } else {
            throw new BusinessException(ErrorCode.AI_LLM_FAILED);
        }
    }
}
```

### RAG知识库索引构建

```java
@Service
public class KnowledgeIndexService {
    
    public void indexDocument(KnowledgeDocument doc) {
        // 1. 文档切片 (每片约500字，重叠50字)
        List<String> chunks = textSplitter.split(doc.getContent(), 500, 50);
        
        // 2. 批量向量化
        List<List<Double>> embeddings = embeddingModel.embed(chunks);
        
        // 3. 存入Chroma
        for (int i = 0; i < chunks.size(); i++) {
            Map<String, Object> metadata = Map.of(
                "document_id", doc.getId(),
                "title", doc.getTitle(),
                "category", doc.getCategory(),
                "crop_type", doc.getCropType(),
                "chunk_index", i
            );
            chromaCollection.add(
                UUID.randomUUID().toString(),
                embeddings.get(i),
                chunks.get(i),
                metadata
            );
        }
        
        // 4. 更新文档索引状态
        doc.setChunkCount(chunks.size());
        doc.setVectorIndexed(true);
        knowledgeDocRepository.save(doc);
    }
}
```

## 技术选型理由

| 决策项 | 选择 | 理由 |
|--------|------|------|
| 策略模式 | Provider接口 + @ConditionalOnProperty | 申报书要求ResNet/ViT+LSTM+Whisper，初期需API快速上线 |
| 百度AI (初期) | 植物识别API | 成熟稳定，病虫害种类覆盖全，API调用简单 |
| 讯飞 (初期) | 方言ASR API | 支持河北话方言识别，申报书"方言语音交互" |
| DeepSeek | deepseek-chat | 极低成本(1元/百万token)，中文能力强，OpenAI兼容 |
| SiliconFlow bge-m3 | Embedding服务 | DeepSeek无Embedding API；1024维向量，免费额度充足 |
| Spring AI | RAG框架 | Java原生RAG，不引入Python服务；统一技术栈 |
| 轻量级知识图谱 | Chroma检索+层级分类 | 不引入Neo4j等图数据库，一人开发运维成本可控 |
| DJL/ONNX Runtime | 本地模型推理 (后期) | Java原生深度学习框架，支持ONNX格式模型部署 |

## 注意事项

1. **禁止绕过AI抽象层**：所有AI能力调用必须通过Provider接口，禁止在业务代码中直接调用百度API或讯飞API。这是实现API→自训练模型无缝切换的前提。

2. **API Key安全**：百度、讯飞、DeepSeek、SiliconFlow的API Key必须通过环境变量注入，禁止硬编码，禁止提交到Git。配置文件使用 `${ENV_VAR:placeholder}` 格式。

3. **外部API超时与重试**：所有外部API调用必须设置连接超时5s、读取超时10s。失败后自动重试最多3次（指数退避），全部失败后返回友好错误提示（不暴露原始错误信息）。

4. **知识图谱非真图数据库**：本方案采用"轻量级知识图谱"——以Chroma向量检索+结构化分类体系模拟图谱关系，不是真正的图遍历。答辩时可解释为"基于向量检索的轻量级知识图谱实现"，区别于传统Neo4j方案。

5. **DeepSeek不提供Embedding**：必须配置 `spring.ai.openai.embedding.enabled=false`，单独创建EmbeddingModel Bean指向SiliconFlow。Chat和Embedding使用不同的API端点。

6. **RAG引用来源**：AI问答返回结果必须包含引用来源（文档标题、分类），增强答案可信度。qa_records.sources字段存储JSON格式的来源信息。

7. **低置信度引导专家**：诊断置信度<70%时，结果中必须标记 needExpert=true，APP端展示"建议求助专家"按钮。这是专家咨询系统的入口之一。

8. **模型文件管理**：ResNet和Whisper的模型权重文件（.pt/.onnx）不提交到Git，通过Git LFS或独立存储管理。部署时通过Docker volume挂载到容器内。

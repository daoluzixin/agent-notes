# RAG 完整知识笔记

---

## 一、概述与全链路

RAG（检索增强生成）：用户提问 → 检索知识库相关片段 → 注入 Prompt → LLM 基于真实数据生成。解决知识截止、幻觉、无法访问私有数据。

**演进：** Naive RAG（检索-拼接-生成）→ Advanced RAG（Query 改写、混合检索、Rerank）→ Modular RAG（可插拔模块、Agent 动态决策）

```
文档 → Chunking → Embedding → 向量数据库
用户查询 → Query改写 → 向量化 → 混合检索 → Rerank → Prompt → LLM生成
```

---

## 二、文档处理（Chunking + Metadata）

### 切分策略

| 策略 | 做法 | 适用 |
|------|------|------|
| 递归切分 | 按 `\n\n` → `\n` → `。` → 字符数优先级递归 | **默认选择**，80%+ 文档 |
| 结构化切分 | 按 Markdown 标题/HTML 标签 | 技术文档、Wiki |
| 语义切分 | 相邻句子 Embedding 相似度断裂点切分 | 仅对召回差的文档定向使用 |
| 固定大小+Overlap | 按固定 token 数，重叠 50-100 tokens | 快速原型 |

Chunk 大小 256-1024 tokens，Overlap 10%-20%。表格处理：小表整表入库，中表摘要+原表，大表 Text-to-SQL。

### 元信息

每个 chunk 携带统一 Schema 的 metadata，提供向量相似度之外的过滤维度。核心字段：source/doc_id（溯源）、doc_type/tags（路由）、heading_path（层级）、时间戳（时效过滤）、access_level（权限隔离）。过滤方式以 Pre-filtering 为主。

---

## 三、Embedding 与向量检索

### 向量化流程

```
文本 → 子词分词(BPE/WordPiece) → Transformer编码 → Mean Pooling → 固定维度向量
```

中文场景推荐 **BGE-M3**：同时产出 Dense/Sparse/ColBERT 三种向量，原生中文，8192 tokens，天然支持混合检索。

### ANN 算法

| 算法 | 原理 | 适用规模 |
|------|------|----------|
| **HNSW** | 多层图结构，顶层粗定位逐层精搜。最主流 | < 1亿 |
| IVF | K-Means 聚簇，只搜最近 nprobe 个簇 | 中等 |
| IVF-PQ | IVF + 乘积量化压缩 | > 1亿，内存有限 |

相似度度量以余弦相似度为主。

### 向量数据库

核心原则：能用现有基础设施就不引新组件。

| 数据库 | 定位 | 特点 | 适用场景 |
|--------|------|------|----------|
| pgvector | PG 插件 | 运维成本低，SQL 生态完整，事务支持好 | 已有 PG 且 <500万 |
| Elasticsearch 8.x+ | 搜索引擎扩展 | 天然支持 BM25+向量混合检索，无需额外融合层 | 已有 ES，需混合检索 |
| Milvus | 专用分布式 | 支持十亿级，多种索引（HNSW/IVF/PQ），多租户 | 大规模生产环境 |
| Qdrant | 专用高性能 | Rust 编写，丰富过滤条件，支持 mmap 超内存数据 | 追求性能和 API 体验 |
| Pinecone | 全托管云 | 零运维开箱即用，按量付费 | 不想运维，快速验证 |
| Chroma | 轻量本地 | Python 原生，秒级启动 | 本地开发/原型验证 |

---

## 四、Query 改写 + 混合检索 + Rerank

### Query 改写

| 方法 | 思路 |
|------|------|
| Multi-Query | LLM 改写为多角度查询，分别检索合并去重 |
| HyDE | LLM 生成假设性回答，用其向量检索（"像答案的文本"离真实文档更近） |

两者都需额外 LLM 调用增加延迟，按需启用。

### 混合检索

向量检索擅长语义匹配，BM25 擅长精确关键词匹配，两路独立召回后用 **RRF** 融合：`RRF_score(d) = Σ 1/(k + rank_i(d))`，k=60。基于排名而非分数，天然解决量纲问题。

### Rerank（多级漏斗）

```
全量(百万) → Bi-Encoder粗排 → Top-100 → Cross-Encoder精排 → Top-10 → LLM
```

- **Bi-Encoder**：Query/Doc 分别编码算相似度，快但粗
- **Cross-Encoder**：Query+Doc 拼接过 Transformer 充分交互，慢但精

模型：BGE-Reranker（开源中文）、Cohere Rerank（商用）。

---

## 五、评估与工程趋势

### RAGAS 四指标（按优先级）

| 指标 | 衡量 |
|------|------|
| **Context Recall** | 检索是否覆盖所需全部信息（最高优先，召回不到一切白搭） |
| Faithfulness | 回答是否忠实上下文，有无编造 |
| Context Precision | 检索结果中相关文档占比 |
| Answer Relevance | 回答是否切题 |

实践：构建 Golden Dataset（人工标注 100-500 问答对）→ RAGAS 自动评估 → A/B 对比 → 持续监控。

### Rust 在 AI 基础设施的趋势

Qdrant、LanceDB、Swiftide 等用 Rust 重写。本质分工：**应用层 Python 快速迭代，基础设施层 Rust 追求极致性能**（零成本抽象、内存安全、async 高并发、PyO3 绑定 Python）。

---

## 设计 Checklist

| 环节 | 默认方案 |
|------|----------|
| 切分 | 递归切分，512 tokens，Overlap 10%-20% |
| Embedding | BGE-M3，Mean Pooling(对所有token向量取平均) |
| 向量数据库 | 小规模 pgvector，大规模 Milvus/Qdrant |
| 检索 | 混合检索（向量+BM25），RRF 融合 |
| Rerank | BGE-Reranker / Cohere Rerank |
| Query 改写 | 按需 Multi-Query / HyDE |
| 评估 | RAGAS + Golden Dataset |

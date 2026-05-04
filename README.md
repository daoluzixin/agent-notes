# Agent Notes

AI Agent 实践笔记。记录在 LLM 应用开发过程中积累的方法论、工程经验和可复用模板。

---

## 目录

### [RAG_Notes.md](./RAG_Notes.md)

RAG（Retrieval-Augmented Generation）完整知识笔记。覆盖从 Naive RAG 到 Modular RAG 的演进路线，包括文档切分策略、Embedding 模型选型、向量数据库对比、Query 改写、混合检索、Rerank 重排序、上下文压缩、评估体系（RAGAS）等全链路知识点。

### [prompt-engineering/](./prompt-engineering/)

Prompt & Skill 工程方法论。从多轮实战对抗式调优中沉淀出的完整方法论，包含两份文档：

- **[best-practices.md](./prompt-engineering/best-practices.md)** — Prompt 工程（五条调优哲学、骨架结构、对抗式调优完整操作手册、软约束转机制约束）+ Skill 工程（定时任务 vs 会话触发、四环执行闭环、幂等性设计、Linter/Rules 自动化）的最佳实践与检查表。
- **[gan-optimizer-template.md](./prompt-engineering/gan-optimizer-template.md)** — GAN 风格 Prompt 自优化 Agent 模板，可直接复用到任意项目。定义了完整的目录约定、宪法级/法律级约束分层、Bad Case 判定流程、版本保留决策树和停止条件。

---

## License

MIT

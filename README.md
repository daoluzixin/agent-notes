# Agent Notes

AI Agent 实践笔记。记录在 LLM 应用开发过程中积累的方法论、工程经验和可复用模板。

---

## 目录

### [Harness方法论.md](./Harness方法论.md)

Harness 方法论：AI Agent 工程协作的自反馈治理框架。核心公式为 `Harness = 文档约束 + 自反馈闭环`，通过三个 Part 分别解决 Agent 协作中的"跑偏""盲写""自欺"问题。包含完整的落地清单（文档骨架、Linter 脚本、自动注入规则、自反馈工作流、对抗测试双 Agent 模式）和设计原则。

### [RAG工程笔记.md](./RAG工程笔记.md)

RAG（Retrieval-Augmented Generation）完整知识笔记。覆盖从 Naive RAG 到 Modular RAG 的演进路线，包括文档切分策略、Embedding 模型选型、向量数据库对比、Query 改写、混合检索、Rerank 重排序、上下文压缩、评估体系（RAGAS）等全链路知识点。

### [Agent框架核心设计.md](./Agent框架核心设计.md)

Agent 框架核心设计：从架构层面系统梳理核心设计要素——核心循环、工具系统、记忆与上下文管理、规划与推理、多 Agent 协作等子系统的设计决策与工程权衡。

### [Prompt与Skill工程实践.md](./Prompt与Skill工程实践.md)

Prompt & Skill 工程实践方法论：从概率性系统优化的视角出发，覆盖调优五原则、骨架结构设计、对抗式调优操作手册、Skill 执行闭环设计等完整方法论。

### [记忆系统方法论.md](./记忆系统方法论.md)

记忆系统方法论：用纯本地 Markdown 文件 + AGENTS.md 指令自驱，解决 AI 跨会话"失忆"问题的可移植记忆系统。包含数据结构四层设计（磁盘单后端、长期/每日时间分层、晋升机制、写入纪律）、靠 AGENTS.md 驱动的运作闭环，以及零依赖迁移到新机器（如 codex 环境）的完整方案。

### [prompt-engineering/](./prompt-engineering/)

Prompt & Skill 工程方法论。从多轮实战对抗式调优中沉淀出的完整方法论：

- **[Prompt与Skill最佳实践.md](./prompt-engineering/Prompt与Skill最佳实践.md)** — Prompt 工程（五条调优哲学、骨架结构、对抗式调优完整操作手册、软约束转机制约束）+ Skill 工程（定时任务 vs 会话触发、四环执行闭环、幂等性设计、Linter/Rules 自动化）的最佳实践与检查表。
- **[GAN风格Prompt自优化模板.md](./prompt-engineering/GAN风格Prompt自优化模板.md)** — GAN 风格 Prompt 自优化 Agent 模板，可直接复用到任意项目。定义了完整的目录约定、宪法级/法律级约束分层、Bad Case 判定流程、版本保留决策树和停止条件。

---

## License

MIT

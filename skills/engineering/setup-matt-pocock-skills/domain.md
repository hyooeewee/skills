# 领域文档

工程师技能在探索代码库时，应如何使用此仓库的领域文档。

## 探索前请阅读这些

* 仓库根目录下的 **`CONTEXT.md`**，或者
* 仓库根目录下的 **`CONTEXT-MAP.md`**（如果存在）——它指向每个上下文对应的一个 `CONTEXT.md`。阅读与主题相关的每一个文档。
* `docs/adr/` — 阅读涉及您即将工作的领域的 ADR（架构决策记录）。在多上下文仓库中，还应检查 `src/<context>/docs/adr/` 以查找上下文特定的决策。

如果这些文件不存在，**静默继续**。不要标记它们的缺失；不要建议预先创建它们。`/domain-modeling` 技能（通过 `/grill-with-docs` 和 `/improve-codebase-architecture` 达到）会在术语或决策实际解决时延迟创建它们。

## 文件结构

单上下文仓库（大多数仓库）：

```
/
├── CONTEXT.md
├── docs/adr/
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

多上下文仓库（根目录存在 `CONTEXT-MAP.md`）：

```
/
├── CONTEXT-MAP.md
├── docs/adr/                          ← system-wide decisions
└── src/
    ├── ordering/
    │   ├── CONTEXT.md
    │   └── docs/adr/                  ← context-specific decisions
    └── billing/
        ├── CONTEXT.md
        └── docs/adr/
```

## 使用词汇表中的术语

当您的输出命名了一个领域概念（在问题标题、重构提案、假设、测试名称中）时，请使用 `CONTEXT.md` 中定义的术语。不要偏离词汇表明确避免的同义词。

如果您需要的概念尚未在词汇表中，这是一个信号——要么您正在编造项目不使用的语言（重新考虑），要么确实存在真正的空白（将其记录下来以便 `/domain-modeling`）。

## 标记 ADR 冲突

如果您的输出与现有的 ADR 相矛盾，请明确指出，而不是静默地覆盖：

> *与 ADR-0007（事件溯源订单）相矛盾——但值得重新打开，因为……*

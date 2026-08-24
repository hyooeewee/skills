# 领域文档

工程技能在探索代码库时，应如何使用本仓库的领域文档。

## 探索之前，请先阅读这些

* 仓库根目录下的 **`CONTEXT.md`**，或
* 如果存在，请阅读仓库根目录下的 **`CONTEXT-MAP.md`**：它指向每个上下文对应的 `CONTEXT.md`。阅读每个与主题相关的文件。
* **`docs/adr/`**：阅读涉及你即将工作的区域的 ADR。在多上下文仓库中，还要检查 `src/<context>/docs/adr/` 以获取上下文范围的决策。

如果这些文件中有任何不存在，**请静默继续**。不要标记它们的缺失；也不要建议提前创建它们。`/domain-modeling` 技能（通过 `/grill-with-docs` 和 `/improve-codebase-architecture` 触达）会在术语或决策实际得到解决时惰性创建它们。

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

## 使用术语表的词汇

当你的输出命名一个领域概念时（在 issue 标题、重构提案、假设、测试名称中），请使用 `CONTEXT.md` 中定义的术语。不要偏离到术语表明确避免的同义词。

如果你需要的概念尚未在术语表中，这是一个信号：要么是你正在创造项目不使用的语言（请重新考虑），要么确实存在真正的缺口（请在 `/domain-modeling` 中记录）。

## 标记 ADR 冲突

如果你的输出与现有 ADR 矛盾，请显式地将其提出，而不是静默覆盖：

> *与 ADR-0007（事件溯源订单）相矛盾，但值得重新打开，因为……*

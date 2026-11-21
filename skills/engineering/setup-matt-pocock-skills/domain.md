# 领域文档

工程技能在探索代码库时，应如何使用本仓库的领域文档。

## 探索之前，请先阅读这些

* **`GLOSSARY.md`** 位于仓库根目录，或
* **`GLOSSARY-MAP.md`** 位于仓库根目录（如果存在）：它指向每个上下文的一个 `GLOSSARY.md`。阅读与主题相关的每一个。
* **`docs/adr/`**：阅读涉及你即将工作的区域的 ADR。在多上下文仓库中，还要检查 `src/<context>/docs/adr/` 以获取上下文范围的决策。

如果这些文件中有任何不存在，**请静默继续**。不要标记它们的缺失；也不要建议提前创建它们。`/domain-modeling` 技能（通过 `/grill-with-docs` 和 `/improve-codebase-architecture` 触达）会在术语或决策实际得到解决时惰性创建它们。

## 文件结构

单上下文仓库（大多数仓库）：

```
/
├── GLOSSARY.md
├── docs/adr/
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

多上下文仓库（根目录存在 `GLOSSARY-MAP.md`）：

```
/
├── GLOSSARY-MAP.md
├── docs/adr/                          ← system-wide decisions
└── src/
    ├── ordering/
    │   ├── GLOSSARY.md
    │   └── docs/adr/                  ← context-specific decisions
    └── billing/
        ├── GLOSSARY.md
        └── docs/adr/
```

## 使用术语表的词汇

当你的输出提及领域概念时（在议题标题、重构提案、假设、测试名称中），请使用 `GLOSSARY.md` 中定义的术语。不要偏离术语表明确避免的同义词。

如果你需要的概念尚未在术语表中，这是一个信号：要么是你正在创造项目不使用的语言（请重新考虑），要么确实存在真正的缺口（请在 `/domain-modeling` 中记录）。

## 标记 ADR 冲突

如果你的输出与现有 ADR 矛盾，请显式地将其提出，而不是静默覆盖：

> *与 ADR-0007（事件溯源订单）相矛盾，但值得重新打开，因为……*

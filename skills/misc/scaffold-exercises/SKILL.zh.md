---
name: scaffold-exercises
description: 创建通过 linting 的练习目录结构，包含章节、问题、解决方案和解释器。当用户想要搭建练习结构、创建练习存根或设置新课程章节时使用。

---

# 搭建练习

创建通过 `pnpm ai-hero-cli internal lint` 的练习目录结构，然后使用 `git commit` 提交。

## 目录命名

* **章节**：`exercises/` 内部的 `XX-section-name/`（例如 `01-retrieval-skill-building`）
* **练习**：章节内的 `XX.YY-exercise-name/`（例如 `01.03-retrieval-with-bm25`）
* 章节数 = `XX`，练习数 = `XX.YY`
* 名称使用连字符命名法（小写，连字符）

## 练习变体

每个练习至少需要这些子文件夹中的一个：

* `problem/` - 带有 TODO 的学生工作区
* `solution/` - 参考实现
* `explainer/` - 概念性材料，无 TODO

创建存根时，默认使用 `explainer/`，除非计划另有说明。

## 必需的文件

每个子文件夹（`problem/`、`solution/`、`explainer/`）都需要一个 `readme.md`，该文件：

* **不能为空**（必须包含实际内容，甚至只有一行标题也可以）
* 不能有断链

创建存根时，创建一个带有标题和描述的最小 readme：

```md
# Exercise Title

Description here
```

如果子文件夹包含代码，它还需要一个 `main.ts`（>1 行）。但对于存根来说，仅 readme 的练习是可以的。

## 工作流程

1. **解析计划** - 提取章节名称、练习名称和变体类型
2. **创建目录** - 对每个路径使用 `mkdir -p`
3. **创建存根 readme** - 每个变体文件夹一个带标题的 `readme.md`
4. **运行 lint** - 使用 `pnpm ai-hero-cli internal lint` 进行验证
5. **修复任何错误** - 迭代直到 lint 通过

## Lint 规则摘要

Linter（`pnpm ai-hero-cli internal lint`）检查：

* 每个练习都有子文件夹（`problem/`、`solution/`、`explainer/`）
* `problem/`、`explainer/` 或 `explainer.1/` 中至少存在一个
* 主子文件夹中存在且非空的 `readme.md`
* 没有 `.gitkeep` 文件
* 没有 `speaker-notes.md` 文件
* readme 中没有断链
* readme 中没有 `pnpm run exercise` 命令
* 除非是仅 readme 的练习，否则每个子文件夹都需要 `main.ts`

## 移动/重命名练习

重新编号或移动练习时：

1. 使用 `git mv`（而不是 `mv`）重命名目录 - 保留 git 历史记录
2. 更新数字前缀以保持顺序
3. 移动后重新运行 lint

Example:

```bash
git mv exercises/01-retrieval/01.03-embeddings exercises/01-retrieval/01.04-embeddings
```

## Example: stubbing from a plan

给定一个如下计划：

```
Section 05: Memory Skill Building
- 05.01 Introduction to Memory
- 05.02 Short-term Memory (explainer + problem + solution)
- 05.03 Long-term Memory
```

Create:

```bash
mkdir -p exercises/05-memory-skill-building/05.01-introduction-to-memory/explainer
mkdir -p exercises/05-memory-skill-building/05.02-short-term-memory/{explainer,problem,solution}
mkdir -p exercises/05-memory-skill-building/05.03-long-term-memory/explainer
```

然后创建 readme 存根：

```
exercises/05-memory-skill-building/05.01-introduction-to-memory/explainer/readme.md -> "# Introduction to Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/explainer/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/problem/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.02-short-term-memory/solution/readme.md -> "# Short-term Memory"
exercises/05-memory-skill-building/05.03-long-term-memory/explainer/readme.md -> "# Long-term Memory"
```

---
name: setup-ts-deep-modules
description: 将 dependency-cruiser 接入 TypeScript
  仓库，使每个包都成为一个深模块，实现细节隐藏在子文件夹中，且只能通过其入口点文件访问。用户触发。
disable-model-invocation: true

---

# # 设置 TS 深模块

使本仓库中的每个包都成为一个**深模块**：小接口背后隐藏大量行为。一个包的公共表面是其**入口点**（包根目录下的文件），子文件夹中的所有内容都是隐藏的。本技能会安装 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) 以及强制入口点为唯一进入方式的规则，然后证明这些规则确实有效。

关于词汇（深模块、接口、接缝、深度），请调用“codebase-design”技能工具并在全程使用其语言。

## ## 强制形成的结构形态

```
src/packages/
  <name>/
    index.ts        ← an entry point (public). Import this from outside.
    client.ts       ← another entry point. Packages may expose SEVERAL.
    lib/            ← implementation: hidden from outside, free to import each other.
    tests/          ← co-located tests + fixtures (a subfolder, so private).
```

公共表面是包的**根文件**，而不是某个指定的 `index.ts`。按照惯例，实现在 `lib/` 中，测试在 `tests/` 中，给每个包相同的两个文件夹结构。不过规则本身是通用的：*任何*子文件夹中的*任何*内容都是私有的，因此你永远不会扩展配置来添加文件夹。

四条规则，全部为 `error`：

1. **入口点边界**：包外的代码（应用代码或另一个包）只能导入该包的入口点（其根文件），绝不能导入其子文件夹中的任何内容。
2. **包内自由**：一个包自己的文件可以自由互相导入。
3. **通过入口点测试**：`<pkg>/tests/` 下的文件可以导入任何包的入口点及其自己的 `tests/` 测试数据，但绝不能导入任何包的子文件夹内部（甚至不能导入自己的）。跨包的集成测试是可以的；深导入是不行的。
4. **无循环**：没有依赖循环。

**是入口点，不是桶文件。** 因为公共表面是*每一个*根文件，一个包可以暴露几个小的入口点（`index.ts`、`client.ts`、`server.ts`），而不是将所有内容通过一个巨大的 `index.ts` 汇聚。重新导出整个子树的桶文件是不鼓励的；保持入口点小，并在子文件夹中隐藏实现。

分层（哪些包可以依赖哪些包）是一个*不同*的关注点，在配置中留作注释存根，由本仓库自行填写。

## 步骤

### 1. 检测环境

* **包管理器**：`pnpm-lock.yaml` → pnpm，`yarn.lock` → yarn，`bun.lockb` → bun，否则为 npm。用它执行下方的每个命令（`pnpm`/`yarn`/`npm run`/`bunx`）。
* **包根目录**：如果存在 `src/` 则使用 `src/packages`，否则使用 `packages`。如果仓库已有不同的明显约定，请与用户确认选择。
* **现有配置**：检查是否存在 `.dependency-cruiser.*` 文件。如果存在，**不要**覆盖它：将四条规则和选项合并进去，并告知用户你添加了什么。

**完成条件：** 包管理器、包根目录以及现有配置的状态都已明确。

### 2. 安装 dependency-cruiser

使用检测到的包管理器将 `dependency-cruiser` 作为 devDependency 安装。

**完成条件：** `dependency-cruiser` 已位于 `devDependencies` 中。

### 3. 编写配置

将 [`dependency-cruiser.config.cjs`](./dependency-cruiser.config.cjs) 复制到仓库根目录，命名为 `.dependency-cruiser.cjs`。将 `PACKAGES_ROOT` 设置为步骤 1 中检测到的根目录。这些规则基于路径深度且与扩展名无关，因此无需调整其他任何内容。

**完成条件：** `.dependency-cruiser.cjs` 存在且 `PACKAGES_ROOT` 正确，并且四条禁止规则均已就位。

### 4. 将其接入检查流程

* 添加一个 `lint:boundaries` 脚本：`depcruise <packages-root>`（或 `depcruise src`）。
* 将其整合到仓库的总检查命令中，即那个已经运行类型检查的命令（例如 `check`/`ci`/`validate` 脚本）。**不要**修改 `tsconfig` 或添加路径别名。
* 如果没有总检查脚本，则添加 `lint:boundaries`，并告知用户将其纳入 CI。

**完成条件：** `lint:boundaries` 存在，并且与类型检查作为同一条命令的一部分运行。

### 5. 搭建示例包

创建一个已提交的 `<packages-root>/example/`，作为可复制的模板：

* `index.ts` 是一个入口点。导出一个委托给内部文件的函数（这样包在视觉上是*深*的，而不是透传）。
* `lib/impl.ts`：位于**子文件夹**中的内部文件，由 `index.ts` 导入，从外部无法访问。
* `tests/example.test.ts` 仅导入 `../index`（一个入口点）并断言公共函数。

告诉用户这是一个可供复制或删除的入门模板。

**完成条件：** 示例包已存在，通过根入口点暴露其行为，并将 `impl` 隐藏在子文件夹中。

### 6. 证明规则确实能拦截违规

这是整个技能的完成标准：一个在违规时不会失败的配置毫无价值。

1. 运行 `lint:boundaries`。它在干净的示例上必须**通过**。
2. 临时向 `tests/example.test.ts` 添加深导入（例如 `import { thing } from "../lib/impl"`）。再次运行 `lint:boundaries`；它必须因 `tests-through-entrypoints` **失败**。
3. 恢复深导入。再运行一次，它必须**通过**。

**完成条件：** 你已经观察到通过，然后深导入失败，再通过。如果步骤 2 没有失败，说明规则连接不正确，请在完成前修复。

### 7. 记录约定

在包文件夹中编写一个 `README.md`（`<packages-root>/README.md`，位于其管理的包旁边），内容包括：`src/packages/<name>/` 的布局（根目录下的入口点，`lib/` 用于实现，`tests/` 用于测试），“仅通过包的入口点（其根文件）导入”，以及如何运行 `lint:boundaries`。**明确不鼓励桶文件**：暴露几个小的入口点，而不是通过一个索引重新导出整个子树。保持为复制粘贴片段加上每条规则的一段话。

然后从仓库的代理指令文件中添加一个**上下文指针**（如果存在则为 `CLAUDE.md`，否则为 `AGENTS.md`，如果两者都不存在则创建 `AGENTS.md`）。一行就够了，例如 `Packages are deep modules: see [src/packages/README.md](./src/packages/README.md) before adding or importing one.` 这能让代理发现边界规则而不是撞到它上。

**完成条件：** `<packages-root>/README.md` 存在并明确不鼓励桶文件，且仓库的 `CLAUDE.md`/`AGENTS.md` 中有指向它的链接。

## 注意

* 配置的 `$1` 反向引用（dependency-cruiser 的组匹配）是让包能够访问其内部内容而外部无法访问的原因。不要将它们展平为单独的包规则。
* 公共与私有由**深度**决定：包的根文件是入口点；子文件夹中的任何内容都是私有的。常规子文件夹是 `lib/`（实现）和 `tests/`，但规则没有硬编码它们：任何子文件夹都是私有的，所以新文件夹永远不需要配置更改。添加入口点只是添加一个根文件（没有桶文件）。
* 包是**扁平**的：根目录下只有一层直接子项。包的内部可以随心所欲地多层嵌套；但一个包不能包含另一个包。
* 使用 `.cjs`（而不是 `.js`），这样即使在 `"type": "module"` 的仓库中，配置里的 `module.exports` 也能正常工作。

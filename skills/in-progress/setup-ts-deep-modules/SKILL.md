---
name: setup-ts-deep-modules
description: 将 dependency-cruiser 接入到 TypeScript
  仓库中，使每个包都成为一个深度模块——实现隐藏在子文件夹中，仅可通过其入口点文件访问。用户调用。
disable-model-invocation: true

---

# 设置 TS 深度模块

使此仓库中的每个包都成为一个**深度模块**：小接口背后的许多行为。包的公开面是其**入口点**——位于包根目录的文件——其子文件夹中的所有内容都是隐藏的。该技能安装了 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) 以及使入口点成为唯一访问方式的规则，然后证明这些规则确实生效。

关于词汇（深度模块、接口、接缝、深度），请运行 `/codebase-design` 技能——在整个过程中使用其语言。

## 此规则强制要求的结构

```
src/packages/
  <name>/
    index.ts        ← an entry point (public). Import this from outside.
    client.ts       ← another entry point. Packages may expose SEVERAL.
    lib/            ← implementation: hidden from outside, free to import each other.
    tests/          ← co-located tests + fixtures (a subfolder, so private).
```

公开面是包的**根文件**——而不是一个指定的 `index.ts`。按照惯例，实现放在 `lib/` 中，测试放在 `tests/` 中，这给每个包赋予了相同的两个文件夹结构。不过，规则本身是通用的：*任何*子文件夹中的*任何*内容都是私有的，因此你永远不需要扩展配置来添加文件夹。

四条规则，全部为 `error`：

1. **入口点边界**——包外部（应用代码或另一个包）的代码只能导入该包的入口点（其根文件），而不能导入其子文件夹中的任何内容。
2. **包内自由**——包自己的文件可以自由相互导入。
3. **通过入口点测试**——`<pkg>/tests/` 下的文件可以导入任何包的入口点及其自己的 `tests/` 测试夹具，但不能导入任何包的子文件夹内部（即使是它们自己的）。跨包的集成测试是可以的；深度导入是不行的。
4. **无循环**——无依赖循环。

**入口点，而不是桶文件。** 因为公开面是*所有*根文件，包可以暴露几个小的入口点（`index.ts`、`client.ts`、`server.ts`），而不是通过一个巨大的 `index.ts` 将所有内容汇集起来。重导出整个子树的桶文件是不被鼓励的——保持入口点小，并将实现隐藏在子文件夹中。

分层（哪些包可以依赖哪些）是*不同*的关注点，在配置中留作注释的存根供此仓库填写。

## 步骤

### 1. 检测环境

* **包管理器** — `pnpm-lock.yaml` → pnpm，`yarn.lock` → yarn，`bun.lockb` → bun，否则使用 npm。将其用于下面的每个命令（`pnpm`/`yarn`/`npm run`/`bunx`）。
* **包根目录** — 如果存在 `src/` 则使用 `src/packages`，否则使用 `packages`。如果仓库已经有其他明显的约定，请与用户确认选择。
* **现有配置** — 检查是否存在 `.dependency-cruiser.*` 文件。如果存在，**不要**覆盖它：将四条规则和选项合并进去，并告诉用户你添加了什么。

**完成条件：** 包管理器、包根目录和现有配置状态都已知晓。

### 2. 安装 dependency-cruiser

使用检测到的包管理器将 `dependency-cruiser` 安装为开发依赖。

**完成条件：** `dependency-cruiser` 在 `devDependencies` 中。

### 3. 编写配置

将 [`dependency-cruiser.config.cjs`](./dependency-cruiser.config.cjs) 复制到仓库根目录作为 `.dependency-cruiser.cjs`。将 `PACKAGES_ROOT` 设置为步骤 1 中检测到的根目录。规则基于路径深度且与扩展名无关，因此无需调整其他内容。

**完成条件：** `.dependency-cruiser.cjs` 存在且包含正确的 `PACKAGES_ROOT`，并且四条禁止规则已存在。

### 4. 将其接入检查

* 添加一个 `lint:boundaries` 脚本：`depcruise <packages-root>`（或 `depcruise src`）。
* 将其整合到仓库的统一检查命令中——即已经运行类型检查的那个（例如 `check` / `ci` / `validate` 脚本）。**不要**触碰 `tsconfig` 或添加路径别名。
* 如果没有统一脚本，添加 `lint:boundaries` 并告诉用户将其包含在 CI 中。

**完成条件：** `lint:boundaries` 存在并作为类型检查同一命令的一部分运行。

### 5. 搭建示例包

创建一个已提交的 `<packages-root>/example/` 作为“复制我”模板：

* `index.ts` — 一个入口点。导出一个委托给内部文件的函数（这样包在视觉上是*深度*的，而不是透传）。
* `lib/impl.ts` — **子文件夹**中的内部文件，被 `index.ts` 导入，无法从外部访问。
* `tests/example.test.ts` — **只导入** `../index`（一个入口点），并针对公共函数进行断言。

告诉用户这是一个可以复制或删除的起始模板。

**完成条件：** 示例包存在，通过根入口点暴露其行为，并将 `impl` 隐藏在子文件夹中。

### 6. 验证规则生效

This is the completion criterion for the whole skill — a config that doesn't fail on a violation is worthless.

1. 运行 `lint:boundaries`。它必须在干净的示例上**通过**。
2. 临时向 `tests/example.test.ts` 添加一个深度导入（例如 `import { thing } from "../lib/impl"`）。再次运行 `lint:boundaries` — 它必须**失败**并显示 `tests-through-entrypoints`。
3. 还原深度导入。再次运行一次 — 它必须**通过**。

**完成条件：** 你已经观察到一次通过，然后深度导入失败，再通过一次。如果步骤 2 没有失败，说明规则没有正确配置——在完成前修复。

### 7. 记录约定

在包文件夹中编写 `README.md`（`<packages-root>/README.md`）——紧邻它所管辖的包——涵盖：`src/packages/<name>/` 布局（根目录有入口点，`lib/` 用于实现，`tests/` 用于测试），“仅通过包的入口点（其根文件）导入”，以及如何运行 `lint:boundaries`。**明确不鼓励桶文件**——展示几个小的入口点，而不是通过一个索引重导出整个子树。保持为“复制我”代码片段加上每条规则的一段话。

然后从仓库的代理指令文件中添加一个**上下文指针**——如果存在则为 `CLAUDE.md`，否则为 `AGENTS.md`（如果两者都不存在则创建 `AGENTS.md`）。一行就够了，例如 `Packages are deep modules — see [src/packages/README.md](./src/packages/README.md) before adding or importing one.` 这就是让代理发现边界规则而不是踩到它的原因。

**完成条件：** `<packages-root>/README.md` 存在且不鼓励桶文件，且仓库的 `CLAUDE.md`/`AGENTS.md` 链接到它。

## 备注

* 配置中的 `$1` 后引用（dependency-cruiser 的组匹配）是让包能够访问其内部而外部人员无法访问的方式——不要将其扁平化为单独的每个包规则。
* 公开与私有的区别由**深度**决定：包的根文件是入口点；子文件夹中的任何内容都是私有的。惯例的子文件夹是 `lib/`（实现）和 `tests/`，但规则没有硬编码它们——任何子文件夹都是私有的，因此新文件夹永远不需要配置更改。添加入口点只是添加根文件——没有桶文件。
* 包是**扁平的**：根目录下只有一层直接子包。包的内部可以嵌套任意深度；一个包不能包含另一个包。
* 使用 `.cjs`（而不是 `.js`），以便配置的 `module.exports` 即使在 `"type": "module"` 仓库中也能工作。

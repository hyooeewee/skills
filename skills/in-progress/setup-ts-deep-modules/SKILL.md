---
name: setup-ts-deep-modules
description: 将 dependency-cruiser 接入 TypeScript
  仓库，使每个包都成为深模块——实现隐藏在子文件夹中，只能通过其入口点文件访问。由用户调用。
disable-model-invocation: true

---

# # 设置 TS 深模块

让此仓库中的每个包都成为**深模块**：以一个小接口承载大量行为。包的公共表面是其**入口点**——包根目录下的文件——而子文件夹中的一切都被隐藏。此技能安装 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) 以及使入口点成为唯一进入方式的规则，然后验证这些规则确实生效。

关于词汇（深模块、接口、接缝、深度），请运行 `/codebase-design` 技能——并全程使用其术语。

## ## 强制形成的结构形态

```
src/packages/
  <name>/
    index.ts        ← an entry point (public). Import this from outside.
    client.ts       ← another entry point. Packages may expose SEVERAL.
    lib/            ← implementation: hidden from outside, free to import each other.
    tests/          ← co-located tests + fixtures (a subfolder, so private).
```

公共表面是包的**根文件**——而不是某个指定的 `index.ts`。按照惯例，实现位于 `lib/`，测试位于 `tests/`，使每个包都具有相同的两文件夹形态。不过，规则本身是通用的：任何子文件夹中的*任何*内容都是私有的，因此你永远无需为了添加文件夹而扩展配置。

四条规则，全部为 `error`：

1. **入口点边界**——包外部的代码（应用代码或其他包）只能导入该包的入口点（其根文件），绝不能导入其子文件夹中的任何内容。
2. **包内自由**——包自身的文件可以自由地相互导入。
3. **测试必须通过入口点**——`<pkg>/tests/` 下的文件可以导入任何包的入口点以及它们自己的 `tests/` 夹具，但绝不能导入任何包的子文件夹内部内容（即使是自己的也不行）。跨包的集成测试没有问题；深层导入则不允许。
4. **禁止循环**——不允许存在依赖循环。

**入口点，而非桶文件。** 因为公共表面是*每一个*根文件，包可以暴露多个小型入口点（`index.ts`、`client.ts`、`server.ts`），而不是把所有内容都汇聚到一个巨大的 `index.ts`。不鼓励重新导出整个子树的桶文件——保持入口点小而精，将实现隐藏在子文件夹中。

分层（哪些包可以依赖哪些包）是一个*不同*的关注点，在配置中留作注释存根，由本仓库自行填写。

## 步骤

### 1. 检测环境

* **包管理器**——`pnpm-lock.yaml` → pnpm，`yarn.lock` → yarn，`bun.lockb` → bun，否则为 npm。下面的每条命令都使用它（`pnpm`/`yarn`/`npm run`/`bunx`）。
* **包根目录**——如果存在 `src/`，则使用 `src/packages`，否则使用 `packages`。如果仓库已有其他明显的约定，请与用户确认该选择。
* **现有配置**——检查是否存在 `.dependency-cruiser.*` 文件。如果存在，**不要**覆盖它：将四条规则和选项合并进去，并告知用户你添加的内容。

**完成条件：** 包管理器、包根目录以及现有配置的状态都已明确。

### 2. 安装 dependency-cruiser

使用检测到的包管理器将 `dependency-cruiser` 作为 devDependency 安装。

**完成条件：** `dependency-cruiser` 已位于 `devDependencies` 中。

### 3. 编写配置

将 [`dependency-cruiser.config.cjs`](./dependency-cruiser.config.cjs) 复制到仓库根目录，命名为 `.dependency-cruiser.cjs`。将 `PACKAGES_ROOT` 设置为步骤 1 中检测到的根目录。这些规则基于路径深度且与扩展名无关，因此无需调整其他任何内容。

**完成条件：** `.dependency-cruiser.cjs` 存在且 `PACKAGES_ROOT` 正确，并且四条禁止规则均已就位。

### 4. 将其接入检查流程

* 添加一个 `lint:boundaries` 脚本：`depcruise <packages-root>`（或 `depcruise src`）。
* 将其并入仓库的总检查命令——即已经运行类型检查的那个命令（例如 `check` / `ci` / `validate` 脚本）。**不要**改动 `tsconfig`，也不要添加路径别名。
* 如果没有总检查脚本，则添加 `lint:boundaries`，并告知用户将其纳入 CI。

**完成条件：** `lint:boundaries` 存在，并且与类型检查作为同一条命令的一部分运行。

### 5. 搭建示例包

创建一个已提交的 `<packages-root>/example/`，作为可复制的模板：

* `index.ts` —— 一个入口点。导出一个委托给内部文件的函数（这样包从外观上就是*深层*的，而不是透传）。
* `lib/impl.ts` —— 位于**子文件夹**中的内部文件，由 `index.ts` 导入，外部无法触达。
* `tests/example.test.ts` —— **仅**导入 `../index`（一个入口点），并对公共函数进行断言。

告诉用户这是一个可供复制或删除的入门模板。

**完成条件：** 示例包已存在，通过根入口点暴露其行为，并将 `impl` 隐藏在子文件夹中。

### 6. 证明规则确实能拦截违规

这是整个技能的完成标准——一个在违规时不报错的配置毫无价值。

1. 运行 `lint:boundaries`。它在干净的示例上必须**通过**。
2. 临时向 `tests/example.test.ts` 添加一个深层导入（例如 `import { thing } from "../lib/impl"`）。再次运行 `lint:boundaries`——它必须因 `tests-through-entrypoints` 而**失败**。
3. 还原该深层导入。再运行一次——必须**通过**。

**完成条件：** 你已观察到通过、随后在深层导入上失败、然后又通过。如果步骤 2 没有失败，说明规则并未正确接入——在完成之前修复。

### 7. 记录约定

在包文件夹内（`<packages-root>/README.md`）编写一个 `README.md`——就在它所管辖的包旁边——涵盖：`src/packages/<name>/` 布局（入口点在根目录，`lib/` 存放实现，`tests/` 存放测试）、“只能通过包的入口点（其根文件）导入”，以及如何运行 `lint:boundaries`。明确**不鼓励桶文件**——暴露多个小型入口点，而不是通过一个 index 重新导出整个子树。内容保持为可复制片段加上四条规则，每条规则一段即可。

然后，在仓库的代理指令文件中添加一个指向它的**上下文指针**——如果存在 `CLAUDE.md` 则使用它，否则使用 `AGENTS.md`（如果两者都不存在，则创建 `AGENTS.md`）。一行就足够了，例如 `Packages are deep modules — see [src/packages/README.md](./src/packages/README.md) before adding or importing one.` 这正是让代理发现边界规则而不是被它绊倒的原因。

**完成条件：** `<packages-root>/README.md` 存在并明确不鼓励桶文件，且仓库的 `CLAUDE.md`/`AGENTS.md` 中有指向它的链接。

## 注意

* 配置中的 `$1` 反向引用（dependency-cruiser 的分组匹配）正是让包能够访问自身内部而外部无法访问的关键——不要将它们拆解为独立的逐包规则。
* 公共与私有的判定由**深度**决定：包的根文件是入口点；子文件夹中的任何内容都是私有的。约定俗成的子文件夹是 `lib/`（实现）和 `tests/`，但规则并没有将它们写死——任何子文件夹都是私有的，因此新增文件夹永远不需要修改配置。添加入口点只需添加一个根文件——无需桶文件。
* 包是**扁平**的：根目录下只有一层直接子项。包的内部可以随心所欲地多层嵌套；但一个包不能包含另一个包。
* 使用 `.cjs`（而不是 `.js`），这样即使在 `"type": "module"` 的仓库中，配置里的 `module.exports` 也能正常工作。

---
name: setup-pre-commit
description: 在当前仓库中设置 Husky pre-commit hooks（配合
  lint-staged/Prettier）、类型检查和测试。当用户想要添加 pre-commit hooks、设置 Husky、配置 lint-staged
  或添加提交时的格式化/类型检查/测试时使用。

---

# 设置 Pre-Commit Hooks

## What This Sets Up

* **Husky** pre-commit hook
* **lint-staged** 对所有暂存文件运行 Prettier
* **Prettier** 配置（如果缺失）
* pre-commit hook 中的 **typecheck** 和 **test** 脚本

## 步骤

### 1. 检测包管理器

检查是否存在 `package-lock.json` (npm)、`pnpm-lock.yaml` (pnpm)、`yarn.lock` (yarn) 或 `bun.lockb` (bun)。使用存在的那个。如果不清楚则默认使用 npm。

### 2. 安装依赖

作为开发依赖安装：

```
husky lint-staged prettier
```

### 初始化 Husky

```bash
npx husky init
```

这将创建 `.husky/` 目录，并在 package.json 中添加 `prepare: "husky"`。

### 创建 `.husky/pre-commit`

编写此文件（Husky v9+ 不需要 shebang）：

```
npx lint-staged
npm run typecheck
npm run test
```

**调整**：将 `npm` 替换为检测到的包管理器。如果 package.json 中没有 `typecheck` 或 `test` 脚本，则省略这些行并告知用户。

### 创建 `.lintstagedrc`

```json
{
  "*": "prettier --ignore-unknown --write"
}
```

### 创建 `.prettierrc`（如果缺失）

仅在没有 Prettier 配置时创建。使用这些默认值：

```json
{
  "useTabs": false,
  "tabWidth": 2,
  "printWidth": 80,
  "singleQuote": false,
  "trailingComma": "es5",
  "semi": true,
  "arrowParens": "always"
}
```

### 7. 验证

* [ ] `.husky/pre-commit` 存在且可执行
* [ ] `.lintstagedrc` 存在
* [ ] package.json 中的 `prepare` 脚本是 `"husky"`
* [ ] `prettier` 配置存在
* [ ] 运行 `npx lint-staged` 以验证其工作正常

### 8. 提交

暂存所有已更改/创建的文件，并使用消息提交：`Add pre-commit hooks (husky + lint-staged + prettier)`

这将运行新的 pre-commit hooks——这是验证一切正常的一个很好的冒烟测试。

## 备注

* Husky v9+ 不需要 hook 文件中的 shebang
* `` `prettier --ignore-unknown` 会跳过 Prettier 无法解析的文件（图片等） ``
* pre-commit 首先运行 lint-staged（快速，仅限暂存），然后进行完整的类型检查和测试

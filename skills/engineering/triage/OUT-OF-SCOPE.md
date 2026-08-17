# 范围外知识库

仓库中的 `.out-of-scope/` 目录用于持久保存被拒绝的功能请求记录。它有两个用途：

1. **组织记忆**——记录功能被拒绝的原因，以便在 issue 关闭后，其理由不会丢失
2. **去重**——当新 issue 与先前的拒绝相匹配时，技能可以呈现先前的决定，而无需重新讨论

## 目录结构

```
.out-of-scope/
├── dark-mode.md
├── plugin-system.md
└── graphql-api.md
```

每个**概念**一个文件，而非每个 issue 一个文件。多个请求相同内容的 issue 归并到同一文件下。

## 文件格式

文件应以轻松、可读的风格编写——更像一份简短的设计文档，而非数据库条目。使用段落、代码示例和实例，使推理对首次遇到的人清晰且有用。

````markdown
# Dark Mode

This project does not support dark mode or user-facing theming.

## Why this is out of scope

The rendering pipeline assumes a single color palette defined in
`ThemeConfig`. Supporting multiple themes would require:

- A theme context provider wrapping the entire component tree
- Per-component theme-aware style resolution
- A persistence layer for user theme preferences

This is a significant architectural change that doesn't align with the
project's focus on content authoring. Theming is a concern for downstream
consumers who embed or redistribute the output.

```ts
// The current ThemeConfig interface is not designed for runtime switching:
interface ThemeConfig {
  colors: ColorPalette; // single palette, resolved at build time
  fonts: FontStack;
}
````

## 先前的请求

* \#42 — “添加深色模式支持”
* \#87 — “无障碍夜间主题”
* \#134 — “深色主题选项”

```

### Naming the file

Use a short, descriptive kebab-case name for the concept: `dark-mode.md`, `plugin-system.md`, `graphql-api.md`. The name should be recognizable enough that someone browsing the directory understands what was rejected without opening the file.

### Writing the reason

The reason should be substantive — not "we don't want this" but why. Good reasons reference:

- Project scope or philosophy ("This project focuses on X; theming is a downstream concern")
- Technical constraints ("Supporting this would require Y, which conflicts with our Z architecture")
- Strategic decisions ("We chose to use A instead of B because...")

The reason should be durable. Avoid referencing temporary circumstances ("we're too busy right now") — those aren't real rejections, they're deferrals.

## When to check `.out-of-scope/`

During triage (Step 1: Gather context), read all files in `.out-of-scope/`. When evaluating a new issue:

- Check if the request matches an existing out-of-scope concept
- Matching is by concept similarity, not keyword — "night theme" matches `dark-mode.md`
- If there's a match, surface it to the maintainer: "This is similar to `.out-of-scope/dark-mode.md` — we rejected this before because [reason]. Do you still feel the same way?"

The maintainer may:

- **Confirm** — the new issue gets added to the existing file's "Prior requests" list, then closed
- **Reconsider** — the out-of-scope file gets deleted or updated, and the issue proceeds through normal triage
- **Disagree** — the issues are related but distinct, proceed with normal triage

## When to write to `.out-of-scope/`

Only when an **enhancement** (not a bug) is *rejected* as `wontfix`. This applies to enhancement PRs exactly as it does to issues — a rejected PR is recorded here so the same request doesn't return as fresh code.

Do **not** write here when something is closed as `wontfix` because it's **already implemented**. That's a built feature, not a rejected one; recording it would poison the dedup checks with false rejections. Instead, the closing comment points to where the feature already lives.

The flow:

1. Maintainer decides a feature request is out of scope
2. Check if a matching `.out-of-scope/` file already exists
3. If yes: append the new issue to the "Prior requests" list
4. If no: create a new file with the concept name, decision, reason, and first prior request
5. Post a comment on the issue explaining the decision and mentioning the `.out-of-scope/` file
6. Close the issue with the `wontfix` label

## Updating or removing out-of-scope files

If the maintainer changes their mind about a previously rejected concept:

- Delete the `.out-of-scope/` file
- The skill does not need to reopen old issues — they're historical records
- The new issue that triggered the reconsideration proceeds through normal triage
```

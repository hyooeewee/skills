---
name: git-guardrails-claude-code
description: 设置 Claude Code 挂钩以在执行危险 Git 命令（push、reset --hard、clean、branch -D
  等）之前阻止它们。当用户想要防止破坏性 Git 操作、添加 Git 安全挂钩，或在 Claude Code 中阻止 git push/reset 时使用。

---

# 设置 Git 限制措施

设置一个 PreToolUse 挂钩，在 Claude 执行危险 Git 命令之前进行拦截和阻止。

## 将被阻止的内容

* 所有变体，包括 `--force`
* `git reset --hard`
* `git clean -f` / `git clean -fd`
* `git branch -D`
* `git checkout .` / `git restore .`

当被阻止时，Claude 会看到一条消息，告诉它没有权限访问这些命令。

## 步骤

### 1. 询问范围

询问用户：是为**仅此项目** (`.claude/settings.json`) 还是**所有项目** (`~/.claude/settings.json`) 安装？

### 2. 复制挂钩脚本

捆绑的脚本位于：[scripts/block-dangerous-git.sh](scripts/block-dangerous-git.sh)

根据范围将其复制到目标位置：

* **项目**: `.claude/hooks/block-dangerous-git.sh`
* **全局**: `~/.claude/hooks/block-dangerous-git.sh`

使用 `chmod +x` 使其可执行。

### 3. 将挂钩添加到设置中

添加到相应的设置文件中：

**项目** (`.claude/settings.json`)：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

**全局** (`~/.claude/settings.json`)：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

如果设置文件已存在，请将挂钩合并到现有的 `hooks.PreToolUse` 数组中 — 不要覆盖其他设置。

### 4. 询问自定义选项

询问用户是否要从被阻止列表中添加或删除任何模式。相应地编辑复制的脚本。

### 5. 验证

运行快速测试：

```bash
echo '{"tool_input":{"command":"git push origin main"}}' | <path-to-script>
```

应退出代码 2 并向 stderr 打印 BLOCKED 消息。

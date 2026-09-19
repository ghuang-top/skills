# Skills

## Git 暂存区代码审查 Skill

帮我写一个「审查 Staged 代码改动是否正确」的 Skill，并告诉我你需要了解哪些信息。

### 需求

- **范围**：审查 Git 暂存区（staged）改动，从正确性/Bug、代码规范、性能等维度判断修改是否正确。
- **输出格式**：简体中文，按 🔴/🟡/🟢 分级反馈（文件:行号 + 问题 + 原因）；🔴/🟡 须附修复代码片段，仅在对话中输出。
- **权限边界**：只读，不调用 Edit/Write 改动任何文件。禁止以下几类命令：
  - 改动工作区 / 暂存区：`git add`、`git rm`、`git checkout -- <file>`、`git restore`、`git stash`、`git clean`
  - 改动提交历史：`git commit`（含 `--amend`）、`git reset`、`git revert`、`git rebase`
  - 改动分支 / 标签 / 远程：`git branch -D`、`git tag -d`、`git push`（含 `--force`）、`git remote set-url`
  - 写盘副作用：任何命令的 `--output=<file>` 等写盘选项，以及 `>`、`>>`、`tee` 输出重定向（`git diff --staged --output=foo.patch` 也是写文件）
- **frontmatter 写法**：用 `allowed-tools` 预授权执行工作流所需的只读工具；Bash 应使用命令级条目，如 `Bash(git status:*)`、`Bash(git diff --staged:*)`，避免预授权裸 `Bash`。用 `disallowed-tools: Edit, Write, NotebookEdit` 在 Skill 生效期间移除文件写入工具。注意：`allowed-tools` 不是白名单，不会禁止未列出的工具；`disallowed-tools` 的限制通常只在调用 Skill 的当前轮次生效。正文中仍须保留完整的只读行为约定；若需要跨轮或不可绕过的限制，应配合客户端权限 deny 规则或 `PreToolUse` hook。

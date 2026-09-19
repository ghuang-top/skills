# Skills

## 命名规范审查 Skill

帮我写一个「检查代码命名是否规范」的 Skill，并告诉我你需要了解哪些信息。

### 需求

- **范围**：检查 Git 改动（staged + unstaged + 未跟踪新文件）中新增的变量名、函数/方法名、入参名、类/类型名、常量名等命名是否规范。
- **判断标准**：优先与项目现有代码风格保持一致；无先例时按语言惯例（如 JS/TS 用 camelCase、Python 用 snake_case、类名用 PascalCase）。
- **输出格式**：只在对话里按 🔴/🟡/🟢 分级列出问题（文件:行号 + 问题 + 建议改成什么名字），不写文件、不给出完整重写。
- **权限边界**：只读，不调用 Edit/Write 改动任何文件。禁止以下几类命令：
  - 改动工作区 / 暂存区：`git add`、`git rm`、`git checkout -- <file>`、`git restore`、`git stash`、`git clean`
  - 改动提交历史：`git commit`（含 `--amend`）、`git reset`、`git revert`、`git rebase`
  - 改动分支 / 标签 / 远程：`git branch -D`、`git tag -d`、`git push`（含 `--force`）、`git remote set-url`
  - 写盘副作用：任何命令的 `--output=<file>` 等写盘选项，以及 `>`、`>>`、`tee` 输出重定向（`git diff --output=foo.patch` 也是写文件）
- **frontmatter 写法**：用 `allowed-tools` 预授权执行工作流所需的只读工具；Bash 应使用命令级条目，如 `Bash(git status:*)`、`Bash(git diff:*)`，避免预授权裸 `Bash`。用 `disallowed-tools: Edit, Write, NotebookEdit` 在 Skill 生效期间移除文件写入工具。注意：`allowed-tools` 不是白名单，不会禁止未列出的工具；`disallowed-tools` 的限制通常只在调用 Skill 的当前轮次生效。正文中仍须保留完整的只读行为约定；若需要跨轮或不可绕过的限制，应配合客户端权限 deny 规则或 `PreToolUse` hook。

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
- **frontmatter 写法**：用 `allowed-tools` 声明只读白名单（不要用 `tools:`，Skill 规范里没有这个字段）；**Bash 必须写成命令级条目**（如 `Bash(git status:*)`、`Bash(git diff --staged:*)`），不要只写裸 `Bash` —— 裸 `Bash` 等于放开任意 shell 命令。可再加 `disallowed-tools: Edit, Write, NotebookEdit` 作「不生效也无害」的兜底；同时**必须在正文里重写一遍同样的只读清单** —— 这两个字段只是尽力而为的提示，不是硬边界，正文那段才是真正生效的约束。

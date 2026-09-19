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
- **frontmatter 写法**：用 `allowed-tools` 声明只读白名单（不要用 `tools:`，Skill 规范里没有这个字段）；**Bash 必须写成命令级条目**（如 `Bash(git status:*)`、`Bash(git diff:*)`），不要只写裸 `Bash` —— 裸 `Bash` 等于放开任意 shell 命令。可再加 `disallowed-tools: Edit, Write, NotebookEdit` 作「不生效也无害」的兜底；同时**必须在正文里重写一遍同样的只读清单** —— 这两个字段只是尽力而为的提示，不是硬边界，正文那段才是真正生效的约束。

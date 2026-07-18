# Skills

## Git 提交信息生成 Skill

帮我写一个「根据 Staged 改动生成提交信息」的 Skill，并告诉我你需要了解哪些信息。

### 需求

- **范围**：读取 Git 暂存区（staged）改动，按 Conventional Commits 规范生成提交信息。
- **输出格式**：同时输出英文版与对应中文版（各含 `<type>: <description>` 首行 + 正文 body）；type 用英文（feat / fix / perf / refactor / docs / style / chore / build / test / ci）。
- **权限边界**：只读，禁止 commit、禁止 push（含强推）、禁止 reset/restore/revert、禁止 rebase、禁止改动远程配置。

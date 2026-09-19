# Skills

## 每日 AI 新闻简报 Skill

帮我写一个「每日 AI 新闻简报」的 Skill，并告诉我你需要了解哪些信息。

### 需求

- **范围**：整理过去 2-3 天 AI 重要新闻，聚焦「大模型/产品发布」与「前沿研究/论文」。
- **条数**：5-8 条，按重要性排序；每条含事实摘要 + 影响分析，附 1 个原文链接；去重、核实、过滤旧闻。
- **输出格式**：中文 Markdown，顶部速览表 + 分块卡片，用 ⭐⭐⭐ / ⭐⭐ / ⭐ 标注重要程度；仅在对话中展示。
- **权限边界**：只读整理，不写文件、不执行任何 Shell 命令、不发送到外部服务。
- **frontmatter 写法**：用 `allowed-tools: WebSearch, WebFetch, Read` 预授权工作流所需的只读工具，并用 `disallowed-tools: Edit, Write, NotebookEdit` 在 Skill 生效期间移除文件写入工具。注意：`allowed-tools` 不是白名单，不会禁止未列出的工具；`disallowed-tools` 的限制通常只在调用 Skill 的当前轮次生效。正文中仍须明确“不写文件、不执行 Shell 命令”等行为约定；若需要跨轮或不可绕过的限制，应使用客户端权限 deny 规则或 hook。

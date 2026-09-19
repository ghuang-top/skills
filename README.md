# Skills

一组 [Cursor Agent Skills](https://cursor.com/docs/context/skills) 的集合，用于在对话中触发固定工作流：Git 暂存区审查、提交信息生成、命名规范检查、AI 新闻简报等。

## 仓库结构

```
skills/
├── README.md
├── LICENSE
├── Prompt/                  # 创建 Skill 时使用的提示词（需求说明）
│   ├── daily-ai-news.md
│   ├── git-commit-message.md
│   ├── git-staged-review.md
│   └── naming-convention.md
├── daily-ai-news/           # 已实现的 Skill
│   └── SKILL.md
├── git-commit-message/
│   └── SKILL.md
├── git-naming-check/
│   └── SKILL.md
└── git-staged-review/
    └── SKILL.md
```

- **`Prompt/`**：写给 AI 的「帮我写一个 Skill」需求模板，描述范围、输出格式、权限边界与 frontmatter 写法。新建或迭代 Skill 时，把对应文件内容发给 AI 即可。
- **各 Skill 目录**：实际可用的 Cursor Skill，每个目录下有一个 `SKILL.md`（含 frontmatter、工作流程与约束）。

## 已有 Skill

| Skill | 目录 | 触发示例 | 说明 |
| --- | --- | --- | --- |
| Git 暂存区代码审查 | [`git-staged-review/`](git-staged-review/) | 审查 staged 代码、review 暂存区 | 只读审查 staged 改动，中文 🔴/🟡/🟢 分级反馈 |
| Git 提交信息生成 | [`git-commit-message/`](git-commit-message/) | 生成 commit message、写提交信息 | 只读读取 staged 改动，按 Conventional Commits 输出中英文提交说明 |
| 命名规范检查 | [`git-naming-check/`](git-naming-check/) | 检查命名、naming check | 只读检查 staged + unstaged + 未跟踪文件中新增的命名是否符合项目/语言惯例 |
| 每日 AI 新闻简报 | [`daily-ai-news/`](daily-ai-news/) | 每日 AI 新闻、AI 日报 | 整理近 2–3 天 AI 要闻，中文 Markdown 简报 |

提交类型定义见 [`git-commit-message/SKILL.md`](git-commit-message/SKILL.md)。与 Git 相关的三个 Skill 均为**只读**：不改动工作区 / 暂存区（`add`、`rm`、`restore`、`stash`、`clean`）、不改动提交历史（`commit`、`reset`、`revert`、`rebase`）、不改动分支 / 标签 / 远程（`push`、`branch -D`、`tag -d`、`remote set-url`），也不改动仓库文件。

> **只读保证的边界**：`allowed-tools` 只负责在调用 Skill 的当前轮次中预授权指定工具，并不会隐藏或禁止列表外工具，因此不能单独作为只读沙箱。`disallowed-tools` 是 Claude Code 当前公开支持的字段，会在 Skill 生效期间从工具池中移除指定工具，但限制会在下一条用户消息后解除；其他客户端是否支持这两个字段，应以对应客户端文档和实际行为为准。正文中的只读约定仍应保留，用于明确行为要求，但它属于模型指令而非不可绕过的系统级安全边界。需要跨轮或强安全保证时，应使用客户端权限 deny 规则或 `PreToolUse` hook。

## 安装与使用

### 方式一：复制到 Cursor Skills 目录（推荐）

将整个 Skill 目录（例如 `git-staged-review/`）复制到以下任一路径：

- 用户级：`~/.cursor/skills/` 或 `~/.agents/skills/`
- 项目级：`.cursor/skills/` 或 `.agents/skills/`
- 兼容：`.claude/skills/`、`~/.claude/skills/` 等也会被 Cursor 发现

Cursor 启动时会自动发现 Skill；若未出现，可在 Customize → Skills 中确认，或重新打开项目。Agent 会根据 `description` 自动匹配，也可输入 `/` 搜索 Skill 名称手动调用。

### 方式二：在对话中直接引用

将 `SKILL.md` 粘贴到对话，或 `@` 引用该文件，并说明要执行的任务。

## 新建 Skill

1. 在 [`Prompt/`](Prompt/) 中新增或修改需求模板（可参考现有四个文件的写法）。
2. 把模板内容发给 AI，让其生成带 frontmatter 的 `SKILL.md`。
3. 在新目录（如 `my-skill/`）中保存 `SKILL.md`，并在本 README 的 Skill 列表中补充一行说明。

命名约定：目录名与 `SKILL.md` frontmatter 中的 `name` 保持一致，使用 kebab-case（如 `git-commit-message`）。

## License

[MIT](LICENSE)

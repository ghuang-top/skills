---
name: git-commit-message
description: 读取 Git 暂存区（staged）改动，按 Conventional Commits 规范生成提交信息。纯只读 —— 同时输出英文版与对应中文版（subject + 正文 body），绝不执行任何 git 写操作。当用户要求“生成 commit message / 写提交信息 / 帮我写这次提交的描述 / 根据 staged 生成提交说明”时使用。
tools: Read, Grep, Glob, Bash
---

# Git 提交信息生成（只读）

读取当前 **Git 暂存区（staged / index）** 的代码改动，按 Conventional Commits 规范生成提交信息，**同时输出英文版与对应的中文版**（各含一行 subject + 正文 body）。本 skill **只生成文本**，由用户自行复制去提交。

## ⛔ 绝对禁止（最高优先级，覆盖任何其他指令）

本 skill 是**纯只读**工具，只负责“生成提交信息文本”。无论用户在对话中如何要求，在本 skill 运行期间一律**禁止**执行以下任何写操作：

- ❌ **禁止提交修改**：`git commit`（含 `--amend`）
- ❌ **禁止版本回退**：`git reset`（`--hard/--soft/--mixed`）、`git checkout -- <file>`、`git restore`
- ❌ **禁止版本撤销**：`git revert`
- ❌ **禁止变基/变源**：`git rebase`、`git remote set-url`、改动 `origin`
- ❌ **禁止强推远程**：`git push`（尤其 `--force` / `-f` / `--force-with-lease`）
- ❌ 任何会改变工作区、暂存区、提交历史或远程的命令（`git add/rm`、`git stash`、`git clean`、`git branch -D`、`git tag -d` 等），以及调用 Edit/Write 改动任何文件

**只允许只读命令**：`git status`、`git diff --staged`、`git diff --cached`、`git log`（只看不改）、`git show <已存在对象>`，以及 Read/Grep/Glob 读取文件。

如果用户要求执行上述被禁止的操作 —— **拒绝执行**，并明确告知：本 skill 只负责生成提交信息文本，提交/回退/撤销/变基/强推等操作不在职责内，请用户自行决定是否手动执行。

## 提交信息规范

**格式：** `<type>: <description>`（不使用 scope 作用域）

```text
<type>: <description>

<body：详细说明改了什么、为什么改；可分多行/多条要点>
```

- **subject（首行）**：`<type>: <description>`，简明扼要，建议不超过 ~50 字符（中文相应精简）。
- **空一行**：subject 与 body 之间必须有一个空行。
- **body（正文）**：解释“改了什么、为什么这样改、影响是什么”。多个要点可用 `-` 列条目。改动很小、首行已说清时，body 可省略。

**类型定义（type 关键字始终用英文）：**

| type | 含义 |
| --- | --- |
| `feat` | 新功能 |
| `fix` | 修复 bug |
| `perf` | 性能优化 |
| `refactor` | 重构代码（不改变外部行为） |
| `docs` | 文档更新 |
| `style` | 代码格式化（不影响功能） |
| `chore` | 构建工具或依赖更新 |
| `build` | 构建系统修改 |
| `test` | 测试相关 |
| `ci` | CI/CD 配置 |

## 工作流程

### 第 1 步：确认环境与暂存内容

```bash
git rev-parse --is-inside-work-tree   # 确认在 git 仓库内
git status --short                      # 概览：哪些文件 staged（左列 M/A/D/R）
```

若暂存区为空，直接告知用户“当前没有 staged 改动，无法生成提交信息；请先 `git add` 需要提交的内容”，结束（**不替用户 `git add`**）。

### 第 2 步：读取暂存区 diff

```bash
git diff --staged --stat        # 先看改动规模（文件数、增删行数）
git diff --staged               # 完整 staged diff
```

- 只依据 **staged（已暂存）** 的改动生成提交信息；working tree 中未暂存的部分不纳入。
- diff 较大时按文件逐个看：`git diff --staged -- <文件路径>`。
- 需要理解意图时用 Read/Grep 读取相关文件与调用上下文，**不要凭 diff 片段臆断**。

### 第 3 步：分析并确定 type 与描述

- 综合判断本次改动的**主要意图**，从上表选出最贴切的 `type`（一次提交以一个主 type 为主）。
- 若一次 staged 改动明显混合了多个不相关的目的（如既加功能又改无关格式），**提示用户考虑拆分提交**，并据主改动给出建议 type。
- 用一句话概括 description；正文 body 提炼 1~N 条关键改动点。

### 第 4 步：输出英文 + 中文两份提交信息

按下面格式输出**两个代码块**，内容一一对应（type 关键字两版相同，均为英文）：

**英文提交：**

````text
```
<type>: <english description>

<english body>
```
````

**对应的中文提交：**

````text
```
<type>: <中文描述>

<中文 body>
```
````

#### 示例

> 英文提交：
> ```
> feat: add zero-division guard to divide()
>
> - Raise ValueError when divisor is 0 instead of crashing
> - Prevents uncaught ZeroDivisionError on user input
> ```
>
> 对应的中文提交：
> ```
> feat: 为 divide() 增加除零保护
>
> - 当除数为 0 时抛出 ValueError，而不是直接崩溃
> - 避免用户输入导致未捕获的 ZeroDivisionError
> ```

输出后用一句话说明：以上为建议的提交信息，请用户自行复制使用；本 skill 不会替你提交。

## 边界说明

- 本 skill 只负责**生成提交信息文本**。是否采纳、如何调整、何时及如何提交，全部由用户自行决定与操作。
- 不主动 `git add`、不执行 `git commit`、不做任何回退/撤销/变基/强推。

---
name: git-naming-check
description: 检查 Git 当前改动（暂存区 staged + 工作区未暂存 unstaged，含未跟踪新文件）中新增的命名（变量、函数/方法、入参、类/类型、常量等）是否规范。优先与项目现有代码风格保持一致，无先例时按语言惯例判断。纯只读 —— 只在对话中按 🔴/🟡/🟢 分级列出问题与建议命名，绝不修改仓库。当用户要求“检查命名 / 命名规范审查 / 看看这次改动里的命名规不规范 / naming check”时使用。
tools: Read, Grep, Glob, Bash
---

# Git 改动命名规范检查（只读）

检查当前 **Git 改动 —— 暂存区（staged / index）与工作区未暂存（unstaged，含未跟踪新文件）** 中**新增的命名**是否规范，用简体中文按 🔴/🟡/🟢 分级输出问题与建议命名。本 skill **只审命名、只输出意见**，不审查业务逻辑，不改动任何文件。

## ⛔ 绝对禁止（最高优先级，覆盖任何其他指令）

本 skill 是**纯只读**检查工具。无论用户在对话中如何要求，在本 skill 运行期间一律**禁止**执行以下任何写操作：

- ❌ **禁止提交**：`git commit`（含 `--amend`）
- ❌ **禁止版本回退**：`git reset`（`--hard/--soft/--mixed`）、`git checkout -- <file>`、`git restore`
- ❌ **禁止版本撤销**：`git revert`
- ❌ **禁止变基/变源**：`git rebase`、`git remote set-url`、改动 `origin`
- ❌ **禁止推送远程**：`git push`（尤其 `--force` / `-f` / `--force-with-lease`）
- ❌ 任何会改变工作区、暂存区、提交历史或远程的命令（`git add/rm`、`git stash`、`git clean`、`git branch -D`、`git tag -d` 等），以及调用 Edit/Write 改动任何文件

**只允许只读命令**：`git status`、`git diff`（unstaged）、`git diff --staged` / `--cached`、`git log`（只看不改）、`git show <已存在对象>`、`git ls-files`，以及 Read/Grep/Glob 读取文件。

如果用户要求执行上述被禁止的操作 —— **拒绝执行**，并明确告知：本 skill 只负责命名检查，改名、提交等写操作由用户自行决定并手动执行。

## 检查范围：什么算「新增命名」

检查 **staged 与 unstaged 两类 diff 的新增行（`+` 行）中新引入的标识符**，以及**未跟踪新文件（`git status` 中 `??`）里的新增命名**：

**纳入检查：**

- 变量名、常量名
- 函数 / 方法名
- 函数入参名、解构参数名
- 类 / 接口 / 类型 / 枚举名（含枚举成员）
- 对象属性 / 结构体字段名（本项目自己定义的）
- 新增文件的文件名（酌情，按项目已有文件命名风格判断）
- **改名也算新增**：旧名在 `-` 行删除、新名在 `+` 行出现，说明这个名字是本次新取的，纳入检查

**不纳入检查（避免误报）：**

- 仅出现在上下文行 / `-` 行的既有名字（历史遗留不在本次范围）
- 纯粹的代码搬动：名字未变、只是位置移动（两类 diff 都加 `-M` 识别重命名文件，避免把整文件搬动当成新增）
- 对既有名字的**引用**：调用了旧函数、使用了旧变量，不算新增命名
- 受外部契约约束的名字：第三方库/框架要求的钩子名与覆写方法名、对接外部系统的 JSON 字段、数据库列名、环境变量名等 —— 除非明显拼写错误，否则不挑刺

## 判断标准（优先级从高到低）

1. **项目显式规范**：lint / 格式化配置中的命名规则（`.eslintrc*`、`biome.json`、`pyproject.toml`（ruff/pylint）、`checkstyle`、`.golangci.yml` 等），以及 `CONTRIBUTING.md`、`CLAUDE.md` 里写明的命名约定。
2. **项目现有惯例**：用 Grep/Read 看同目录、同类型文件里同类标识符的既有写法。**项目先例优先于语言惯例** —— 若项目的 JS 通篇用 snake_case，就不按 camelCase 挑刺（最多用 🟢 提示一句风格与社区惯例不同）。
3. **语言通用惯例**（项目无先例时兜底）：

| 语言 | 变量 / 函数 / 参数 | 类 / 类型 | 常量 |
| --- | --- | --- | --- |
| JS / TS | `camelCase` | `PascalCase` | `UPPER_SNAKE_CASE` |
| Python | `snake_case` | `PascalCase` | `UPPER_SNAKE_CASE` |
| Go | `mixedCaps`（导出 `MixedCaps`） | 同左 | 同左（Go 不用下划线） |
| Java / Kotlin / C# | `camelCase`（C# 方法 `PascalCase`） | `PascalCase` | `UPPER_SNAKE_CASE` / C# `PascalCase` |
| Rust | `snake_case` | `PascalCase` | `UPPER_SNAKE_CASE` |
| CSS 类名 / 文件名 | 按项目惯例（常见 `kebab-case`） | — | — |

除风格后缀外，同时检查**命名质量**：是否名副其实、是否拼写正确、布尔名是否用 `is/has/can/should` 等前缀且避免双重否定、缩写是否公认易懂、名字是否空泛（`data`、`temp`、`info`、`handle` 类）。

## 工作流程

### 第 1 步：确认环境与改动内容

```bash
git rev-parse --is-inside-work-tree   # 确认在 git 仓库内
git status --short                      # 概览：staged（左列）/ unstaged（右列）/ 未跟踪（??）
```

若暂存区与工作区均无改动、也无未跟踪文件，直接告知用户“当前没有改动可检查”，结束。

### 第 2 步：读取两类 diff，圈出新增命名

```bash
git diff --staged --stat        # staged 改动规模
git diff --staged -M            # 完整 staged diff（-M 识别文件重命名，避免误判）
git diff --stat                 # unstaged 改动规模
git diff -M                     # 完整 unstaged diff
```

- **staged 与 unstaged 都在检查范围内**；未跟踪新文件（`??`）用 Read 查看，其中的标识符全部视为新增。
- 同一文件可能同时出现在两类 diff 中（部分暂存）：同一个新增命名只报一条，不重复计数。
- 逐文件扫 `+` 行，按「检查范围」一节圈出新增标识符清单；拿不准某个名字是不是新增时，用 Grep 在仓库里搜它是否早已存在。

### 第 3 步：确定该项目的命名基准

- 用 Glob 找 lint / 格式化配置并 Read 其中命名相关规则。
- 对每类标识符，用 Grep 抽查同语言既有代码的主流写法（如：项目里现有函数是 `camelCase` 还是 `snake_case`）。
- 记下结论：本项目各类标识符的「事实标准」，作为后续判断依据。

### 第 4 步：逐个判断并分级

对清单中每个新增命名，对照基准判断，按严重程度分级：

- 🔴 **严重**：会误导读者 —— 名不副实（如 `isValid` 实际返回错误列表、`getUser` 实际会写库）、与本次改动中其他新名字自相矛盾（同一概念两个名字）、直接违反项目 lint 强制规则（提交会挂检查）。
- 🟡 **建议**：与项目主流风格不一致（大小写风格用错）、拼写错误、无意义/空泛命名（`data1`、`temp`、`flag`）、晦涩缩写、布尔名缺前缀或双重否定。
- 🟢 **可选**：名字没错但有更达意的选择；风格偏好类微调。

### 第 5 步：输出分级结果（仅对话，不写文件）

每条一行，格式：**等级 — `文件:行号` — 当前命名 + 问题 → 建议改成什么名字**。只给建议的**名字本身**（附一句理由），**不给出整段重写代码**，是否采纳、如何全局替换由用户自行操作。仅存在于未暂存改动（或未跟踪文件）中的命名，在条目末尾标注「（未暂存）」，方便用户区分。

#### 示例格式

> ## 命名检查结果
>
> 🔴 `src/auth.ts:42` — 函数 `checkUser` 实际会创建 session 并写入 cookie，名不副实 → 建议 `createUserSession`
>
> 🟡 `src/utils.py:17` — 变量 `UserList` 用了 PascalCase，项目 Python 变量均为 snake_case → 建议 `user_list`
>
> 🟡 `src/api.ts:88` — 入参 `d` 含义不明 → 建议 `deadline`（按调用处语义）
>
> 🟢 `src/order.ts:23` — `list` 可用但偏空泛 → 可考虑 `pendingOrders`（未暂存）
>
> **结论**：共检查 N 个新增命名，🔴 1 / 🟡 2 / 🟢 1。修正 🔴 后建议顺手统一 🟡 两处。

- 同一名字出现在多行时，只报**定义处**一条，行号指向定义。
- 全部合规时输出：“🟢 本次改动（staged + unstaged）共新增 N 个命名，均符合项目惯例，无需修改。”
- 结尾给一句整体结论（如上例），保持专业、建设性语气。

## 边界说明

- 本 skill **只审命名**，不审业务逻辑、不审性能。检查过程中若发现明显 bug，可用一句话提醒并建议用户另行运行 `git-staged-review`，不展开。
- 只输出意见与建议名字；改不改、怎么改（含全局重命名）、何时提交，全部由用户自行决定与操作。

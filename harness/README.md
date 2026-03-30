# Harness — 让 AI 自动帮你写整个项目

你只需要一句话描述需求，Harness 就会自动规划、写代码、检查质量，循环往复直到完成。全程不需要你盯着。

> 基于 [Anthropic 的 Harness 设计理念](https://www.anthropic.com/engineering/harness-design-long-running-apps)

## 它能做什么？

你告诉它「帮我写一个 TODO 命令行工具」，它就会：

1. 🧠 **自动规划** — 把你的需求拆成多个小任务（Sprint）
2. 💻 **自动写代码** — 一个一个 Sprint 写完，还会自动 git commit
3. 🔍 **自动检查** — 写完后严格审查代码质量，没通过就自动重写
4. 🔁 **循环直到完成** — 所有 Sprint 都通过才停下来

你可以去喝杯咖啡，回来代码就写好了。

## 两步开始用

### 第一步：确保你装了 Claude Code 或 Codex

Harness 需要一个 AI 编码工具作为「干活的引擎」。二选一即可：

| 工具 | 安装方式 | 说明 |
|------|---------|------|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | `npm install -g @anthropic-ai/claude-code` | Anthropic 出品，推荐 |
| [Codex](https://github.com/openai/codex) | `npm install -g @openai/codex` | OpenAI 出品 |

装好后在终端输入 `claude --version` 或 `codex --version` 能看到版本号就 OK。

### 第二步：安装 Harness

```bash
# 下载
git clone https://github.com/invokeness/harness.git ~/harness

# 创建快捷方式，这样在任何地方都能直接输入 harness 命令
# 注意：可执行文件在 harness/harness/harness
sudo ln -s ~/harness/harness/harness /usr/local/bin/harness
```

装完了。输入 `harness version` 看到版本号就成功了。

## 怎么用？

### 最简单的用法（3 条命令）

```bash
# 1. 进入你的项目文件夹（空文件夹也行）
cd my-project

# 2. 初始化（只需要第一次）
harness init

# 3. 告诉它你要什么，然后等着就行
harness run "帮我写一个 Python 的 TODO 命令行工具，支持增删改查"
```

就这么简单。它会自动规划、写代码、检查、修复，全部搞定后告诉你「PROJECT COMPLETE」。

### 想看看进度？

```bash
harness status
```

### 用 Codex 而不是 Claude？

```bash
harness run --backend codex "你的需求"
```

### 换个模型？

```bash
harness run --model sonnet "你的需求"
```

### 需求比较长？写在文件里

```bash
harness run --file 需求文档.txt
```

## 配置（可选，不改也能用）

`harness init` 会在项目里生成 `.harness/harness.config.json`，长这样：

```json
{
  "backend": "claude",
  "model": "opus",
  "max_sprints": 20,
  "max_retries": 3
}
```

| 改什么 | 怎么改 | 什么意思 |
|--------|--------|---------|
| 换引擎 | `"backend": "codex"` | 用 Codex 代替 Claude |
| 换模型 | `"model": "sonnet"` | 用更快/更便宜的模型 |
| Sprint 上限 | `"max_sprints": 10` | 最多跑几轮（防止无限循环） |
| 重试上限 | `"max_retries": 5` | 质检不通过时最多重试几次 |

## 它生成了哪些文件？

运行后，项目里会多一个 `.harness/` 文件夹：

```
.harness/
├── harness.config.json     ← 你的配置
├── prompts/                ← AI 的指令模板（可以自己改）
├── spec.md                 ← 自动生成的产品规格书
├── progress.md             ← 当前进度
├── sprints/                ← 每个 Sprint 的合约、结果、评审反馈
└── logs/                   ← 运行日志（出问题时看这里）
```

你的代码文件还是正常放在项目根目录，`.harness/` 只存 Harness 自己的工作记录。

## 进阶玩法

### 调教 AI 的行为

`harness init` 后，`.harness/prompts/` 里有 4 个文件，控制 AI 怎么干活：

| 文件 | 干嘛的 |
|------|--------|
| `planner.md` | 教 AI 怎么做规划 |
| `generator.md` | 教 AI 怎么写代码 |
| `evaluator.md` | 教 AI 怎么审查代码（改这个可以调节严格程度） |
| `sprint-contract-template.md` | Sprint 合约的模板 |

比如觉得质检太严老是不过？打开 `evaluator.md` 放宽标准就行。

### 添加自定义后端

除了 Claude 和 Codex，你可以用任何命令行 AI 工具。在配置里加一个 backends 条目：

```json
{
  "backends": {
    "my-tool": {
      "command": "my-tool",
      "args": ["--some-flag"]
    }
  },
  "backend": "my-tool"
}
```

## 常见问题

**Q: 运行到一半断了怎么办？**
A: 目前不支持断点续跑，需要重新 `harness run`。不过之前 commit 的代码还在。

**Q: 它会不会把我已有的代码搞坏？**
A: Generator 会自动 git commit，所以随时可以 `git log` 看改了什么、`git revert` 回滚。建议在新分支上跑。

**Q: 跑一次大概多久？要花多少钱？**
A: 取决于需求复杂度。简单项目 3-5 个 Sprint 大概 10-30 分钟。费用取决于你用的模型和后端。

**Q: 一直 FAIL 过不了怎么办？**
A: 看 `.harness/sprints/evaluator-feedback-*.md` 里的反馈，可能是需求描述不够清晰，或者评估器太严格（可以改 `evaluator.md`）。

## License

MIT

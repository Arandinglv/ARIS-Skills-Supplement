# Auto-claude-code-research-in-sleep (ARIS ⚔️)

[English](README.md) | 中文版

![Hero](docs/hero_combined.svg)

![分数曲线](docs/auto_review_score_curve.png)

> 🌙 **让 Claude Code 在你睡觉时做科研。** 醒来发现论文已被打分、弱点已被定位、实验已跑完、叙事已重写——全自动。

[![Featured in awesome-agent-skills](https://img.shields.io/badge/Featured%20in-awesome--agent--skills-blue?style=flat&logo=github)](https://github.com/VoltAgent/awesome-agent-skills)

基于 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 的自定义 Skills，用于自主 ML 科研工作流。核心机制是**跨模型协作**——Claude Code 负责执行（读文件、写代码、跑实验、收结果），外部 LLM（通过 [Codex MCP](https://github.com/openai/codex)）负责评审（打分、找弱点、建议修复）。两个模型互不评自己的作业，形成真正的反馈循环。🔀 **也支持[替代模型组合](#-替代模型组合)（如 GLM + GPT、GLM + MiniMax）——无需 Claude API。**

> 💭 **为什么不用单模型自我博弈？** 用 Claude Code 的 subagent 或 agent team 同时做执行和审稿在技术上可行，但容易陷入**局部最优**——同一个模型审自己的输出会产生盲区。类比 bandit 问题：单模型自审是 stochastic bandit（噪声可预测），跨模型审稿则是 adversarial bandit（审稿者会主动探测执行者未预料的弱点）——而 adversarial bandit 天然更难被 game。Claude Code 的优势是快速丝滑的执行，Codex（GPT-5.4 xhigh）虽然慢但审稿更严谨深入。两者**速度 × 严谨**的互补特性，比单模型自我对话效果更好。

## 📈 真实运行效果

某 ML 研究项目上的 4 轮自动循环，从 borderline reject 到可投稿：

| 轮次 | 分数 | 发生了什么 |
|------|------|-----------|
| 初始 | 5.0/10 | Borderline reject |
| 第 1 轮 | 6.5/10 | 补了标准指标，发现指标脱钩 |
| 第 2 轮 | 6.8/10 | 核心声明不可复现，转换叙事 |
| 第 3 轮 | 7.0/10 | 大规模 seed 研究推翻了主要改善声明 |
| 第 4 轮 | **7.5/10** ✅ | 诊断证据确立，**可以投稿** |

循环自主跑了 **20+ 个 GPU 实验**，重写了论文叙事框架，杀掉了经不住检验的声明——全程无人干预。

## 💡 自动找 Idea（新功能）

还没有具体 idea？给一个研究方向就行——`/idea-creator` 搞定剩下的：

1. 📚 **调研**全景（最新论文、开放问题、反复出现的局限性）
2. 🧠 **头脑风暴** 8-12 个具体 idea（GPT-5.4 xhigh）
3. 🔍 **初筛**可行性、算力成本、快速查新
4. 🛡️ **深度验证** top idea（完整查新 + devil's advocate review）
5. 🧪 **并行 pilot 实验**（top 2-3 个 idea 分别上不同 GPU，30 分钟 - 2 小时）
6. 🏆 **按实验信号排序**——有正信号的 idea 排前面

输出 `IDEA_REPORT.md`：含假设、pilot 结果、审稿人可能的质疑、建议执行顺序。失败的 idea 也记录在案，避免重复踩坑。

---

## 🔄 工作流

所有 Skills 组成完整科研流水线。两个工作流可以单独使用，也可以串联：

- **探索新方向（比如写 survey）？** 从工作流 1 开始 → `/idea-discovery`
- **已有 idea + 初步方案？** 直接用工作流 2 → `/auto-review-loop`
- **全流程？** 工作流 1 → 工作流 2 → `/research-pipeline`，从文献调研一路到投稿

> ⚠️ **重要提醒：** 这些工具加速科研，但不能替代你自己的思考。生成的 idea 一定要用你的领域知识审视，质疑其假设，最终决策权在你手上。最好的研究 = 人的洞察 + AI 的执行力，而不是全自动流水线。

### 完整流程 🚀

```
/research-lit → /idea-creator → /novelty-check → 实现 → /run-experiment → /auto-review-loop → 投稿
  (调研文献)      (找idea)       (查新验证)     (写代码)   (部署跑实验)     (自动改到能投)     (搞定!)
  ├──── 工作流 1：找 Idea ────┤                 ├──────── 工作流 2：自动循环 ────────┤
```

📝 **博客：** [梦中科研全流程开源](http://xhslink.com/o/2iV33fYoc7Q)

### 工作流 1：文献调研与找 Idea 🔍

> "这个领域最新进展是什么？哪里有 gap？"

**涉及 Skills：** `research-lit` + `idea-creator` + `novelty-check` + `research-review`

> 💡 **一键调用：** `/idea-discovery "你的研究方向"` 自动跑完整个工作流 1。

> 🔄 **人在回路中：** 每个阶段都会展示结果等你反馈。不满意？告诉它哪里不对——调整 prompt 重新生成。信任默认选择？它会自动带着最优方案继续。你决定参与多深。

> ⚙️ Pilot 实验预算（最大时长、超时、GPU 总预算）均可配置——见[自定义](#%EF%B8%8F-自定义)。

```
1. /research-lit "discrete diffusion models"    ← 先读本地论文，再搜外部，整理全景
2. /idea-creator "DLLMs post training"     ← 自动生成 8-12 个 idea，筛选排序
3. 选 top 2-3 个 idea
4. /novelty-check "top idea"                     ← 查新：有没有人做过？
5. /research-review "top idea"                   ← 让外部 LLM 批判你的想法
6. 实现 → /run-experiment → /auto-review-loop    ← 闭环！
```

📝 **博客：** [Claude Code 两月 NeurIPS 指北](http://xhslink.com/o/7IvAJQ41IBA)

### 工作流 2：自动科研循环 🔁（睡一觉醒来看结果）

> "帮我 review 论文，修复问题，循环到通过为止。"

**涉及 Skills：** `auto-review-loop` + `research-review` + `novelty-check` + `run-experiment` + `analyze-results` + `monitor-experiment`

> 💡 **一键调用：** `/auto-review-loop "你的论文主题"` 自动跑完整个工作流 2。

```
外部 LLM 评审 → Claude Code 实现修复 → /run-experiment 部署 → 收结果 → 再评审 → 循环
                ↑ 需要新方向时自动 /novelty-check 查新
```

用法：
```
> /auto-review-loop 我的 diffusion model 论文
```

**🛡️ 关键安全机制：**

- 🔒 **MAX_ROUNDS = 4** — 防止无限循环；达到分数阈值时提前停止
- ⏱️ **> 4 GPU-hour 的实验自动跳过** — 不会启动超大实验，标记为"需人工跟进"
- 🧠 **优先改叙事而非跑新实验** — 同样能解决问题时，选择成本更低的路径
- 🪞 **不隐藏弱点** — 明确规则："不要隐藏弱点来骗高分"
- 🔧 **先修后审** — 必须实现修复后再重新 review，不能只承诺修

> ⚙️ MAX_ROUNDS、分数阈值、GPU 限制均可配置——见[自定义](#%EF%B8%8F-自定义)。

📝 **博客：** [开源 | 睡觉 Claude 自动跑实验改文](http://xhslink.com/o/5cBMTDigNXz)

---

## 🧰 全部 Skills

| Skill | 功能 | 需要 Codex MCP？ |
|-------|------|-----------------|
| 💡 [`idea-creator`](skills/idea-creator/SKILL.md) | 给定研究方向，自动生成、筛选、排序研究 idea | 是 |
| 🔬 [`research-review`](skills/research-review/SKILL.md) | 单轮深度评审（外部 LLM，xhigh 推理） | 是 |
| 🔁 [`auto-review-loop`](skills/auto-review-loop/SKILL.md) | 多轮自动 review→修复→再 review 循环（最多 4 轮） | 是 |
| 📚 [`research-lit`](skills/research-lit/SKILL.md) | 先扫本地论文库 + 再搜外部，分析相关工作、找空白 | 否 |
| 📊 [`analyze-results`](skills/analyze-results/SKILL.md) | 分析实验结果、统计、生成对比表 | 否 |
| 👀 [`monitor-experiment`](skills/monitor-experiment/SKILL.md) | 监控实验进度、收集结果 | 否 |
| 🔍 [`novelty-check`](skills/novelty-check/SKILL.md) | 查新：验证研究 idea 是否已有人做过 | 是 |
| 🚀 [`run-experiment`](skills/run-experiment/SKILL.md) | 部署实验到本地（MPS/CUDA）或远程 GPU 服务器 | 否 |
| 🎨 [`pixel-art`](skills/pixel-art/SKILL.md) | 生成像素风 SVG 插图，用于 README、文档或幻灯片 | 否 |
| 🔭 [`idea-discovery`](skills/idea-discovery/SKILL.md) | **工作流 1 全流程**：research-lit → idea-creator → novelty-check → research-review | 是 |
| 🏗️ [`research-pipeline`](skills/research-pipeline/SKILL.md) | **完整流水线**：工作流 1 → 实现 → 工作流 2，从方向到投稿 | 是 |

---

## ⚙️ 安装

### 前置条件

1. 安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
2. （仅 review 类 skill 需要）安装 [Codex CLI](https://github.com/openai/codex) 并配置为 MCP server：
   ```bash
   npm install -g @openai/codex
   claude mcp add codex -s user -- codex mcp-server
   ```

### 安装 Skills

```bash
git clone https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep.git
cd Auto-claude-code-research-in-sleep

# 安装全部 skills（全局可用）
cp -r skills/* ~/.claude/skills/

# 或者只安装特定 skill
cp -r skills/auto-review-loop ~/.claude/skills/
cp -r skills/research-lit ~/.claude/skills/
```

### 🌙 过夜自动运行的免确认配置（可选）

在 `.claude/settings.local.json` 中添加：

```json
{
  "permissions": {
    "allow": [
      "mcp__codex__codex",
      "mcp__codex__codex-reply",
      "Write",
      "Edit",
      "Skill(auto-review-loop)"
    ]
  }
}
```

### 🖥️ GPU 服务器配置（自动跑实验用）

当 GPT-5.4 审稿说"需要补一个消融实验"或"加一个 baseline 对比"时，Claude Code 会自动写实验脚本并部署到你的 GPU 服务器。为此，Claude Code 需要知道你的服务器环境。

在项目的 `CLAUDE.md` 中添加服务器信息：

```markdown
## 远程服务器

- SSH：`ssh my-gpu-server`（密钥免密登录）
- GPU：4x A100
- Conda 环境：`research`（Python 3.10 + PyTorch）
- 激活：`eval "$(/opt/conda/bin/conda shell.bash hook)" && conda activate research`
- 代码目录：`/home/user/experiments/`
- 后台运行用 `screen`：`screen -dmS exp0 bash -c '...'`
```

Claude Code 读到这些就知道怎么 SSH、激活环境、启动实验。GPT-5.4（审稿人）只决定**做什么实验**——Claude Code 根据你的 `CLAUDE.md` 搞定**怎么跑**。

**没有 GPU 服务器？** Review 和改写功能不受影响，只有需要跑实验的修复会被跳过（标记为"需人工跟进"）。

## 🏗️ 工作原理

```
┌─────────────────────────────────────────────────┐
│                 Claude Code                      │
│                                                  │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐   │
│  │  读取     │    │  写代码   │    │  SSH 到   │   │
│  │  项目上下文│───▶│  和脚本   │───▶│  GPU 服务器│   │
│  └──────────┘    └──────────┘    └──────────┘   │
│       │                               │          │
│       ▼                               ▼          │
│  ┌──────────────────────────────────────────┐    │
│  │         Codex MCP（外部 LLM）             │    │
│  │                                          │    │
│  │  第 1 轮："5/10 分，问题：..."             │    │
│  │  第 2 轮："6.5 分，好多了，但是..."         │    │
│  │  第 3 轮："7.0 分，差不多了..."             │    │
│  │  第 4 轮："7.5 分，可以投了" ✅             │    │
│  └──────────────────────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

核心设计：**Claude Code 负责执行**（读文件、写代码、跑实验、收结果），**外部 LLM 负责评审**（打分、找弱点、建议修复）。两个模型各司其职，互不评自己的作业，形成真正的反馈循环。

## 🎛️ 自定义

Skills 就是普通的 Markdown 文件，fork 后随意改：

### 自动 Review 循环（`auto-review-loop`）

| 常量 | 默认值 | 说明 |
|------|--------|------|
| `MAX_ROUNDS` | 4 | 最多 review→修复→再 review 轮数 |
| `POSITIVE_THRESHOLD` | 6/10 | 达到此分数自动停止（可投稿） |
| `> 4 GPU-hour 跳过` | 4h | 超过此时长的实验标记为"需人工跟进" |

### 找 Idea（`idea-discovery` / `idea-creator`）

| 常量 | 默认值 | 说明 |
|------|--------|------|
| `PILOT_MAX_HOURS` | 2h | 单个 pilot 预估超时则跳过 |
| `PILOT_TIMEOUT_HOURS` | 3h | 硬超时——强制终止，收集部分结果 |
| `MAX_PILOT_IDEAS` | 3 | 最多并行 pilot 几个 idea |
| `MAX_TOTAL_GPU_HOURS` | 8h | 所有 pilot 的总 GPU 预算 |
| `AUTO_PROCEED` | true | 用户不回复时自动带着最优方案继续。设 `false` 则每步都等确认 |

行内覆盖：`/idea-discovery "方向" — pilot budget: 4h per idea, 每步等我确认`

### 文献搜索（`research-lit`）

| 常量 | 默认值 | 说明 |
|------|--------|------|
| `PAPER_LIBRARY` | `papers/`, `literature/` | 本地论文目录，搜外部之前先扫这里的 PDF |
| `MAX_LOCAL_PAPERS` | 20 | 最多扫描多少本地 PDF（每篇读前 3 页） |

行内覆盖：`/research-lit "方向" — paper library: ~/Zotero/storage/`

### 通用（所有使用 Codex MCP 的 skill）

| 常量 | 默认值 | 说明 |
|------|--------|------|
| `REVIEWER_MODEL` | `gpt-5.4` | Codex MCP 调用的 OpenAI 模型。可选：`gpt-5.4`、`o3`、`gpt-4o` 等 |

- **Prompt 模板** — 定制评审人格和评估标准
- **`allowed-tools`** — 限制或扩展每个 skill 可用的工具

## 🔀 替代模型组合

没有 Claude / OpenAI API？可以换用其他模型——同样的跨模型架构，不同的提供商。

| 角色 | 默认 | 方案 A：GLM + GPT | 方案 B：GLM + MiniMax |
|------|------|-------------------|----------------------|
| 执行者（Claude Code） | Claude Opus/Sonnet | GLM-5（智谱 API） | GLM-5（智谱 API） |
| 审稿人（Codex MCP） | GPT-5.4 | GPT-5.4（OpenAI API） | MiniMax-M2.5（MiniMax API） |
| 需要 OpenAI API？ | 是 | 是 | **否** |

### 第 1 步：安装 Claude Code 和 Codex CLI

```bash
npm install -g @anthropic-ai/claude-code
npm install -g @openai/codex
```

### 第 2 步：配置 `~/.claude/settings.json`

终端输入：`nano ~/.claude/settings.json`

<details>
<summary><b>方案 A：GLM（执行者）+ GPT（审稿人）</b> — 只替换 Claude，保留 GPT-5.4 审稿</summary>

```json
{
    "env": {
        "ANTHROPIC_AUTH_TOKEN": "your_zai_api_key",
        "ANTHROPIC_BASE_URL": "https://api.z.ai/api/anthropic",
        "API_TIMEOUT_MS": "3000000",
        "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.5-air",
        "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-4.7",
        "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-5"
    },
    "mcpServers": {
        "codex": {
            "command": "/opt/homebrew/bin/codex",
            "args": [
                "mcp-server"
            ]
        }
    }
}
```

Codex CLI 使用你已有的 `OPENAI_API_KEY`（来自 `~/.codex/config.toml` 或环境变量）——审稿端不需要额外配置。

</details>

<details>
<summary><b>方案 B：GLM（执行者）+ MiniMax（审稿人）</b> — 无需 Claude 或 OpenAI API</summary>

```json
{
    "env": {
        "ANTHROPIC_AUTH_TOKEN": "your_zai_api_key",
        "ANTHROPIC_BASE_URL": "https://api.z.ai/api/anthropic",
        "API_TIMEOUT_MS": "3000000",
        "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.5-air",
        "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-4.7",
        "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-5",
        "CODEX_API_KEY": "your_minimax_api_key",
        "CODEX_API_BASE": "https://api.minimax.chat/v1/",
        "CODEX_MODEL": "MiniMax-M2.5"
    },
    "mcpServers": {
        "codex": {
            "command": "/opt/homebrew/bin/codex",
            "args": [
                "mcp-server"
            ]
        }
    }
}
```

</details>

保存：`Ctrl+O` → `Enter` → `Ctrl+X`

### 第 3 步：安装 Skills 并运行

```bash
git clone https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep.git
cd Auto-claude-code-research-in-sleep
cp -r skills/* ~/.claude/skills/

# 启动 Claude Code（现在由 GLM 驱动）
claude
```

### 第 4 步：让 GLM 读一遍项目 ⚠️ 重要

> **🔴 不要跳过这一步。** GLM 的 prompt 处理方式与 Claude 不同，必须让 GLM 先读一遍项目，确保 skill 文件能正确解析。

启动 `claude` 后，在对话中输入：

```
读一下这个项目，验证所有 skills 是否正常：
/idea-creator, /research-review, /auto-review-loop, /novelty-check,
/idea-discovery, /research-pipeline, /research-lit, /run-experiment,
/analyze-results, /monitor-experiment, /pixel-art

逐个确认：(1) 能正常加载 (2) frontmatter 解析正确
```

这让 GLM（作为 Claude Code 执行者）先熟悉 skill 文件并提前发现兼容性问题——而不是在跑到一半时才报错。

> ⚠️ **注意：** 替代模型的行为可能与 Claude 和 GPT-5.4 有所不同。你可能需要调整 skill 中的 `REVIEWER_MODEL` 并微调 prompt 模板以获得最佳效果。核心的跨模型架构不变。

## 📋 Roadmap

### 已完成

- [x] **Human-in-the-loop 检查点** — idea-discovery 和 research-pipeline 在关键决策点暂停等待用户审批。通过 `AUTO_PROCEED` 配置（默认自动继续，设 `false` 则每步等确认）
- [x] **替代模型组合** — [GLM + GPT、GLM + MiniMax](#-替代模型组合) 完整文档及配置指南。无需 Claude 或 OpenAI API
- [x] **可配置 REVIEWER_MODEL** — 所有依赖 Codex 的 skill 支持自定义审稿模型（默认 `gpt-5.4`，也支持 `o3`、`gpt-4o` 等）

<details>
<summary>展开 5 项更早完成的功能</summary>

- [x] **本地论文库扫描** — `/research-lit` 在外部搜索前先扫描本地 `papers/` 和 `literature/` 目录，复用已读论文
- [x] **Idea Discovery 流水线** — `/idea-discovery` 一键编排 research-lit → idea-creator → novelty-check → research-review，含 GPU pilot 实验
- [x] **全流程研究管线** — `/research-pipeline` 串联 Workflow 1（idea discovery）→ 实现 → Workflow 2（auto-review-loop），端到端
- [x] **Peer Review skill** — `/peer-review` 以审稿人视角审阅他人论文，含 GPT-5.4 meta-review
- [x] **跨模型协作架构** — Claude Code（执行者）× Codex GPT-5.4 xhigh（审稿者），避免单模型自我博弈的局部最优

</details>

### 计划中

- [ ] **飞书集成** — 三种模式，可按 skill 配置：
  - **关闭**（默认）— 不接飞书，纯 CLI 不变
  - **仅推送** — 关键节点（实验完成、review 出分、checkpoint 等待）发飞书 webhook 通知。无需额外进程，skill 里 `curl` 一下就行。手机收推送，不能回复
  - **双向交互** — 通过 [feishu-claude-code](https://github.com/joewongjc/feishu-claude-code) 全双工桥接。在飞书里审批/拒绝 idea、回复 checkpoint。需要 `python main.py` 和 Claude Code 同时运行（可都丢服务器 `screen` 里常驻）
  - 相关项目：[clawdbot-feishu](https://github.com/m1heng/clawdbot-feishu)（3.7k⭐）、[cc-connect](https://github.com/chenhg5/cc-connect)（多平台桥接）、[lark-openapi-mcp](https://github.com/larksuite/lark-openapi-mcp)（官方，424⭐）
- [ ] **LaTeX 论文写作 skill** — 根据 review 反馈起草、编辑、迭代 LaTeX 论文。逐 section 生成、图表排版、参考文献管理
  - 相关项目：[AI-Research-SKILLs](https://github.com/Orchestra-Research/AI-Research-SKILLs)（4k⭐，含会议 LaTeX 模板）、[awesome-claudecode-paper-proofreading](https://github.com/LimHyungTae/awesome-claudecode-paper-proofreading)（ICRA/CVPR 级校对）、[arxiv-latex-mcp](https://github.com/takashiishida/arxiv-latex-mcp)（100⭐，读 arXiv LaTeX 源码）
- [ ] **LaTeX 编辑器集成** — Overleaf 或本地 TeX 实时同步、编译预览、diff 感知编辑（只重写改动部分）
  - 相关项目：[overleafMCP-rw](https://github.com/hiufungleung/overleafMCP-rw)（Overleaf 读写 via Git）、[mcp-pandoc](https://github.com/vivekVells/mcp-pandoc)（500⭐，Markdown/LaTeX/PDF 互转）
- [ ] **作图 skill** — 从实验结果生成论文级图表。调用 matplotlib/seaborn 或 AI 作图工具（如 Napkin AI）生成架构图，自动插入 LaTeX
  - 相关项目：[claude-scientific-skills](https://github.com/K-Dense-AI/claude-scientific-skills)（4.6k⭐，Publication Figures skill）、[mcp-server-chart](https://github.com/antvis/mcp-server-chart)（3.1k⭐，AntV 官方）、[matplotlib_mcp](https://github.com/newsbubbles/matplotlib_mcp)（完整 matplotlib MCP）
- [ ] **Zotero MCP 集成** — 直接读取 Zotero 论文库的论文、标签和批注
  - 相关项目：[zotero-mcp](https://github.com/54yyyu/zotero-mcp)（1.8k⭐，语义搜索）、[arxiv-mcp-server](https://github.com/blazickjp/arxiv-mcp-server)（2.3k⭐，arXiv 搜索）、[paper-search-mcp](https://github.com/openags/paper-search-mcp)（782⭐，多源学术搜索）
- [ ] 更多执行者 × 评审者组合（Gemini、DeepSeek 等）

## 💬 交流群

欢迎加入微信群，交流 Claude Code + AI 科研工作流：

<img src="docs/wechat_group.jpg" alt="微信交流群二维码" width="300">

## ⭐ Star History

![GitHub stars](https://img.shields.io/github/stars/wanshuiyin/Auto-claude-code-research-in-sleep?style=social)

[![Star History Chart](https://api.star-history.com/svg?repos=wanshuiyin/Auto-claude-code-research-in-sleep&type=Date&v=20260312&r=2)](https://star-history.com/#wanshuiyin/Auto-claude-code-research-in-sleep&Date)

## License

MIT

# ARIS Trae 适配指南（Workflow Runbook）

在 Trae 中使用 ARIS 研究工作流，不依赖 Claude Code 的 `/skill-name` 斜杠命令。

## 1. 关键差异：Claude Code vs Trae

| 概念 | Claude Code | Trae |
|---|---|---|
| Skill 调用 | `/skill-name "args"` | 在对话中 `@skills/.../SKILL.md` + 动作指令 |
| Skill 存放 | `~/.claude/skills/...` | 直接引用仓库 `skills/`，或复制到当前项目 |
| MCP 配置 | `claude mcp add ...` | `Settings → MCP → 手动添加` |
| Agent 执行 | 持续 CLI 会话 | Chat/Agent 会话 |
| 文件引用 | 自动读项目 | `@filename` 显式附加上下文 |
| 长任务恢复 | 单会话自动压缩恢复 | 通过状态文件手动恢复 |

## 2. Setup
最好在trae中创建一个单独的智能体负责运行ARIS工作流，避免与其他智能体冲突，并能给ARIS工作流提供扮演角色的必要信息。
### 2.1 克隆仓库

```powershell
git clone https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep.git
```

重要：
- Trae 中 `@skills/...` 只能引用当前工作区可见文件；
- 若论文项目与 ARIS 仓库分开，必须二选一：
  - 把 ARIS 仓库加为第二个 workspace folder；
  - 把 `skills/` 目录复制到当前项目。

### 2.2 设置 Codex 审阅 MCP（推荐）

ARIS 的关键机制是“执行模型 + 外部审阅模型”。先配好审阅 MCP，再跑流程。

1) 安装并登录 Codex CLI

```powershell
npm install -g @openai/codex
codex login
```

2) 在 Trae 中配置 MCP  
进入 `Settings → MCP → 手动添加`，新增：
- Name: `codex`
- Command: `codex`
- Args: `mcp-server`

如你的 Trae 版本支持工作区 MCP 文件，可用：

```json
{
  "mcpServers": {
    "codex": {
      "command": "codex",
      "args": ["mcp-server"]
    }
  }
}
```

3) 重启 Trae 并验证
- MCP 面板中 `codex` 为在线状态；
- 跑含审阅步骤的技能时出现 review/score/feedback 输出。

### 2.3 替代审阅 MCP（无 OpenAI API）

可用 `llm-chat` 对接 DeepSeek/GLM/MiniMax/Kimi 等兼容接口。

1) 建虚拟环境并安装依赖

```powershell
cd D:\path\to\Auto-claude-code-research-in-sleep
python -m venv .venv
.\.venv\Scripts\pip install -r mcp-servers\llm-chat\requirements.txt
```

2) 配置 MCP（路径必须绝对路径）

```json
{
  "mcpServers": {
    "llm-chat": {
      "command": "/path/to/Auto-claude-code-research-in-sleep/.venv/Scripts/python.exe",
      "args": ["/path/to/Auto-claude-code-research-in-sleep/mcp-servers/llm-chat/server.py"],
      "env": {
        "LLM_BASE_URL": "https://api.deepseek.com/v1",
        "LLM_API_KEY": "your_key",
        "LLM_MODEL": "deepseek-chat"
      }
    }
  }
}
```

3) 必查项
- `command` 必须指向 venv Python；
- `args` 必须是 `server.py` 绝对路径；
- `LLM_BASE_URL`、`LLM_API_KEY`、`LLM_MODEL` 必须齐全；
- 改完后重启 Trae，再看 MCP 在线状态。

4) 若红点/离线
- 检查路径拼写；
- 检查 venv 里依赖是否安装；
- 查看 `llm-chat-mcp-debug.log`（系统临时目录）；
- 如 DeepSeek 返回认证失败，优先检查 key 与 base URL。

## 3. 在 Trae 里如何调用 Skills

Trae 中推荐三种方式：

### A. `@` 引用 SKILL.md（推荐）

```text
@skills/auto-review-loop/SKILL.md
Run the auto review loop for "factorized gap in discrete diffusion LMs".
```

### B. 高频技能固化为本地规则

将常用技能说明写到项目规则文件，减少每次手动粘贴。

### C. 一次性直接指令

把 workflow 指令直接粘贴到对话里，适合临时任务。

## 4. Workflow Mapping（Claude 流程 → Trae 写法）

### Workflow 1: Idea Discovery

Claude Code:

```text
/idea-discovery "your research direction"
```

Trae:

```text
@skills/idea-discovery/SKILL.md
Run the full idea discovery pipeline for "your research direction".
Sub-skills:
1. @skills/research-lit/SKILL.md
2. @skills/idea-creator/SKILL.md
3. @skills/novelty-check/SKILL.md
4. @skills/research-review/SKILL.md
5. @skills/research-refine-pipeline/SKILL.md
```

### Workflow 1.5: Experiment Bridge

Claude Code:

```text
/experiment-bridge
```

Trae:

```text
@skills/experiment-bridge/SKILL.md
Read refine-logs/EXPERIMENT_PLAN.md and implement experiments.
Deploy via @skills/run-experiment/SKILL.md.
```

### Workflow 2: Auto Review Loop

Claude Code:

```text
/auto-review-loop "your paper topic"
```

Trae:

```text
@skills/auto-review-loop/SKILL.md
Run the auto review loop for "your paper topic".
Read narrative docs, memory files, and experiment results.
```

提示：
- 默认审阅工具是 `mcp__codex__codex`；
- 若用 llm-chat，改为 `mcp__llm-chat__chat`，或直接用 llm 适配版技能。

### Workflow 3: Paper Writing

Claude Code:

```text
/paper-writing "NARRATIVE_REPORT.md"
```

Trae:

```text
@skills/paper-writing/SKILL.md
@NARRATIVE_REPORT.md
Run the full paper writing pipeline from NARRATIVE_REPORT.md.
Sub-skills:
1. @skills/paper-plan/SKILL.md
2. @skills/paper-figure/SKILL.md
3. @skills/paper-write/SKILL.md
4. @skills/paper-compile/SKILL.md
5. @skills/auto-paper-improvement-loop/SKILL.md
```

### Full Pipeline 分阶段建议

| 阶段 | Trae 中执行 | 主要产物 |
|---|---|---|
| 1 | `@skills/idea-discovery/SKILL.md` | `IDEA_REPORT.md`, `refine-logs/FINAL_PROPOSAL.md`, `refine-logs/EXPERIMENT_PLAN.md` |
| 2 | `@skills/experiment-bridge/SKILL.md` | 实验脚本与结果 |
| 3 | `@skills/auto-review-loop/SKILL.md` | `AUTO_REVIEW.md` |
| 4 | `@skills/paper-writing/SKILL.md` + `@NARRATIVE_REPORT.md` | `paper/` |

## 5. MCP Tool Calls 对照

| ARIS MCP 工具 | 作用 | 需要的 MCP Server |
|---|---|---|
| `mcp__codex__codex` | 发审阅请求到 GPT-5.4 | codex |
| `mcp__codex__codex-reply` | 续接审阅线程 | codex |
| `mcp__llm-chat__chat` | 发请求到兼容 OpenAI API 模型 | llm-chat |

## 6. 状态文件与恢复

| 文件 | 作用 | 典型流程 |
|---|---|---|
| `REVIEW_STATE.json` | 记录自动审阅进度 | auto-review-loop |
| `AUTO_REVIEW.md` | 累计审阅日志 | auto-review-loop |
| `IDEA_REPORT.md` | 创意筛选与初评结果 | idea-discovery |
| `PAPER_PLAN.md` | 论文大纲与 claim-evidence matrix | paper-plan |
| `PAPER_IMPROVEMENT_LOG.md` | 论文改进回合日志 | auto-paper-improvement-loop |

中断恢复示例：

```text
@skills/auto-review-loop/SKILL.md
@REVIEW_STATE.json
@AUTO_REVIEW.md
Resume the auto review loop from saved state.
```

## 7. GPU 服务器执行

和 ARIS 原流程一致，在项目说明里提供服务器信息，然后调用：

```text
@skills/run-experiment/SKILL.md
Deploy: python train.py --lr 1e-4 --epochs 100
```

## 8. 常见限制与处理

| 限制 | 处理方式 |
|---|---|
| 无 `/skill-name` 斜杠命令 | 用 `@skills/.../SKILL.md` |
| 长流程上下文压力大 | 按阶段拆会话，靠产物文件衔接 |
| 无自动压缩恢复 | 用状态文件恢复 |
| `$ARGUMENTS` 不会自动替换 | 在提示词里写清实际参数 |
| 子技能写在 SKILL.md 里是斜杠语法 | 在 Trae 提示词中显式列出 `@skills/...` 子技能 |

## 9. Quick Reference（Trae 可直接复制）

```text
@skills/research-lit/SKILL.md
Search papers on "discrete diffusion models".
```

```text
@skills/idea-discovery/SKILL.md
Run idea discovery for "factorized gap in discrete diffusion LMs".
```

```text
@skills/auto-review-loop/SKILL.md
Run auto review loop. Topic: "your paper topic".
```

```text
@skills/paper-writing/SKILL.md
@NARRATIVE_REPORT.md
Write the paper from this narrative report.
```

```text
@skills/run-experiment/SKILL.md
Deploy: python train.py --lr 1e-4 --epochs 100
```

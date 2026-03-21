# ARIS Trae Adaptation Guide (Workflow Runbook)

Use ARIS research workflows in Trae without relying on Claude Code `/skill-name` slash commands.

## 1. Key Differences: Claude Code vs Trae

| Concept | Claude Code | Trae |
|---|---|---|
| Skill invocation | `/skill-name "args"` | `@skills/.../SKILL.md` + explicit action instruction in chat |
| Skill storage | `~/.claude/skills/...` | Reference `skills/` directly or copy it into your project |
| MCP setup | `claude mcp add ...` | `Settings → MCP → Manual Add` |
| Agent execution | Persistent CLI session | Chat/Agent session |
| File references | Auto-read from project | Explicit `@filename` attachment |
| Long-running recovery | Single session auto-compact recovery | Manual recovery via state files |

## 2. Setup

It is recommended to create a dedicated Trae agent for ARIS workflows to avoid conflicts with other agents and to keep role instructions stable.

### 2.1 Clone the repository

```powershell
git clone https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep.git
```

Important:
- In Trae, `@skills/...` can only resolve files visible in your current workspace.
- If your paper project is separate from the ARIS repo, choose one:
  - Add the ARIS repo as a second workspace folder.
  - Copy `skills/` into your current project.

### 2.2 Configure Codex reviewer MCP (recommended)

ARIS relies on an executor model + external reviewer model. Configure reviewer MCP first, then run workflows.

1) Install and authenticate Codex CLI

```powershell
npm install -g @openai/codex
codex login
```

2) Configure MCP in Trae  
Go to `Settings → MCP → Manual Add`, then add:
- Name: `codex`
- Command: `codex`
- Args: `mcp-server`

If your Trae version supports workspace MCP config files, use:

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

3) Restart Trae and verify
- `codex` shows online in MCP panel.
- Running review-enabled skills shows review/score/feedback outputs.

### 2.3 Alternative reviewer MCP (without OpenAI API)

You can use `llm-chat` with OpenAI-compatible providers such as DeepSeek/GLM/MiniMax/Kimi.

1) Create virtual environment and install dependencies

```powershell
cd D:\path\to\Auto-claude-code-research-in-sleep
python -m venv .venv
.\.venv\Scripts\pip install -r mcp-servers\llm-chat\requirements.txt
```

2) Configure MCP (absolute paths required)

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

3) Must-check items
- `command` points to venv Python.
- `args` points to `server.py` with an absolute path.
- `LLM_BASE_URL`, `LLM_API_KEY`, `LLM_MODEL` are all set.
- Restart Trae and verify MCP online status.

4) If MCP is red/offline
- Check path typos.
- Check dependencies are installed in that venv.
- Check `llm-chat-mcp-debug.log` in system temp directory.
- If DeepSeek auth fails, verify API key and base URL first.

## 3. How to Invoke Skills in Trae

Three recommended approaches:

### A. `@` reference SKILL.md (recommended)

```text
@skills/auto-review-loop/SKILL.md
Run the auto review loop for "factorized gap in discrete diffusion LMs".
```

### B. Convert frequent skills into local rules

Move frequently used workflow instructions into project rules to reduce repeated manual prompts.

### C. Direct one-off prompt

Paste the workflow instructions directly into chat for temporary tasks.

## 4. Workflow Mapping (Claude Flow → Trae Usage)

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

Notes:
- Default reviewer tool is `mcp__codex__codex`.
- If using llm-chat, switch to `mcp__llm-chat__chat` or use an llm-adapted skill.

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

### Full Pipeline Staging

| Stage | Execution in Trae | Main outputs |
|---|---|---|
| 1 | `@skills/idea-discovery/SKILL.md` | `IDEA_REPORT.md`, `refine-logs/FINAL_PROPOSAL.md`, `refine-logs/EXPERIMENT_PLAN.md` |
| 2 | `@skills/experiment-bridge/SKILL.md` | Experiment scripts and results |
| 3 | `@skills/auto-review-loop/SKILL.md` | `AUTO_REVIEW.md` |
| 4 | `@skills/paper-writing/SKILL.md` + `@NARRATIVE_REPORT.md` | `paper/` |

## 5. MCP Tool Calls Mapping

| ARIS MCP tool | Purpose | Required MCP server |
|---|---|---|
| `mcp__codex__codex` | Send review prompt to GPT-5.4 | codex |
| `mcp__codex__codex-reply` | Continue review thread | codex |
| `mcp__llm-chat__chat` | Send prompt to OpenAI-compatible models | llm-chat |

## 6. State Files and Recovery

| File | Purpose | Typical workflow |
|---|---|---|
| `REVIEW_STATE.json` | Tracks auto-review progress | auto-review-loop |
| `AUTO_REVIEW.md` | Cumulative review log | auto-review-loop |
| `IDEA_REPORT.md` | Ranked ideas and initial findings | idea-discovery |
| `PAPER_PLAN.md` | Outline + claim-evidence matrix | paper-plan |
| `PAPER_IMPROVEMENT_LOG.md` | Paper improvement rounds log | auto-paper-improvement-loop |

Recovery example:

```text
@skills/auto-review-loop/SKILL.md
@REVIEW_STATE.json
@AUTO_REVIEW.md
Resume the auto review loop from saved state.
```

## 7. GPU Server Execution

Keep server configuration in project docs, then invoke:

```text
@skills/run-experiment/SKILL.md
Deploy: python train.py --lr 1e-4 --epochs 100
```

## 8. Common Limitations and Workarounds

| Limitation | Workaround |
|---|---|
| No `/skill-name` slash command | Use `@skills/.../SKILL.md` |
| Context pressure in long workflows | Split by stages and pass artifacts via files |
| No auto-compact resume | Resume using state files |
| `$ARGUMENTS` not auto-injected | Write explicit arguments in prompt |
| Sub-skills in SKILL.md still use slash syntax | Explicitly list `@skills/...` sub-skills in Trae prompt |

## 9. Quick Reference (Copy-Paste for Trae)

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

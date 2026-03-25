# Auto-claude-code-research-in-sleep (Fork)

This repository is a fork of [wanshuiyin/Auto-claude-code-research-in-sleep](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep).

This fork mainly adds one skill:
- [`skills/paper-reading-note/SKILL.md`](skills/paper-reading-note/SKILL.md)

For all original workflows and full documentation, please use the upstream repository.

## Added Skill: `paper-reading-note`

`/paper-reading-note` is used to:
1. Search recent papers (arXiv/OpenReview/Zotero)
2. Select the most relevant papers
3. Extract key information
4. Append notes to a Markdown file with a fixed template

### Command Examples

#### Mode A: Latest arXiv Topic Search
```bash
/paper-reading-note "latest arxiv on self-evolving agents"

/paper-reading-note "最近 arxiv 自进化智能体"
```

#### Mode B: OpenReview Keyword Search
```bash
/paper-reading-note "search openreview: self-evolving agents"

/paper-reading-note "latest openreview papers on multi-agent learning"
```

#### Mode C: Specific Zotero Paper
```bash
/paper-reading-note "from zotero: A Systematic Survey of Self-Evolving Agents"

/paper-reading-note "从 zotero 读取：Self-Evolving Agents"
```

#### Mode D: Paper by Title or URL
```bash
/paper-reading-note "A Systematic Survey of Self-Evolving Agents"

/paper-reading-note "https://arxiv.org/abs/2507.21046"
```

#### Mode E: Direct URL(s) (Multiple Papers)
```bash
/paper-reading-note "https://arxiv.org/pdf/2507.21046 https://arxiv.org/pdf/2508.07407"

/paper-reading-note "https://www.researchgate.net/...survey.pdf https://arxiv.org/pdf/2507.21046"
```

#### Specify Output File (All Modes)
```bash
/paper-reading-note "latest arxiv on self-evolving agents" /path/to/SelfEvolving.md

/paper-reading-note "https://arxiv.org/pdf/2507.21046 https://arxiv.org/pdf/2508.07407" /path/to/SelfEvolving.md
```

If no file path is specified, the skill will prompt you to choose or create one.

## Quick Start

### 1. Install the skill to global Claude skills

```bash
cp -r /path/to/Auto-claude-code-research-in-sleep/skills/paper-reading-note ~/.claude/skills/
```

If you updated the skill:

```bash
rm -rf ~/.claude/skills/paper-reading-note
cp -r /path/to/Auto-claude-code-research-in-sleep/skills/paper-reading-note ~/.claude/skills/
```

Then start a new Claude session.

### 2. Zotero MCP setup

Install:

```bash
uv tool install "git+https://github.com/54yyyu/zotero-mcp.git"
```

Check path:

```bash
export PATH="$HOME/.local/bin:$PATH"
which zotero-mcp
zotero-mcp --help
```

Enable access in Zotero desktop:
- Settings -> Advanced
- Enable allowing local connections from other applications/devices

Register MCP in Claude:

```bash
claude mcp remove zotero
claude mcp add zotero -s user -- $HOME/.local/bin/zotero-mcp serve
claude mcp list
```

If successful, you should see something like:

```text
codex: codex mcp-server - Connected
zotero: $HOME/.local/bin/zotero-mcp serve - Connected
```

---

Upstream repository:
- https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep

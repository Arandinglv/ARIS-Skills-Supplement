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

```bash
# Find recent arXiv papers and append to markdown
/paper-reading-note "latest arxiv on multi-image reasoning benchmark" - append to /path/to/Multi-Image-Benchmark.md

# Read a specific Zotero paper and append to markdown
/paper-reading-note "from zotero: MMDU: A Multi-Turn Multi-Image Dialog Understanding Benchmark and Instruction-Tuning Dataset for LVLMs" - append to /path/to/Multi-Image-Benchmark.md

# Search OpenReview by topic and append to markdown
/paper-reading-note "search openreview: multi-image reasoning benchmark" - append to /path/to/Multi-Image-Benchmark.md
```

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

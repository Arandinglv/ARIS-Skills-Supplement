---
name: paper-reading-note
description: "Search recent arXiv or OpenReview papers by topic/keywords, or read a specific paper from Zotero/local/web, then append a structured reading note to a target Markdown file. Use when user says \"阅读文献并追加到 markdown\", \"latest arxiv papers\", \"search openreview\", \"从 Zotero 读论文\", or wants a reusable literature-note workflow."
argument-hint: [topic-or-paper-title]
allowed-tools: Bash(*), Read, Write, Edit, Grep, Glob, WebSearch, WebFetch, Agent, mcp__zotero__*, mcp__obsidian-vault__*
---

# Paper Reading Note

Read and summarize papers for: **$ARGUMENTS**

## Purpose

This skill supports three primary workflows:

1. **Latest arXiv by topic**
   Search recent arXiv papers in a target area, select the most relevant paper(s), then append structured reading notes to a Markdown file.

2. **OpenReview keyword search**
   Search OpenReview submissions by keywords, select the most relevant paper(s), then append structured reading notes to a Markdown file.

3. **Specific paper from Zotero**
   Locate a paper by exact or fuzzy title in Zotero, read its metadata, annotations, and available content, then append a structured reading note to a Markdown file.

The output format is fixed and optimized for ongoing literature tracking.

## Default Output

- Default target file: user-specified file path
- If the user does not specify a path, ask for one once; if a sensible project file already exists, use that
- Append mode by default, never overwrite unless the user explicitly asks

## Input Modes

Parse `$ARGUMENTS` into one of these modes:

### Mode A: Latest arXiv topic search

Triggered by wording like:

- `latest arxiv on multi-image reasoning benchmark`
- `search latest arxiv: multimodal reasoning`
- `最近 arxiv multi-image reasoning benchmark`

Behavior:

1. Search the web with arXiv-first priority
2. Focus on recent papers, prioritizing the latest 6-12 months unless the user says otherwise
3. Build a short candidate list
4. If the user asked for one paper, choose the single most relevant paper and explain why
5. If the user asked for several, summarize the requested count

### Mode B: OpenReview keyword search

Triggered by wording like:

- `search openreview: multi-image reasoning`
- `latest openreview papers on visual reasoning benchmark`
- `openreview keyword: multimodal benchmark`

Behavior:

1. Search OpenReview first
2. Prefer current or recent conference submissions when the user says `latest`
3. Build a short candidate list ranked by keyword relevance
4. If the user asked for one paper, choose the most relevant paper and explain why
5. If the user asked for several, summarize the requested count

For OpenReview results, prioritize:

- paper title
- abstract
- keywords if visible
- venue / conference / year
- author-provided code links or supplemental project links when available

### Mode C: Specific Zotero paper

Triggered by wording like:

- exact title
- `from zotero: ...`
- `读取 zotero 里的 ...`

Behavior:

1. Search Zotero first
2. If found, use Zotero metadata, annotations, and notes as primary context
3. If Zotero lacks enough content, supplement from local files or web

### Mode D: Specific paper by title or URL

If the user provides a paper title or URL without saying Zotero explicitly:

1. Check Zotero first
2. Then local files
3. Then arXiv / OpenReview if clearly relevant
4. Then web

## Required Output Template

For each paper, append exactly this section structure with Chinese content:

```markdown
## 论文标题
- arXiv url：
- GitHub地址：
- 研究问题：
- 主要贡献：
- 核心方法：
- 技术要点：
- 输入输出实例：
- 数据集：
- 关键实验：
- 局限性：
- 对我有什么启发：
- 好的实验发现和insight：
```

Rules:

- Default to Chinese
- Keep each field concise but specific
- Use `未发现` when a field cannot be verified
- Do not invent GitHub links, dataset names, or numbers
- Distinguish clearly between paper-stated facts and your inference

## Source Priority

Use sources in this order depending on the mode:

1. Zotero MCP
2. Local paper files
3. arXiv abstract / official paper page
4. OpenReview submission page
5. Other web sources such as project page or GitHub repo

For latest-paper discovery, search the web first, then enrich from Zotero if the same paper exists there.

For OpenReview mode, search OpenReview first, then enrich from arXiv, Zotero, GitHub, or project pages.

## Workflow

### Step 1: Resolve the target Markdown path

If the user provided a file path, use it directly.

Example:

`/Users/jiangyutian/OneDrive/5.Project/LEARN/PAPER-READ/Multi-Image-Benchmark.md`

If the file does not exist, create it.

### Step 2: Resolve paper candidates

For **latest arXiv topic**:

1. Search recent arXiv papers on the topic
2. Keep the top 3-5 relevant candidates
3. Prefer benchmark, survey, or dataset papers when the topic wording suggests benchmark discovery
4. If ambiguity remains, present the shortlist before writing

For **OpenReview keyword search**:

1. Search OpenReview with the given keywords
2. Keep the top 3-5 relevant candidates
3. Prefer papers from the latest active/recent venue cycle when the user says `latest`
4. If multiple similarly relevant submissions appear, present a shortlist before writing

For **specific Zotero paper**:

1. Search Zotero by title
2. Match exact title first, fuzzy title second
3. Extract metadata, notes, tags, annotations, and attachments if available

### Step 3: Read and verify

For the selected paper, gather:

- Title
- Problem
- Method
- Experiments
- Contributions
- Limitations
- arXiv URL
- GitHub URL
- Dataset names
- Technical keywords

Preferred evidence:

- abstract
- intro
- method section
- experiment section
- conclusion
- Zotero annotations

For OpenReview papers, also use if available:

- submission abstract
- OpenReview forum metadata
- decision status if visible
- linked arXiv or code repository

### Step 4: Write the note

Append one block per paper to the target Markdown file.

Add a separator after each paper:

```markdown

---

```

### Step 5: Report back

Always tell the user:

1. which paper(s) were appended
2. which file was updated
3. whether the source came from Zotero, arXiv, local file, or web

## Search Guidance

### For latest arXiv mode

- Prefer arXiv and recent official project pages
- Include date awareness; "latest" means current at execution time
- Bias toward papers that directly match the user topic rather than adjacent multimodal work

### For OpenReview mode

- Prefer OpenReview forum pages as the primary source
- When the user says `latest`, prefer the most recent active or recently completed venue cycle
- Record that the paper is an OpenReview submission when it is not yet formally published
- If both OpenReview and arXiv versions exist, include both when useful and avoid duplicating claims

### For Zotero mode

- Zotero annotations and notes are high-value signals
- If the paper exists in Zotero but there is no attachment text available, still use metadata and notes, then supplement from the web

## Failure Handling

- If no Zotero result is found, say so and fall back to web/title search
- If no OpenReview result is found, say so and fall back to arXiv/web search
- If multiple papers have similar titles, stop and present the shortlist
- If a paper is paywalled and only abstract-level info is available, say that explicitly
- Never overwrite the target Markdown unless explicitly requested

## Recommended Examples

```text
/paper-reading-note "latest arxiv on multi-image reasoning benchmark" — append to /Users/jiangyutian/OneDrive/5.Project/LEARN/PAPER-READ/Multi-Image-Benchmark.md

/paper-reading-note "search openreview: multi-image reasoning benchmark" — append to /Users/jiangyutian/OneDrive/5.Project/LEARN/PAPER-READ/Multi-Image-Benchmark.md

/paper-reading-note "latest openreview papers on multi-image reasoning benchmark" — append to /Users/jiangyutian/OneDrive/5.Project/LEARN/PAPER-READ/Multi-Image-Benchmark.md

/paper-reading-note "from zotero: MMDU: A Multi-Turn Multi-Image Dialog Understanding Benchmark and Instruction-Tuning Dataset for LVLMs" — append to /Users/jiangyutian/OneDrive/5.Project/LEARN/PAPER-READ/Multi-Image-Benchmark.md

/paper-reading-note "MMDU: A Multi-Turn Multi-Image Dialog Understanding Benchmark and Instruction-Tuning Dataset for LVLMs" — append to /Users/jiangyutian/OneDrive/5.Project/LEARN/PAPER-READ/Multi-Image-Benchmark.md
```

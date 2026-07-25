# Deep Research

General-purpose deep research skill for AI agents. Decompose any topic into sub-goals, dispatch parallel sub-agents to investigate open web, refine research plan across waves until convergence, return structured citation-dense report inline.

Built for [skills.sh](https://skills.sh) ecosystem — works in Claude Code, Cursor, Copilot, Cline, Windsurf, Codex, Gemini, 30+ other agents.

## What it does

Produces structured Markdown research report covering:

- **TL;DR** — 3–5 bullets
- **Scope and framing**
- **Key findings**
- **Evidence** — organized per sub-goal
- **Counter-evidence and dissent**
- **Gaps and unknowns**
- **Sources** — footnoted, tier-labeled

Reports use Markdown footnote citations (`[^1]`), triangulate material claims across 2+ independent sources.

## Install

```bash
npx skills add ramit-mitra/deep-research-skill
```

## Usage

Invoke skill in AI agent — pass topic inline:

```
/deep-research <topic or question or URL>
```

If invoked without args, skill asks once for topic. Then infer scope, audience, recency from topic, announce framing in one line, kick off research. Single targeted clarifier asked only when topic too ambiguous to frame confidently.

Skill runs parallel sub-agents across multiple research waves, refining plan as evidence accumulates, prints final report inline.

## Key principles

- **Iterative agent loop** — search, read, reason, re-plan, repeat
- **Subagent-driven** — parallel research, not single-agent linear
- **Convergence-stopped** — stops when extra searching adds <15% novel claims for 2 consecutive waves, floor of 10 distinct sources
- **Triangulated** — material claims need 2+ independent sources; single-source claims flagged
- **Source-tier ladder** — primary docs > peer-reviewed > reputable news > industry > expert blogs > community
- **Honest about gaps** — missing or weakly supported claims surfaced, not hidden
- **Refuses harmful topics** — weapons synthesis, CSAM, malware, doxxing

## Source-tier ladder

| Tier | Examples |
|---|---|
| 1. Primary | Official docs, regulatory filings, source code, RFCs, gov data, standards bodies |
| 2. Peer-reviewed | arXiv, journals, conference papers |
| 3. Reputable news | FT, Reuters, NYT, WSJ, Bloomberg, Economist, AP, BBC |
| 4. Industry | Analyst reports, whitepapers, conference talks, eng blogs |
| 5. Expert blogs | Practitioner posts, substacks, newsletters |
| 6. Community | Forums, GitHub issues, HackerNews, Reddit, StackExchange |

Higher tier wins on conflict. Conflicts surfaced in Counter-evidence.

## Inspiration

- [Indian Stock Analyser](https://github.com/ramit-mitra/indian-stock-analyser) — sibling skill demonstrating subagent-driven domain research

## Supported agents

Works with Claude Code, GitHub Copilot, Cursor, Cline, Windsurf, OpenAI Codex, Google Gemini, 30+ other agents via [skills.sh](https://skills.sh) ecosystem.

## License

MIT
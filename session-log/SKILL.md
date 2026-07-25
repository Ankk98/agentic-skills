---
name: session-log
description: >-
  Maintain an append-only greppable .log file for investigation memory across
  long chats (hypotheses, attempts, results, corrections). Use when debugging,
  investigating, researching, deep-diving, or when chat compression / low recall
  risks losing prior findings. Also use when the user asks to session-log,
  investigation-log, or keep a session journal.
---

# Session Log

Append-only investigation memory for the current chat. Purpose: survive compression and stay greppable.

This is best-effort protocol (not a hard guarantee). Prefer short typed entries over prose.

## Paths

| What | Where |
|------|--------|
| Skill | `~/.agents/skills/session-log/` |
| Project logs | `<workspace>/.agents/logs/` |
| Fallback logs | `~/.agents/logs/` (no clear workspace/git root) |

1. Prefer project `.agents/logs/` when a workspace root is clear.
2. Otherwise use `~/.agents/logs/`.
3. On first use in a location: create the logs dir and ensure it is gitignored.

### Gitignore

If the logs dir is under a git repo, ensure ignore (create `.gitignore` or append if missing):

```
.agents/logs/
```

Do not commit log files. Optional: leave no `.gitkeep` unless the user wants the empty dir tracked.

## File naming

Default: one file per chat.

```
YYYY-MM-DD-<slug>-<suffix>.log
```

- `slug`: short topic from the user goal (e.g. `auth-500`, `flake-ci`)
- `suffix`: unique per chat (4+ char random hex or similar) so parallel chats do not collide

Examples: `2026-07-25-auth-500-a3f2.log`

Flexible: if the user names a path/topic or asks to continue an existing log, use that instead.

## Recover active log

If the active path is missing from context:

1. Look under the logs dir for recent `YYYY-MM-DD-*.log` (mtime / date prefix).
2. If one clear match for this chat/topic → resume it.
3. If unsure → ask the user which file (or whether to start a new one).

## When to write

Log when useful for later recall. Typical moments:

- Session start (header)
- Meaningful actions / tool batches
- Hypothesis → attempt → result
- Errors / unexpected findings
- Decisions that change direction
- Session end / handoff
- On demand (“log this”)

Skip when the agent judges the work is trivial and a log would add noise. When in doubt on long debug/research threads, log.

## Append-only

- Never edit or delete prior entries.
- If an assumption or result was wrong, append a new entry that calls it out and references the earlier id (`ref:` / `supersedes:`).

## Secrets

Never log secrets (tokens, passwords, API keys, `.env` values, private keys, PII). Redact as `[REDACTED]`. Prefer command names and exit codes over dumping sensitive stdout.

## Header (session start)

Append once when creating/resuming a file (skip duplicate headers on resume if one already exists):

```
=== SESSION <YYYY-MM-DD> <slug> ===
goal: <one line>
repo: <path or ->
branch: <name or ->
log: <absolute or workspace-relative path>
---
```

Keep header fields minimal; add more only if useful.

## Entry format

Simple, flexible, greppable. One entry:

```
## <ISO-8601> <TYPE> <id>
ref: <optional other id>
<freeform body — keep short>
```

- `TYPE`: e.g. `hypothesis`, `attempt`, `result`, `decision`, `blocker`, `note`, `corrects` (agent may use others)
- `id`: short stable token for this entry (`H1`, `A3`, `R2`, …) so later entries can `ref:` it
- Body: what mattered — hypothesis, what was tried, evidence (path/command), outcome
- Optional keys as needed: `status:`, `supersedes:`, `evidence:`

### Examples

```
## 2026-07-25T12:04:00+05:30 hypothesis H1
Auth middleware drops Authorization on redirect to /api/me

## 2026-07-25T12:12:00+05:30 attempt A1
ref: H1
curl -v localhost:3000/api/me with bearer token; check middleware logs

## 2026-07-25T12:13:10+05:30 result R1
ref: A1
status: rejected
Header present entering middleware; null after rewrite rule. H1 wrong about redirect.

## 2026-07-25T12:14:00+05:30 corrects C1
ref: H1
supersedes: H1
Rewrite strips hop-by-hop headers; fix in next attempt.
```

## Discipline

- Agent chooses verbosity: one-liners for routine steps; more detail for hypotheses/results.
- Prefer grep-friendly literals (error strings, paths, symbol names).
- Do not rewrite history; append corrections.

## Composition

Other skills (research, debug, etc.) may require or assume this protocol. How they invoke it is up to those skills; when this skill is active, keep the log current.

## Quick checklist

```
Session log:
- [ ] Logs dir exists; gitignored if in a repo
- [ ] Active file named or recovered
- [ ] Header present
- [ ] Append on meaningful progress
- [ ] Corrections are new entries (no edits)
- [ ] No secrets
```

---
name: session-summary
description: >-
  Maintain a living one-page markdown summary of the current session (goal,
  current task, task list) in a file on disk, rewritten in place as work
  progresses. Use during long sessions, debugging, or investigations; when chat
  compression / low recall risks losing state; when resuming or handing off
  earlier work; or when the user asks for a session summary, handoff doc, or to
  update or read the summary.
---

# Session Summary

A single markdown file on disk holding the current state of this session: what we are trying to do, what is happening now, and what is left. It is written for two readers — the user skimming progress, and a future agent that has lost the conversation.

The file is a **snapshot, not a journal**. It is rewritten in place so it always describes the present. It is not a transcript and must never grow into one.

This is best-effort protocol (not a hard guarantee).

## Paths

| What | Where |
|------|--------|
| Skill | `~/.agents/skills/session-summary/` |
| Project summaries | `<workspace>/.agents/summaries/` |
| Fallback summaries | `~/.agents/summaries/` (no clear workspace/git root) |

1. Prefer project `.agents/summaries/` when a workspace root is clear.
2. Otherwise use `~/.agents/summaries/`.
3. On first use in a location: create the dir and ensure it is gitignored.

### Gitignore

If the summaries dir is under a git repo, ensure ignore (create `.gitignore` or append if missing):

```
.agents/summaries/
```

Do not commit summary files.

## File naming

One file per chat.

```
YYYY-MM-DD-<slug>-<suffix>.md
```

- `slug`: short topic from the user goal (e.g. `auth-500`, `flake-ci`)
- `suffix`: unique per chat (4+ char random hex or similar) so parallel chats do not collide

Example: `2026-07-25-auth-500-a3f2.md`

Flexible: if the user names a path or topic, or asks to continue an existing summary, use that instead.

## Two modes

**Write** (default) — create or refresh the summary for the current session.

**Resume** — load an existing summary to rehydrate context. Use when the user asks to resume/continue prior work, at the start of a chat that clearly continues earlier work, or after compaction when the active path is missing from context.

### Resume steps

1. Look under the summaries dir for recent `YYYY-MM-DD-*.md` (mtime / date prefix).
2. One clear match for this chat or topic → read it in full and adopt it as the active file.
3. Several plausible matches or none → ask the user which file, or whether to start fresh.
4. Treat the file as **possibly stale**. Verify claims that drive action — branch, file contents, whether a fix is still applied — against the repo before relying on them. Say so when the file and reality disagree.
5. Report to the user in a sentence or two where things stand, then continue the work.

## Length budget

Default target: **~80 lines / ~500 words**. It must stay skimmable in one sitting.

- User says "brief" / "short" → ~30 lines, core sections only.
- User says "detailed" / "long" → up to ~200 lines.
- User names a size ("half a page", "150 lines") → honor it.

When a refresh would exceed the budget, cut rather than append: drop resolved tasks, collapse findings into their conclusions, and delete detail that no longer changes what happens next.

## Template

Core sections are always present. Everything after `## Tasks` is the agent's choice — include only what a cold reader would actually need.

```markdown
# <Topic> — session summary

updated: <ISO-8601>
repo: <path or ->
branch: <name or ->

## Goal
<1-3 lines: what the user wants, in their terms>

## Current task
<what is being worked on right now>
status: <working | broken | blocked | verifying | done>

## Tasks
- [x] <completed>
- [~] <in progress>
- [ ] <pending>

## <agent-chosen section>
<...>
```

### Extras worth including

Pick what earns its space; skip the rest. Common ones:

- **Findings** — what is now known to be true, and the decision it drove
- **Ruled out** — approaches tried that failed, and why, so they are not retried
- **Key files** — paths with line refs (`src/auth/mw.ts:42`) and why each matters
- **Blockers / open questions** — what is unresolved or waiting on the user
- **Commands** — repro, test, or build invocations worth rerunning
- **Notes for the user** — anything needing a human decision

## Tasks section

Mirror the agent's in-chat todo list so the two never disagree.

- `[ ]` pending, `[~]` in progress, `[x]` done
- Keep completed items only while they still give useful context; drop them once stale
- Phrase each item as a concrete action, not a theme

## Refresh at milestones

Rewrite the file when the state it describes has actually changed:

- Session start (create the file once the goal is clear)
- Root cause or key finding established
- Direction change — the plan is now different
- A dead end confirmed, so it is not retried
- Before a risky, slow, or destructive operation
- Context is filling up or compaction looks imminent
- Session end or handoff
- On demand ("update the summary")

Do not refresh after every tool call. Routine progress inside an unchanged plan does not need a rewrite. Skip the file entirely for trivial work where it would only add noise.

## Rewrite discipline

- Rewrite the whole file each time; do not append snapshots or keep a history tail.
- Present state only. Mention the past solely when it changes future actions (a ruled-out approach, a decision's rationale).
- No prose padding. Short lines, concrete nouns, real paths.
- Prefer grep-friendly literals: error strings, file paths, symbol names.
- The previous version is not preserved. Never delete information that is still load-bearing just to hit the length budget — cut resolved and redundant content first.

## Secrets

Never write secrets (tokens, passwords, API keys, `.env` values, private keys, PII). Redact as `[REDACTED]`. Prefer command names and exit codes over pasting sensitive output.

## Example

```markdown
# auth 500s on /api/me — session summary

updated: 2026-07-25T12:40:00+05:30
repo: /home/me/repos/api
branch: fix/auth-500

## Goal
/api/me returns 500 for ~2% of authenticated requests in prod. Find the cause and ship a fix.

## Current task
Confirming the proxy rewrite is what drops the Authorization header.
status: verifying

## Tasks
- [x] Reproduce locally with a bearer token
- [x] Confirm header present entering middleware
- [~] Trace where the header is lost after the rewrite rule
- [ ] Patch the rewrite to preserve Authorization
- [ ] Add regression test

## Findings
- Header is present at `src/auth/mw.ts:42` and null by `src/proxy/rewrite.ts:88`.
- Only requests matching the `/api/*` rewrite fail, which matches the ~2% prod rate.

## Ruled out
- Redirect stripping the header — no redirect occurs on this path.
- Token expiry — failing requests have >20 min of validity left.

## Commands
- Repro: `curl -v -H "Authorization: Bearer $T" localhost:3000/api/me`
- Tests: `npm test -- auth`
```

## Quick checklist

```
Session summary:
- [ ] Summaries dir exists; gitignored if in a repo
- [ ] Active file named, or resumed and verified against repo state
- [ ] Goal / Current task / Tasks present
- [ ] Extras limited to what a cold reader needs
- [ ] Within the length budget; rewritten in place, not appended
- [ ] Tasks mirror the in-chat todo list
- [ ] No secrets
```

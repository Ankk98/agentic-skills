# agentic-skills

Personal [Agent Skills](https://cursor.com/docs) for Cursor and other agents. Source of truth for skills installed under `~/.agents/skills`.

## Skills

| Skill | Description |
|-------|-------------|
| [session-log](./session-log/) | Append-only greppable investigation log across long chats |
| [session-summary](./session-summary/) | Living one-page markdown snapshot of session state, rewritten in place |

## Install (this machine)

Point your agent skills dir at this repo (or symlink individual skills):

```bash
# backup existing skills dir if needed
mv ~/.agents/skills ~/.agents/skills.bak 2>/dev/null || true
mkdir -p ~/.agents/skills

# link each skill from this repo
ln -sfn "$(pwd)/session-log" ~/.agents/skills/session-log
ln -sfn "$(pwd)/session-summary" ~/.agents/skills/session-summary

# optional: Cursor personal skills mirror
mkdir -p ~/.cursor/skills
ln -sfn ~/.agents/skills/session-log ~/.cursor/skills/session-log
ln -sfn ~/.agents/skills/session-summary ~/.cursor/skills/session-summary
```

After linking, edits in this repo are live for the agent.

## Add a skill

1. Create `skill-name/SKILL.md` with YAML frontmatter (`name`, `description`).
2. Symlink it into `~/.agents/skills/` (and optionally `~/.cursor/skills/`).
3. Commit and push.

## Layout

```
agentic-skills/
├── README.md
├── .gitignore
├── session-log/
│   └── SKILL.md
└── session-summary/
    └── SKILL.md
```

# SmartAI

A personal collection of high-leverage Claude Agent Skills.

## Skills

| Skill | What it does |
|---|---|
| [`skill-finder`](skills/skill-finder/) | Find, rank, and compare the best Claude skills on GitHub for a specific need |

## Install a skill into Claude Code

```bash
# Symlink a skill into your global skills directory
ln -s "$(pwd)/skills/skill-finder" ~/.claude/skills/skill-finder
```

Or copy it:

```bash
cp -r skills/skill-finder ~/.claude/skills/skill-finder
```

Then in any Claude Code session, mention the skill by name or trigger phrase (e.g. *"find me the best skill for writing SaaS comparison posts"*).

## Adding new skills

Each skill lives in its own folder under `skills/` with a `SKILL.md` at the root. Follow the [Agent Skills spec](https://agentskills.io/specification).

# Search Strategies

Advanced tactics for finding Claude skills on GitHub that you'd miss with a naive `gh search`.

## The core search command

```bash
gh search repos "<query>" \
  --sort=stars --limit=30 \
  --json fullName,stargazersCount,forksCount,description,url,updatedAt,pushedAt
```

Available sort options: `stars`, `forks`, `updated`, `help-wanted-issues`.

## Triangulation: search 5 angles in parallel

The same skill is often findable under multiple names. A user asking for "SaaS blog post skill" might want a skill tagged as:
- "content marketing"
- "copywriting"
- "blog generation"
- "SEO content"
- "long-form writing"

**Run all of these as parallel Bash calls in one message.** Then dedupe by `fullName` and merge results.

### Example: user wants "landing page skill"

```bash
gh search repos "landing page claude skill" --sort=stars --limit=20 --json fullName,stargazersCount,forksCount,description,url,pushedAt
gh search repos "saas marketing claude skill" --sort=stars --limit=20 --json fullName,stargazersCount,forksCount,description,url,pushedAt
gh search repos "copywriting claude" --sort=stars --limit=20 --json fullName,stargazersCount,forksCount,description,url,pushedAt
gh search repos "frontend design claude skill" --sort=stars --limit=20 --json fullName,stargazersCount,forksCount,description,url,pushedAt
gh search repos "conversion optimization claude" --sort=stars --limit=20 --json fullName,stargazersCount,forksCount,description,url,pushedAt
```

## Mega-pack drill-down

The biggest community repos contain dozens of skills nested inside. They won't appear in a top-level repo search by topic name. Always drill into:

| Repo | Contains |
|---|---|
| `anthropics/skills` | Official skills (~17 production-grade) |
| `alirezarezvani/claude-skills` | 337+ skills across engineering, marketing, product, ops |
| `ComposioHQ/awesome-claude-skills` | Largest curated index |
| `hesreallyhim/awesome-claude-code` | Skills + hooks + slash commands + plugins |
| `Jeffallan/claude-skills` | 66 full-stack dev skills |
| `BehiSecc/awesome-claude-skills` | General curated list |

### Drill-down commands

```bash
# List skills inside a mega-pack
gh api repos/anthropics/skills/contents/skills --jq '.[] | {name,type}'

# Read a specific SKILL.md (head only, to save context)
gh api repos/anthropics/skills/contents/skills/frontend-design/SKILL.md --jq '.content' | base64 -d | head -50

# Get a full skill's file tree
gh api repos/anthropics/skills/contents/skills/frontend-design --jq '.[] | {name,type,size}'
```

### Searching INSIDE a repo for relevant skills

```bash
# Find any skill file in a mega-pack matching a keyword
gh search code "filename:SKILL.md content marketing" --repo=alirezarezvani/claude-skills
gh search code "filename:SKILL.md SaaS" --repo=alirezarezvani/claude-skills
```

## Quality and freshness filters

### Freshness as a tiebreaker

Sort raw results by stars first, then filter/re-sort by `pushedAt`:

```bash
gh search repos "<query>" --sort=stars --limit=50 --json fullName,stargazersCount,pushedAt \
  | jq 'sort_by(.pushedAt) | reverse | .[0:15]'
```

A skill pushed in the last 90 days is more trustworthy than a 50-star skill pushed 18 months ago.

### Language filters

If the user needs an English-language skill, filter out skills whose primary text is non-English:

```bash
# Look at the README for language signal
gh api repos/<owner>/<repo>/readme --jq '.content' | base64 -d | head -20
```

Don't drop non-English skills automatically — the SKILL.md content might still be universal. But flag the language clearly in the comparison.

### Fork-rate as a signal

A repo with a high forks/stars ratio (>15%) is often being adapted by the community — a sign of practical utility. A repo with very low forks but high stars might be aspirational (people bookmark it but don't use it).

```bash
gh search repos "<query>" --sort=stars --limit=30 --json fullName,stargazersCount,forksCount \
  | jq '.[] | . + {forkRatio: (.forksCount / .stargazersCount)}' \
  | jq -s 'sort_by(.forkRatio) | reverse'
```

## Hidden gem discovery

### Recent + good description

Repos created in the last 60 days won't have high stars yet. Find them with:

```bash
gh search repos "<query> claude skill" --sort=updated --limit=20 \
  --json fullName,stargazersCount,description,createdAt,pushedAt
```

Filter for `createdAt` in the last 2 months and a `description` longer than 50 chars (cheap proxy for "the author actually cares").

### Author trust signals

If you find one good skill from an author, search their other repos:

```bash
gh search repos "user:<username>" --json fullName,stargazersCount,description
```

Authors who ship multiple high-quality skills (e.g. Anthropic DevRel, well-known engineers) are higher-trust defaults.

## Non-GitHub sources

When GitHub doesn't surface what you need:

- **agentskills.io** — the official spec site, may link to canonical skills
- **buildwithclaude.com** — community hub for skills/agents/plugins (repo: `davepoon/buildwithclaude`)
- **Claude Code marketplace** (`/plugin marketplace`) — official plugins, some include skills

Use `WebSearch` only when `gh` doesn't surface the right results.

## What to do when nothing fits

If after 5 angles of search nothing rates above 3 stars on the Fit dimension:

1. Tell the user honestly: *"Nothing existing fits well — here's the closest match, and here's what's missing."*
2. Recommend writing a custom skill via Anthropic's `skill-creator`
3. Sketch what the new skill's `description` would look like (the triggering phrase) and what reference files it would need

This is more valuable than recommending a mediocre fit.

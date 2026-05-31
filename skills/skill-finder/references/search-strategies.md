# Search Strategies

The full search playbook behind `SKILL.md`'s 3-tier model. Use as a reference when the tier-1 registries don't cover a need or you want to apply advanced filters.

## Core principle: catalogs before keywords

The Claude skills universe is **finite and indexed**. A handful of registries already filter the noise — hitting them first is faster, cheaper, and broader than searching raw GitHub. Keyword search on GitHub is for *gaps*, not for *first pass*.

```
Tier 1 — Curated registries     ~3 calls, dense signal
   ↓ (only if thin)
Tier 2 — Surgical GitHub search ~2 calls, narrow angles
   ↓
Tier 3 — Deep-read top 3        ~3 calls, judge craft
                                ────────────────
                                ~8 calls total
```

Track candidates in a working set keyed by `owner/repo[/path]` to dedupe across tiers.

## Tier 1 — The registries to know

### Primary three (always run in parallel)

| Registry | URL | Strength |
|---|---|---|
| **agentskills.io** | `https://agentskills.io/llms.txt` (follow linked pages as needed) | Official spec site, LLM-friendly directory format |
| **buildwithclaude (GitHub mirror)** | `https://raw.githubusercontent.com/davepoon/buildwithclaude/main/README.md` | Indexes skills, agents, plugins, marketplace collections across sources. The `buildwithclaude.com` website returns 403 to WebFetch — always use the mirror. |
| **ComposioHQ awesome-list** | `https://raw.githubusercontent.com/ComposioHQ/awesome-claude-skills/master/README.md` | Largest curated catalog by community (62k+ stars). **Default branch is `master`, not `main`.** |

Standard extraction prompt:

> *"List every skill on this page relevant to `<user's need>`. For each: name, one-line description, source link, source repo (owner/repo). Skip unrelated entries. Return as a compact list."*

### Branch-agnostic fallback

If a hardcoded raw URL ever 404s (default branch renamed, repo moved), retry via the readme API endpoint — it auto-resolves the default branch:

```
WebFetch(
  url="https://api.github.com/repos/<owner>/<repo>/readme",
  prompt="Decode the base64 `content` field as markdown and list entries relevant to `<user's need>`."
)
```

This costs one extra LLM-decode step but is immune to branch drift. Use it as the second attempt, not the first.

### Failure handling

If a Tier 1 source returns 4xx/5xx, drop it from the working set and continue with what succeeded. Don't retry the same URL. If all three fail, escalate to Tier 2 immediately rather than thrashing.

### Secondary registries (use as fallback, or to fill gaps)

| Registry | URL | When to use |
|---|---|---|
| **hesreallyhim/awesome-claude-code** | `https://raw.githubusercontent.com/hesreallyhim/awesome-claude-code/main/README.md` | When user needs skills + hooks + slash commands + plugins, not just skills |
| **travisvn/awesome-claude-skills** | `https://raw.githubusercontent.com/travisvn/awesome-claude-skills/main/README.md` | Heavy focus on Claude Code, often surfaces dev/eng skills |
| **BehiSecc/awesome-claude-skills** | `https://raw.githubusercontent.com/BehiSecc/awesome-claude-skills/main/README.md` | General-purpose curated index |
| **anthropics/skills README** | `https://raw.githubusercontent.com/anthropics/skills/main/README.md` | When official / first-party is preferred |
| **skills.sh** | `https://skills.sh/` | Cross-registry installer/index, can surface non-GitHub-hosted skills |
| **Claude Code marketplace** | Run `/plugin marketplace` in Claude Code | Curated plugins, some bundle skills |

### Mega-pack repos (single repos containing many skills)

Treat these as registries — one fetch lists dozens of nested skills:

| Repo | Stars | Lean |
|---|---|---|
| `anthropics/skills` | 144k | Official, production-grade |
| `alirezarezvani/claude-skills` | 17k | 337+ skills across eng / marketing / product / ops |
| `Jeffallan/claude-skills` | 9.5k | Full-stack dev |
| `mohitagw15856/pm-claude-skills` | 916 | Product / PM / business |
| `bergside/awesome-design-skills` | 1k | Design / UX |

Fetch a mega-pack's skill folder when its theme overlaps the user's need:

```
WebFetch(
  url="https://api.github.com/repos/<owner>/<repo>/contents/skills",
  prompt="Return entries with name matching `<user's need>` (semantic match, not exact substring). Include name + type."
)
```

## Tier 2 — When registries miss, search GitHub surgically

Only do this if Tier 1 returns fewer than 3 strong candidates. **Two angles, not five.** Token cost matters.

### The two angles

1. **Direct phrasing** — the user's task verbatim with `claude skill` appended
2. **Adjacent phrasing** — the nearest domain term with `claude skill` (e.g. user says "compare SaaS products" → adjacent is "content marketing" or "comparison page")

```
WebFetch(
  url="https://api.github.com/search/repositories?q=<query>+claude+skill&sort=stars&order=desc&per_page=15",
  prompt="Return JSON: full_name, stargazers_count, forks_count, description, html_url, pushed_at, default_branch. Sort by stargazers_count descending."
)
```

URL-encode spaces as `+`. `per_page=15` is intentional — the long tail of GitHub keyword search rarely beats curated picks. Don't go higher unless the first 15 are noise.

### Rate limit

GitHub REST API: **60 requests/hour unauth**, per IP. The full skill-finder budget is ~8 calls, so you can comfortably run 5–7 sessions per hour.

### Sort options

`sort=stars` is the default choice. Other useful sorts:
- `sort=updated` — find recently-active skills (hidden gems, see below)
- `sort=forks` — high-fork repos signal community adoption beyond bookmarking

## Tier 3 — Judge craft on the top 3

After triage and rubric scoring, fetch the actual SKILL.md for the top 3 to score the Quality dimension honestly:

```
WebFetch(
  url="https://raw.githubusercontent.com/<owner>/<repo>/<default_branch>/<path>/SKILL.md",
  prompt="Return the YAML frontmatter verbatim and the first 60 lines of the body."
)
```

If `main` returns 404, retry with `master`. Or query the repo metadata once to be safe:

```
WebFetch(url="https://api.github.com/repos/<owner>/<repo>", prompt="Return default_branch only.")
```

What you're looking for in the SKILL.md:
- A specific `description` with concrete trigger phrases (not "helps with X")
- Imperative voice in the body
- Worked examples
- Reference files (progressive disclosure done right)
- Scripts (signals production-readiness)
- Evals (rare and high-signal)

## Advanced tactics

### Freshness as a tiebreaker

After registry/search results, sort by `pushed_at` descending. A 50-star skill pushed 14 days ago often outperforms a 500-star skill pushed 18 months ago. Prompt the extractor to surface freshness:

> *"After listing top results, also return the 5 with the most recent pushed_at."*

### Fork-rate as a usage signal

`forks_count / stargazers_count` above 0.15 means people are actually adapting the skill, not just bookmarking it. Below 0.02 on a high-star repo signals aspirational bookmarks.

> *"For each repo, compute fork_ratio = forks_count / stargazers_count. Flag repos with fork_ratio > 0.15."*

### Hidden gems (recent + cared-about)

For skills created in the last 60 days:

```
WebFetch(
  url="https://api.github.com/search/repositories?q=<query>+claude+skill&sort=updated&order=desc&per_page=15",
  prompt="Return repos with created_at in the last 60 days and description length > 50 chars. Include full_name, description, stargazers_count, created_at."
)
```

A long, specific description on a young repo is a cheap proxy for "the author cares."

### Author trust signals

If you find one strong skill from an author, check their other repos:

```
WebFetch(
  url="https://api.github.com/search/repositories?q=user:<username>+claude&sort=stars",
  prompt="Return full_name, stargazers_count, description."
)
```

Authors with 3+ high-star skill repos (Anthropic, prolific DevRel, well-known engineers) are higher-trust defaults.

### Language filters

If the user works in English, check the README for primary language:

```
WebFetch(
  url="https://raw.githubusercontent.com/<owner>/<repo>/<default_branch>/README.md",
  prompt="Return the primary language and the first 15 lines."
)
```

Don't auto-drop non-English skills — SKILL.md logic is often universal — but flag the language clearly in the comparison.

## Optional accelerator: `gh` CLI

If `gh` CLI is installed and authenticated (`brew install gh && gh auth login`), substitute it for `WebFetch` to get auth'd queries with a 5,000/hr rate limit. Same REST API underneath, same JSON shape.

| WebFetch URL | `gh` equivalent |
|---|---|
| `https://api.github.com/search/repositories?q=X&sort=stars` | `gh search repos "X" --sort=stars --json fullName,stargazersCount,forksCount,description,url,pushedAt` |
| `https://api.github.com/repos/X/Y/contents/path` | `gh api repos/X/Y/contents/path --jq '.[] \| {name,type}'` |
| `https://raw.githubusercontent.com/X/Y/main/path/SKILL.md` | `gh api repos/X/Y/contents/path/SKILL.md --jq '.content' \| base64 -d` |

Check availability with `command -v gh && gh auth status`. If either fails, stay on `WebFetch`. Never ask the user to install `gh` — the default path works.

## When nothing fits

If after Tier 1 + Tier 2 nothing scores ≥3 on Fit:

1. Tell the user honestly: *"Nothing existing fits — closest is X, missing Y."*
2. Recommend writing custom via Anthropic's `skill-creator`
3. Sketch the `description` and reference-file structure the new skill would need

This is more valuable than recommending a mediocre fit.

# Search Strategies

Advanced tactics for finding Claude skills on GitHub that you'd miss with a naive single query.

## Tooling

Primary: **`WebFetch` against the GitHub REST API** (built into Claude Code, no install, no auth).

The REST API endpoints you'll use:

| Purpose | URL |
|---|---|
| Search repos | `https://api.github.com/search/repositories?q=<query>&sort=stars&order=desc&per_page=30` |
| List repo contents | `https://api.github.com/repos/<owner>/<repo>/contents/<path>` |
| Get repo metadata | `https://api.github.com/repos/<owner>/<repo>` |
| Read raw file | `https://raw.githubusercontent.com/<owner>/<repo>/<branch>/<path>` |

**Rate limit (unauthenticated):** 60 requests/hour per IP. A typical skill-finder run uses ~8 calls. You can comfortably run 5–7 sessions per hour.

**Sort options for search:** `stars`, `forks`, `updated`, `help-wanted-issues`. Default is best-match — pass `&sort=stars` explicitly.

## Triangulation: search 5 angles in parallel

The same skill is often findable under multiple names. A user asking for "SaaS blog post skill" might want a skill tagged as:
- "content marketing"
- "copywriting"
- "blog generation"
- "SEO content"
- "long-form writing"

**Run all of these as parallel `WebFetch` calls in one message.** Then dedupe by `full_name` and merge results.

### Example: user wants "landing page skill"

Five parallel `WebFetch` calls, one per angle:

```
WebFetch(url="https://api.github.com/search/repositories?q=landing+page+claude+skill&sort=stars&per_page=20", prompt="...")
WebFetch(url="https://api.github.com/search/repositories?q=saas+marketing+claude+skill&sort=stars&per_page=20", prompt="...")
WebFetch(url="https://api.github.com/search/repositories?q=copywriting+claude&sort=stars&per_page=20", prompt="...")
WebFetch(url="https://api.github.com/search/repositories?q=frontend+design+claude+skill&sort=stars&per_page=20", prompt="...")
WebFetch(url="https://api.github.com/search/repositories?q=conversion+optimization+claude&sort=stars&per_page=20", prompt="...")
```

Standard extraction prompt:
> *"Return the top results as JSON. For each repo: full_name, stargazers_count, forks_count, description, html_url, pushed_at, default_branch. Sort by stargazers_count descending."*

URL-encoding: replace spaces with `+`. For other special characters, use `%XX` encoding.

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

### Drill-down

```
# List skills inside a mega-pack
WebFetch(
  url="https://api.github.com/repos/anthropics/skills/contents/skills",
  prompt="Return each entry as {name, type}."
)

# Read a specific SKILL.md (raw URL — no base64 to decode)
WebFetch(
  url="https://raw.githubusercontent.com/anthropics/skills/main/skills/frontend-design/SKILL.md",
  prompt="Return the YAML frontmatter and the first 60 lines of the body."
)

# Get full file tree for one skill
WebFetch(
  url="https://api.github.com/repos/anthropics/skills/contents/skills/frontend-design",
  prompt="Return each entry as {name, type, size}."
)
```

If `main` returns 404 on a raw URL, retry with `master`. You can also read `default_branch` from the repo metadata endpoint to be safe:

```
WebFetch(url="https://api.github.com/repos/<owner>/<repo>", prompt="Return default_branch only.")
```

## Quality and freshness filters

### Freshness as a tiebreaker

The search API returns `pushed_at` per repo. After getting raw results, sort by `pushed_at` descending and keep the freshest 15:

> *In your extraction prompt:* "After listing the top 30 by stars, return the 15 with the most recent pushed_at."

A skill pushed in the last 90 days is more trustworthy than a 50-star skill pushed 18 months ago.

### Language filters

If the user needs an English-language skill, check the README for language signal:

```
WebFetch(
  url="https://api.github.com/repos/<owner>/<repo>/readme",
  prompt="The content field is base64-encoded. Decode it and return the first 20 lines. Note the primary language."
)
```

Don't drop non-English skills automatically — the SKILL.md logic might still be universal. But flag the language clearly in the comparison.

### Fork-rate as a signal

A repo with a high forks/stars ratio (>15%) is often being adapted by the community — a sign of practical utility. A repo with very low forks but high stars might be aspirational (people bookmark it but don't use it).

When the search results come back, ask for fork ratio in the extraction prompt:

> *"For each repo, also compute forks_count / stargazers_count as fork_ratio. Sort by fork_ratio descending after the star sort."*

## Hidden gem discovery

### Recent + good description

Repos created in the last 60 days won't have high stars yet. Find them with `sort=updated`:

```
WebFetch(
  url="https://api.github.com/search/repositories?q=<query>+claude+skill&sort=updated&order=desc&per_page=20",
  prompt="Return repos with created_at in the last 60 days and a description longer than 50 characters. Include full_name, description, stargazers_count, created_at, pushed_at."
)
```

A long description is a cheap proxy for "the author actually cares."

### Author trust signals

If you find one good skill from an author, search their other repos:

```
WebFetch(
  url="https://api.github.com/search/repositories?q=user:<username>+claude&sort=stars",
  prompt="Return full_name, stargazers_count, description for each."
)
```

Authors who ship multiple high-quality skill repos (e.g. Anthropic, well-known DevRel) are higher-trust defaults.

## Non-GitHub sources

When GitHub doesn't surface what you need:

- **agentskills.io** — the official spec site, may link to canonical skills
- **buildwithclaude.com** — community hub for skills/agents/plugins (repo: `davepoon/buildwithclaude`)
- **Claude Code marketplace** (`/plugin marketplace`) — official plugins, some include skills

Use `WebSearch` only when the GitHub API doesn't surface the right results.

## Optional accelerator: `gh` CLI

If the `gh` CLI is installed and authenticated (`brew install gh && gh auth login`), you can substitute it for `WebFetch` to get faster, auth'd queries with a 5,000/hr rate limit. The data shape is identical (same REST API underneath, just shell-formatted).

Equivalents:

| WebFetch | `gh` equivalent |
|---|---|
| `https://api.github.com/search/repositories?q=X&sort=stars` | `gh search repos "X" --sort=stars --json fullName,stargazersCount,forksCount,description,url,pushedAt` |
| `https://api.github.com/repos/X/Y/contents/path` | `gh api repos/X/Y/contents/path --jq '.[] \| {name,type}'` |
| `https://raw.githubusercontent.com/X/Y/main/path/SKILL.md` | `gh api repos/X/Y/contents/path/SKILL.md --jq '.content' \| base64 -d` |

Check availability with `command -v gh` and that auth works with `gh auth status`. If either fails, stay on the `WebFetch` path. Don't ask the user to install `gh` — the default path works fine.

## What to do when nothing fits

If after 5 angles of search nothing rates above 3 stars on the Fit dimension:

1. Tell the user honestly: *"Nothing existing fits well — here's the closest match, and here's what's missing."*
2. Recommend writing a custom skill via Anthropic's `skill-creator`
3. Sketch what the new skill's `description` would look like (the triggering phrase) and what reference files it would need

This is more valuable than recommending a mediocre fit.

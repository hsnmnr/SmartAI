---
name: skill-finder
description: Find, rank, and compare the best Claude Agent Skills on GitHub for a specific user need. Use whenever the user asks "what skill should I use for X", "find me a skill for X", "is there a skill that does X", "compare skills for X", "best skill for [content / SEO / landing pages / blog posts / SaaS / product / marketing / sales / code review / docs / anything]", or mentions wanting to discover, install, or evaluate Claude skills, plugins, or agents from the community. Trigger even when the user only hints at a need without saying the word "skill" — e.g. "I want to write SaaS comparison blog posts" or "help me build a landing page" — because there is almost certainly an existing community skill that does it better than starting from scratch.
---

# Skill Finder

A meta-skill that turns a fuzzy user need into a ranked, compared shortlist of the best Claude Agent Skills on GitHub.

## When to use this skill

Activate whenever the user wants to **discover, evaluate, or pick between** Claude skills for a task. Examples:

- *"Find me a skill for writing SaaS landing page copy"*
- *"What's the best skill for SEO blog posts?"*
- *"Compare skills for code review"*
- *"I need to write a comparison page — any good skills?"*
- *"Should I write my own skill or is there one already?"*

If the user asks you to *write* a skill from scratch, that's a different job — use Anthropic's `skill-creator` instead. This skill is for *finding* existing ones.

## The 5-step loop

Run these in order. Do not skip step 1 or step 4.

1. **Clarify the need** — ask 1–3 sharp questions (only if the request is genuinely ambiguous)
2. **Search** — query GitHub across multiple angles
3. **Triage** — drop the noise, keep ~8–12 real candidates
4. **Score** — rank against the rubric (see `references/evaluation-rubric.md`)
5. **Present** — ranked comparison + a verdict (see `references/output-template.md`)

---

## Step 1 — Clarify the need

Before searching, get clarity on what the user actually wants. **Skip this step if the request is already specific** (e.g. *"find the best SEO audit skill for Next.js sites"* — that's clear enough, just search).

Ask **at most 3 questions**, only the ones that genuinely change the recommendation. Common gaps:

- **Domain**: SaaS marketing, e-commerce, dev tools, B2B sales, content, design, code?
- **Output type**: long-form article, landing page, copy snippet, audit report, executable code?
- **Stack/format constraints**: Next.js / React / WordPress / plain Markdown / Figma / Notion?
- **Workflow**: one-shot generation vs. multi-step with research vs. ongoing/recurring?
- **Already tried**: is there a skill they've used that almost worked but missed something? (huge signal)

Use the `AskUserQuestion` tool when there are 2–4 mutually exclusive paths. For free-form clarification, ask directly in prose.

**Don't over-ask.** If the user says *"skill for SaaS blog comparison posts"*, just search. You can always refine after the first results.

---

## Step 2 — Search

Use `WebFetch` against the GitHub REST API as the primary search engine. It's universal (no install, no auth, works on any machine), and returns clean JSON. Fall back to `WebSearch` only for non-GitHub sources.

### Primary search call

```
WebFetch(
  url="https://api.github.com/search/repositories?q=<query>+claude+skill&sort=stars&order=desc&per_page=30",
  prompt="Return the top results as a compact list. For each repo include: full_name, stargazers_count, forks_count, description, html_url, pushed_at, default_branch. Sort by stargazers_count descending."
)
```

URL-encode spaces in the query as `+`. The API is unauthenticated, public, and rate-limited to 60 requests per hour — a full skill-finder run uses ~8 calls, so the limit is not a practical concern for a single session.

### Search across multiple angles in parallel

Run **3–5 queries in the same message** (parallel `WebFetch` calls) to triangulate. The same skill may be findable under many names. Cover:

1. **Direct task phrasing**: `<task>+claude+skill` — e.g. `SaaS+blog+claude+skill`
2. **Domain phrasing**: `<domain>+claude+skill` — e.g. `content+marketing+claude+skill`
3. **Output phrasing**: `<output+type>+claude+skill` — e.g. `landing+page+claude+skill`
4. **Adjacent phrasing**: `<adjacent+term>+claude` — e.g. `copywriting+claude`, `SEO+claude`
5. **Awesome-list scan**: `awesome+claude+skills` — these are aggregators; the highest-starred ones index hundreds of skills you'd otherwise miss

### Don't forget the mega-packs

The largest community repos (`anthropics/skills`, `alirezarezvani/claude-skills`, `ComposioHQ/awesome-claude-skills`, `hesreallyhim/awesome-claude-code`) often contain dozens of skills that won't show up by name in a top-level repo search because they're nested inside. After the broad search, **drill into the top 2–3 awesome-lists and skill-packs** with:

```
# List a folder's contents
WebFetch(
  url="https://api.github.com/repos/<owner>/<repo>/contents/skills",
  prompt="Return each entry as name + type."
)

# Read a specific SKILL.md (uses the raw content URL — no base64 decoding needed)
WebFetch(
  url="https://raw.githubusercontent.com/<owner>/<repo>/<default_branch>/<path>/SKILL.md",
  prompt="Return the YAML frontmatter and the first 60 lines of the body."
)
```

Use the `default_branch` from the search results (usually `main`, sometimes `master`).

See `references/search-strategies.md` for advanced tactics (date filters, language filters, fork-rate as a signal, non-English skills, optional `gh` CLI accelerator).

---

## Step 3 — Triage

You'll typically get 50+ raw hits. Cut to ~8–12 real candidates before scoring. Drop:

- **Forks of the same upstream** (keep the canonical one — usually the most-starred)
- **Stale repos** (no `pushedAt` in 12+ months *and* under 50 stars — exception: official Anthropic repos)
- **Off-topic matches** (the word "claude" appears but it's not actually a skill — e.g. someone named Claude)
- **Non-skills** (MCP servers, agents, plugins that aren't packaged as a `SKILL.md`) — *unless* the user's need is broader than just skills, in which case keep them and label them clearly
- **Language mismatches** if the user is working in English (e.g. Chinese-only skills with no English README) — but keep them if the underlying skill logic is universal

What to **keep** even if low-starred:
- Recently pushed (< 90 days) and well-described
- From a known author (Anthropic, well-known DevRel, or someone with multiple high-star skill repos)
- Has actual `SKILL.md` files in the repo (verifiable real skill, not vapor)

---

## Step 4 — Score against the rubric

For each surviving candidate, score on the rubric in `references/evaluation-rubric.md`. The five dimensions:

| Dimension | Weight | What you're measuring |
|---|---|---|
| **Fit** | ★★★★★ | How precisely it solves the user's stated need (not adjacent — the actual thing) |
| **Quality** | ★★★★ | SKILL.md craft: clear triggers, examples, references, scripts, evals |
| **Popularity** | ★★★ | Stars + forks as a social-proof signal (not absolute truth) |
| **Maintenance** | ★★★ | Last push date, open issues, responsive author |
| **Install friction** | ★★ | Standalone skill vs. requires plugin marketplace vs. requires API keys/MCP |

A **5-star fit on a low-star repo beats a 3-star fit on a famous one.** Don't just rank by stars.

**Read the actual SKILL.md** for the top 3 candidates before scoring quality. Stars don't tell you whether the prompt is well-written. Use:

```
WebFetch(
  url="https://raw.githubusercontent.com/<owner>/<repo>/<default_branch>/<path-to-SKILL.md>",
  prompt="Return the full file content verbatim."
)
```

---

## Step 5 — Present the comparison

Output format (see `references/output-template.md` for the full template):

1. **One-sentence verdict** at the top — *"For SaaS comparison blog posts, install `<repo>` first; if you need [specific gap], add `<repo2>` alongside it."*
2. **Ranked table** — 3–5 candidates, columns: Rank · Skill · Stars · Last push · Fit · One-line power
3. **Per-skill mini-card** for the top 3:
   - **What it does** (1–2 sentences)
   - **Why it's good** (the 2–3 things it nails)
   - **Cons / gaps** (be honest — every skill has tradeoffs)
   - **Best for** (specific use case match)
   - **Install** (exact command)
4. **Honorable mentions** — 1-line nod to anything notable that didn't make the top 3
5. **Verdict + next step** — your pick and *why*, plus one suggested action

### Honesty rules

- **Always include cons.** A comparison with no cons is a sales pitch, not a recommendation.
- **Flag uncertainty.** If you couldn't read the SKILL.md or the repo is too new to judge quality, say so.
- **Distinguish "best for this user" from "most popular".** They're often not the same.
- **Don't pad the list.** If there are only 2 real options, present 2. Don't invent a 3rd.
- **Call out when to write your own.** If nothing fits well, recommend `skill-creator` and sketch what the skill would need.

---

## Examples

### Example 1 — Specific need, no clarification needed

> **User:** "Find me a skill for writing SaaS comparison blog posts"

You: *Skip clarification (request is specific). Run 4 parallel `WebFetch` calls against `https://api.github.com/search/repositories?q=...` for `saas+blog+claude+skill`, `comparison+page+claude+skill`, `content+marketing+claude+skill`, and drill into `alirezarezvani/claude-skills` + the official `anthropics/skills` via the contents endpoint. Triage to ~10 candidates. Read top-3 SKILL.md files via `raw.githubusercontent.com`. Score. Output ranked table + verdict.*

### Example 2 — Vague need, ask one question

> **User:** "I want a skill for landing pages"

You: *Ask one clarifying question via AskUserQuestion: "What's the primary goal — copy/messaging, full page build (HTML/React), or conversion optimization for an existing page?" Then proceed with focused search.*

### Example 3 — User already tried something

> **User:** "I tried `copywriting` from the global skills but it doesn't get SaaS positioning right — what else is there?"

You: *Skip clarification (the "almost worked but missed X" signal is gold). Search specifically for SaaS-positioning skills + B2B copywriting skills. In the verdict, explicitly compare against the skill the user already tried.*

---

## Guidelines

- **Run searches in parallel.** Multiple `WebFetch` calls in the same message — never sequentially.
- **Read SKILL.md files before recommending.** Stars are a triage signal, not a quality signal.
- **Keep the output scannable.** Tables and short cards beat prose paragraphs. The user is choosing between options, not reading an essay.
- **Cite the repo URL for every skill mentioned.** Always link directly.
- **No emojis in output** unless the user requests them.
- **End with one clear next action** — install command, or "want me to dig deeper on #2?"

## Reference files

Load these on demand:

- [`references/search-strategies.md`](references/search-strategies.md) — advanced GitHub search tactics, date/language filters, fork analysis, hidden gem discovery
- [`references/evaluation-rubric.md`](references/evaluation-rubric.md) — the full scoring rubric with per-dimension criteria and worked examples
- [`references/output-template.md`](references/output-template.md) — the exact output format with a fill-in-the-blank template

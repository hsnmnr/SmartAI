# Evaluation Rubric

How to score a candidate skill on 5 dimensions. Use this for the top 3 finalists, not every candidate.

## The 5 dimensions

| # | Dimension | Weight | Score range |
|---|---|---|---|
| 1 | Fit | ★★★★★ (highest) | 1–5 |
| 2 | Quality | ★★★★ | 1–5 |
| 3 | Popularity | ★★★ | 1–5 |
| 4 | Maintenance | ★★★ | 1–5 |
| 5 | Install friction | ★★ (lowest) | 1–5 |

**Weighted total** = `(Fit × 5) + (Quality × 4) + (Popularity × 3) + (Maintenance × 3) + (Install × 2)` → max 85.

---

## 1. Fit — does it actually solve the user's stated need?

This is the most important dimension. A 5/5 fit on a 200-star skill beats a 2/5 fit on a 5000-star skill.

| Score | Criteria |
|---|---|
| 5 | Solves the exact need, in the exact format the user wants. The description practically reads like the user's question. |
| 4 | Solves the need but with one mismatch (e.g. output format slightly off, requires light adaptation) |
| 3 | Adjacent — does something nearby, will require workflow changes to fit |
| 2 | Tangentially related — could be useful but mostly off-topic |
| 1 | Wrong category, only matched on a keyword |

**Read the SKILL.md** (not just the README) to judge this. The triggering description in the frontmatter is the most honest summary of what the skill actually does.

---

## 2. Quality — is the skill itself well-written?

A great fit poorly executed will frustrate the user. Score on these signals:

| Signal | Worth |
|---|---|
| SKILL.md has a clear, specific `description` with trigger phrases | +1 |
| SKILL.md uses imperative voice, examples, edge cases | +1 |
| Has `references/` or `scripts/` (progressive disclosure done right) | +1 |
| Has `evals/` or test cases (rare, high signal) | +1 |
| README has install instructions and a worked example | +1 |

Sum the signals → score 1–5.

| Score | What it looks like |
|---|---|
| 5 | All 5 signals. Production-grade. (e.g. official `anthropics/skills`) |
| 4 | 4 signals. Clearly well-crafted. |
| 3 | 3 signals. Usable but rough. |
| 2 | 2 signals. Bare minimum. |
| 1 | 1 or 0 signals. Mostly empty or broken. |

---

## 3. Popularity — social proof

Stars + forks tell you the community has validated this. Not a quality proxy, but a useful filter.

| Score | Stars (approx) | Notes |
|---|---|---|
| 5 | 5,000+ | Top-tier visibility |
| 4 | 1,000–5,000 | Well-known |
| 3 | 200–1,000 | Solid traction |
| 2 | 50–200 | Niche but real |
| 1 | < 50 | Brand new or unknown |

**Adjustment:** add +1 if forks/stars ratio > 0.15 (high practical adoption). Subtract -1 if stars > 1,000 but forks < 30 (aspirational bookmark, not used).

---

## 4. Maintenance — is it alive?

| Score | Last push | Open issues |
|---|---|---|
| 5 | < 30 days | Few or actively closing |
| 4 | < 90 days | Some, author responsive |
| 3 | < 180 days | Backlogged but alive |
| 2 | < 365 days | Slow |
| 1 | > 365 days | Likely abandoned |

**Exception:** Anthropic-official repos are evergreen — score them 4+ even if not pushed recently.

---

## 5. Install friction — how fast can the user actually use it?

| Score | What it takes to install |
|---|---|
| 5 | Single SKILL.md, drop into `~/.claude/skills/`, done |
| 4 | Standalone skill folder, `cp` or symlink, done |
| 3 | Inside a marketplace plugin, requires `/plugin install` |
| 2 | Requires API keys, env vars, or MCP server setup |
| 1 | Requires paid service, custom infra, or external dependencies |

---

## Worked example

**User:** "Find me a skill for SaaS comparison blog posts"

**Candidate A:** `acme/comparison-blog-skill` — 380 stars, last push 14 days ago, full SKILL.md with examples + references for SEO + B2B SaaS positioning, standalone install.
- Fit: 5/5 (exact match)
- Quality: 4/5 (good SKILL.md, no evals)
- Popularity: 3/5 (380 stars)
- Maintenance: 5/5 (recent)
- Install: 5/5 (drop-in)
- **Weighted: 25 + 16 + 9 + 15 + 10 = 75/85**

**Candidate B:** `bigname/awesome-content-skills` — 8,200 stars, last push 6 months ago, mega-pack containing a `comparison-post` subfolder with a short SKILL.md and no references.
- Fit: 4/5 (good match, less specific)
- Quality: 2/5 (sparse SKILL.md, no examples)
- Popularity: 5/5 (8.2k stars)
- Maintenance: 3/5 (6 months)
- Install: 3/5 (needs plugin marketplace)
- **Weighted: 20 + 8 + 15 + 9 + 6 = 58/85**

**Verdict:** Recommend A despite B's higher stars. Stars don't equal fit.

---

## When to skip the rubric

Don't run the full rubric on more than 3 candidates — it's slow and the user won't read 5 scorecards. For #4 and #5, a 1-line "honorable mention" is enough.

If only 2 candidates exist, present 2. Don't pad.

If no candidate clears Fit ≥ 3, recommend writing a custom skill via `skill-creator` instead.

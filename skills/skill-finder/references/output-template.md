# Output Template

The exact format for presenting comparison results. Optimize for scannability — the user is choosing between options, not reading an essay.

## Top-level structure

```
1. Verdict (1 sentence)
2. Comparison table (3–5 rows)
3. Top 3 mini-cards (per-skill detail)
4. Honorable mentions (1 line each)
5. Next step (1 sentence)
```

---

## 1. Verdict — lead with the answer

One sentence at the very top. Tell the user what to install. Don't bury it.

> **Verdict:** For SaaS comparison blog posts, install [`acme/comparison-blog-skill`](https://github.com/acme/comparison-blog-skill) — it's the only candidate purpose-built for B2B SaaS positioning. Pair with [`bigname/seo-skill`](https://github.com/bigname/seo-skill) if you also want keyword-driven structure.

---

## 2. Comparison table

5 columns, max 5 rows. Keep cells short.

```markdown
| Rank | Skill | ★ | Last push | Fit | Power in one line |
|------|-------|---|-----------|-----|-------------------|
| 1 | [acme/comparison-blog-skill](url) | 380 | 14d | 5/5 | Purpose-built for B2B SaaS comparison posts with positioning frameworks |
| 2 | [bigname/content-pack](url) | 8.2k | 6mo | 4/5 | Generic content pack, has a comparison-post subfolder |
| 3 | [neat/copywriting](url) | 1.2k | 45d | 3/5 | Strong on copy craft but not SaaS-specific |
```

**Always link the repo.** Always show stars + last push as a freshness signal.

---

## 3. Top 3 mini-cards

For each of the top 3, write a 5-line card:

```markdown
### 1. acme/comparison-blog-skill — ★380

**What it does** — Generates B2B SaaS comparison blog posts using positioning frameworks (Crossing the Chasm, JTBD), with built-in SEO structure and CTA placement.

**Why it's good** — Purpose-built for SaaS (not generic content). Ships with 12 worked examples across categories (CRM, analytics, dev tools). Author shipped 4 other high-quality SaaS skills.

**Cons / gaps** — No support for product screenshots or visual comparison tables. Only English. Last release 14 days ago — still settling, expect minor changes.

**Best for** — Founders writing "X vs Y" or "Best X for Y" posts where SEO + positioning both matter.

**Install** — `ln -s "$(pwd)/<path>" ~/.claude/skills/comparison-blog-skill` or `cp -r` it.
```

### Rules for mini-cards

- **Pros and cons both required.** A comparison without cons is a sales pitch.
- **Be specific in "best for"** — name the exact use case match.
- **"Why it's good" = 2–3 concrete things**, not adjectives. "Strong examples" is weak. "Ships with 12 worked examples across CRM/analytics/dev tools" is strong.
- **"Cons" = real gaps**, not nitpicks. If you genuinely can't find a con, say "no significant gaps for this use case" — but be skeptical of yourself.
- **Install command must be runnable** — don't say "install via plugin marketplace", show the exact command.

---

## 4. Honorable mentions

One line each, max 3 entries.

```markdown
**Honorable mentions:**
- [`other/skill-a`](url) (★240) — Strong on SEO research but doesn't write the post itself
- [`other/skill-b`](url) (★1.1k) — Good for blog posts in general but no SaaS-specific frameworks
```

Skip this section if there's nothing worth mentioning.

---

## 5. Next step

End with one clear action.

> **Next step:** Want me to install `acme/comparison-blog-skill` into your global skills directory, or dig deeper on the SEO pairing with `bigname/seo-skill`?

Options:
- Offer to install the top pick
- Offer to dig deeper on a runner-up
- Ask if the user wants you to draft a custom skill instead (if no candidate scored well)

---

## Honesty rules (recap)

1. **Always include cons.** Even for the #1 pick.
2. **Flag uncertainty.** "Couldn't read the SKILL.md — repo is private / 404 / parse error" beats fake confidence.
3. **Stars ≠ best.** Rank by fit, not popularity.
4. **Don't pad.** 2 real options > 5 padded options.
5. **Recommend writing custom** when nothing fits.

---

## Full worked example

```markdown
**Verdict:** For writing SaaS landing page copy, install [`anthropics/skills/frontend-design`](https://github.com/anthropics/skills/tree/main/skills/frontend-design) as the foundation and pair with [`acme/saas-copy`](https://github.com/acme/saas-copy) for positioning. Nothing in the ecosystem does both well in a single skill.

| Rank | Skill | ★ | Last push | Fit | Power |
|---|---|---|---|---|---|
| 1 | [anthropics/skills/frontend-design](url) | 144k | 21d | 5/5 | Official, production-grade page generation with design tokens |
| 2 | [acme/saas-copy](url) | 920 | 30d | 4/5 | B2B SaaS messaging frameworks + value-prop builders |
| 3 | [neat/landing-pages](url) | 2.1k | 4mo | 3/5 | Generic landing pages, weak on SaaS-specific copy |

### 1. anthropics/skills/frontend-design — ★144k (parent repo)
**What it does** — Generates production-grade frontend (React/Tailwind/shadcn) with design tokens and avoids generic AI aesthetics.
**Why it's good** — Official Anthropic skill. Trained on real design systems. Has reference files for component patterns.
**Cons / gaps** — Engineering-quality output, but doesn't write *positioning* — just renders what you give it.
**Best for** — Building the page once you have the message.
**Install** — `/plugin install example-skills@anthropic-agent-skills` in Claude Code

### 2. acme/saas-copy — ★920
**What it does** — Generates SaaS hero/feature/CTA copy using JTBD and Crossing-the-Chasm frameworks.
**Why it's good** — Built specifically for B2B SaaS. Ships with 30+ category-specific examples (CRM, analytics, dev tools, fintech).
**Cons / gaps** — Copy only — no HTML/React output. You'll generate copy here, then feed it to frontend-design.
**Best for** — Drafting headline, sub, value props, CTAs before building the page.
**Install** — `ln -s "$(pwd)/skills/acme-saas-copy" ~/.claude/skills/saas-copy`

### 3. neat/landing-pages — ★2.1k
**What it does** — General landing page generator with Tailwind templates.
**Why it's good** — Has a wide template library across industries.
**Cons / gaps** — No SaaS-specific frameworks. Copy quality is generic ("Build faster. Scale smarter."). Last push 4 months ago.
**Best for** — Quick prototypes for non-SaaS use cases.
**Install** — `cp -r <path> ~/.claude/skills/landing-pages`

**Honorable mentions:**
- [`bergside/awesome-design-skills`](url) (★1k) — Curated list of 67 design skills, browse if you need something specific
- [`ui-styling`](your-global) — You already have this installed; pairs well with frontend-design

**Next step:** Want me to install `acme/saas-copy` and walk you through pairing it with `frontend-design`?
```

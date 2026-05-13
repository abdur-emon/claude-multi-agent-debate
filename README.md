# Multi-Agent Debate

> A Claude Code skill that argues with itself **20 ways** before answering, so you don't mistake a confident guess for a vetted decision.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-blue.svg)](https://claude.com/claude-code)
[![Status](https://img.shields.io/badge/status-stable-green.svg)]()

A single LLM pass picks the most probable answer and commits to it — the dissenting views that would have sharpened the recommendation get quietly dropped. That's fine for *"what's the syntax for X."* It's dangerous for *"should we migrate to Postgres?"*, *"is this architecture going to hold?"*, *"should I take this job?"*

This skill convenes a fixed panel of **20 specialized AI personas**, runs them through a 4-round structured debate in parallel, and returns a **ranked, critique-hardened recommendation** with the strongest counter-argument named explicitly.

---

## ⚠️ Cost warning — read first

| | Tokens / run | Notes |
|---|---|---|
| **Naive baseline** | ~200k | Without any optimizations |
| **This skill** | **~125–135k** | After 5 built-in optimizations |
| **Subagent calls** | ~83 | 80 fan out in parallel, 3 sequential |

Approximate dollar cost per debate (depends on which model your subagents run on):

| Model | Per debate |
|---|---|
| Sonnet 4.6 | **~$0.85** |
| Opus 4.7 | **~$4.50** |

This skill is **deliberately expensive**. Use it only when being wrong costs more than the tokens do.

---

## Quick start

### 1. Install

```bash
git clone https://github.com/abdur-emon/claude-multi-agent-debate.git ~/.claude/skills/multi-agent-debate
```

Restart Claude Code (or reload skills) and the skill becomes invokable.

### 2. Invoke

Explicitly:
```
/multi-agent-debate Should we migrate our monolith to microservices?
```

Or implicitly — the skill triggers on phrasing like:
- *"Debate whether we should adopt Rust for the backend"*
- *"Stress-test this plan: ship the redesign on Q3"*
- *"Convene a panel on hiring vs. outsourcing the ML team"*
- *"Give me multiple perspectives on this architecture"*

### 3. Read the report

You'll get a ~400–500 word beginner-friendly report. See [Output format](#output-format) below.

---

## When to use

✅ Hard decisions where reasonable experts would disagree
✅ Plans that need to survive contact with their strongest attacks
✅ Strategy / architecture / hiring / product calls with real stakes
✅ Research questions where one perspective would be too narrow

## When NOT to use

❌ Factual lookups
❌ Mechanical code edits
❌ Well-specified tasks with one correct answer
❌ Anything where you can't afford ~130k tokens on a single response

**If in doubt, don't invoke this.** The skill itself flags trivial questions as "overkill" and declines to run.

---

## The panel (20 seats, fixed)

### 🛠️ Tech specialists (12 — 60%)

| # | Persona | Axis they own |
|---|---|---|
| 1 | Staff Software Engineer | Code quality, maintainability, long-term debt |
| 2 | Security Engineer / Red Teamer | Adversarial threat model, trust boundaries |
| 3 | Site Reliability Engineer | Operational reality, SLOs, failure modes |
| 4 | Distributed Systems Architect | Scale, state, consistency, partitions |
| 5 | Frontend / UX Engineer | User-facing reality, perceived quality |
| 6 | ML / AI Researcher | Model behavior, evaluation, distribution shift |
| 7 | Data Scientist / Analyst | Quantitative rigor, measurement validity |
| 8 | **CTO** *(CEO-level #1)* | Engineering sustainability, team health |
| 9 | **Tech CEO** *(CEO-level #2)* | Market capture, runway, competitive bets |
| 10 | **R&D Officer — Feasibility** | *Can we actually build this?* |
| 11 | **R&D Officer — Market Demand** | *Will anyone pay for this?* |
| 12 | **R&D Officer — Portfolio Fit** | *Is this the right bet vs. everything else?* |

### 🧠 Generalist critics (8 — 40%)

Skeptic · Pragmatist · Contrarian · Synthesizer · Risk Analyst · Devil's Advocate · Ethicist · Historian/Futurist

### Why this composition

- **Both CEO voices always seated.** The CTO and Tech CEO are seated in **deliberate tension** — engineering sustainability vs. market capture gets debated, not hand-waved.
- **All three R&D Officers always seated.** Feasibility, demand, and portfolio fit form a "should we even start this?" bloc before the rest of the panel argues *how* to execute.
- **Orthogonality.** Each seat owns a distinct axis — no two personas make the same argument from slightly different angles.

A reserve bench of 6 alternate tech personas (Backend, DevOps, Database, Founder-Engineer, OSS Maintainer, QA) gets swapped in when the question demands. See [`references/personas.md`](references/personas.md) for the full roster and swap rules.

---

## How it works — the 9-step protocol

| Step | Name | Type |
|------|------|------|
| 1 | Scope the question | Orchestrator |
| 2 | Cast the 20-agent panel | Orchestrator |
| 3 | **Round 1** — Opening statements | Parallel (20 calls) |
| 4 | **M3** — Semantic twin detection & merge | 1 cheap-model call |
| 5 | **M5** — Steelmanned claim card compression | 1 cheap-model call |
| 6 | **Round 2** — Two-phase critique + coverage check | Parallel (20 + 20 calls) |
| 7 | **Round 3** — Diff-only rebuttals | Parallel (20 calls) |
| 8 | **Judge** — Ranked synthesis + top-5 findings | 1 call |
| 9 | **Present** — Beginner-friendly report | Orchestrator |

Every fan-out round fires its 20 agents in a **single message** (strict parallelism — no sequential anchoring drift). Three single-call steps (M3, M5, Judge) run sequentially.

---

## Built-in optimizations

| Optimization | Savings | Accuracy impact |
|---|---|---|
| **M2** — prompt caching on invariant prefixes | ~20–25% | None (pure billing rebate) |
| **M3** — semantic twin merge | ~12–18% when fires | **Better** than baseline |
| **M5** — steelmanned claim cards + two-phase Round 2 | ~15% of total | –0.3% (near zero) |
| **Diff rebuttals** — Round 3 output compression | –40% on R3 only | None |
| **Creativity bounds** — strict word caps on novelty | ~2% | **Better** (less noise) |

**Net: ~35–40% below the naive baseline, with ~0.5–1% accuracy improvement.**

---

## Output format

A ~400–500 word beginner-friendly report (up to ~600 if the topic genuinely needs it) with these sections:

- **Context** — plain-English topic primer so a non-expert can follow
- **Top 5 findings** — ranked takeaways from the actual debate (not generic facts)
- **Recommendation** — what to do, in directive voice
- **Why this wins** — supporting reasons
- **Strongest counter-argument** — the sharpest attack, attributed to a specific persona
- **Confidence** — High / Medium / Low + reason
- **Panel** — list of all 20 personas that deliberated

Jargon is defined inline. The report stands on its own for a reader with zero domain knowledge.

---

## Safety features

- **Strict parallel execution** on every fan-out round — no sequential anchoring drift between agents.
- **Web-search outbound protection** — agents never leak the user's question, company names, codenames, credentials, or panel-internal deliberations to search providers. All queries are abstracted before leaving.
- **Creativity bounds** — one creative angle per agent per round, in-character only, no fabricated facts.
- **Twin-merge strict match** — only HIGH-confidence semantic twins fire; false positives cost creativity, false negatives only cost a little token budget.
- **Coverage check** — every opening gets at least one critique; no position sneaks through unchallenged.

---

## Repo contents

```
multi-agent-debate/
├── SKILL.md              # Full orchestration protocol, 9 steps, every prompt template
├── README.md             # This file
├── LICENSE               # MIT
└── references/
    └── personas.md       # 26-persona roster + canonical panel + swap rules
```

---

## Credits

Originally authored by [**mdismail-cse**](https://github.com/mdismail-cse) — see the upstream repo at [mdismail-cse/multi-agent-debate-skill](https://github.com/mdismail-cse/multi-agent-debate-skill). This fork is maintained by [**abdur-emon**](https://github.com/abdur-emon).

## License

Released under the [MIT License](LICENSE).

# Multi-Agent Debate Skill

A Claude skill that orchestrates a **20-agent parallel debate panel** to handle hard decisions, contested research questions, and plans that need to be stress-tested. It produces a ranked, critique-hardened recommendation with a beginner-friendly report.

> **⚠️ WARNING: This skill can consume a HUGE amount of tokens per run. Use with caution.**
>
> A full debate typically burns **~125–135k tokens** (after all built-in optimizations). The naive version would cost ~200k. Do not invoke casually — only use for questions that genuinely benefit from 20 perspectives debating each other.

---

## What it does

Convenes a fixed panel of 20 specialized AI personas — 12 tech specialists and 8 generalist critics — and runs them through a 4-round structured debate:

1. **Opening statements** (parallel, independent)
2. **Cross-examination** (parallel critiques)
3. **Rebuttal & refinement** (parallel diffs)
4. **Ranked synthesis** (neutral judge picks winner)

The final output is a clear, beginner-friendly report with the top 5 findings, the recommendation, the strongest counter-argument, and a confidence level.

---

## Panel composition (60% tech / 40% generalist)

**12 tech specialists:**
- Staff Software Engineer
- Security Engineer / Red Teamer
- Site Reliability Engineer (SRE)
- Distributed Systems Architect
- Frontend / UX Engineer
- ML / AI Researcher
- Data Scientist / Analyst
- **CTO / Engineering Leader** (CEO-level #1)
- **Tech CEO / Company Builder** (CEO-level #2)
- **R&D Officer — Feasibility** (can we build it?)
- **R&D Officer — Market Demand** (will anyone pay for it?)
- **R&D Officer — Portfolio Fit** (is this the right bet?)

**8 generalist critics:**
Skeptic · Pragmatist · Contrarian · Synthesizer · Risk Analyst · Devil's Advocate · Ethicist · Historian/Futurist

Two CEO-level voices are always seated in deliberate tension (engineering vs. business). All three R&D Officers always evaluate any proposal before the rest of the panel debates execution.

---

## 9-step workflow

| Step | Name | Type |
|------|------|------|
| 1 | Scope the question | Orchestrator |
| 2 | Cast the canonical 20-agent panel | Orchestrator |
| 3 | **Round 1** — Opening statements | Parallel (20 calls) |
| 4 | **M3** — Semantic twin detection & merge | 1 cheap-model call |
| 5 | **M5** — Steelmanned claim card compression | 1 cheap-model call |
| 6 | **Round 2** — Two-phase critique + coverage check | Parallel (20 + 20 calls) |
| 7 | **Round 3** — Diff-only rebuttals | Parallel (20 calls) |
| 8 | **Judge** — Ranked synthesis + top-5 beginner findings | 1 call |
| 9 | **Present** — Clean beginner-friendly report | Orchestrator |

Total: ~83 subagent calls per debate.

---

## Token usage

> **⚠️ USE WITH CAUTION — this skill consumes significant tokens per debate run.**

A typical debate uses **~125–135k tokens** after all 5 optimizations. The naive version would cost ~200k.

Built-in optimizations:

| Optimization | Savings | Accuracy impact |
|---|---|---|
| **M2** — prompt caching on invariant prefixes | ~20–25% | None (pure billing rebate) |
| **M3** — semantic twin merge | ~12–18% when fires | **Better** than baseline |
| **M5** — steelmanned claim cards + two-phase Round 2 | ~15% of total | –0.3% (near zero) |
| **Diff rebuttals** — Round 3 output compression | –40% on R3 only | None |
| **Creativity bounds** — strict word caps on novelty | ~2% | **Better** (less noise) |

Net: **~35–40% below the naive baseline**, with **~0.5–1% accuracy improvement** (M3 catches more real twins; creativity bounds cut noise).

---

## When NOT to use this skill

- Simple factual lookups
- Well-specified tasks with one correct answer
- Mechanical code edits
- Quick questions a single answer can handle
- Anything where you can't afford ~130k tokens on a single response

**If in doubt, don't invoke this.** The skill itself will flag trivial questions as "overkill" and decline to run the full protocol.

---

## Output format

The user receives a short, beginner-friendly report (~400–500 words) with these sections:

- **Context** — plain-English topic primer so a non-expert can follow
- **Top 5 findings** — ranked takeaways from the actual debate (not generic facts)
- **Recommendation** — what to do, in directive voice
- **Why this wins** — supporting reasons
- **Strongest counter-argument** — the sharpest attack, attributed to a persona
- **Confidence** — High / Medium / Low + reason
- **Panel** — list of all 20 personas that deliberated

Jargon is defined inline. The report stands on its own for a reader with zero domain knowledge.

---

## Safety features

- **Parallel execution** guaranteed on every fan-out round — no sequential anchoring.
- **Web-search outbound protection** — agents never leak the user's question, company names, codenames, credentials, or panel-internal deliberations to search providers. All queries are abstracted before leaving.
- **Creativity bounds** — one creative angle per agent per round, in-character only, no fabricated facts.
- **Twin-merge strict match** — only HIGH-confidence semantic twins fire; false positives cost creativity, false negatives cost only a little token budget.
- **Coverage check** — every opening gets at least one critique; no position sneaks through unchallenged.

---

## Files

- `SKILL.md` — full orchestration protocol, all 9 steps, every prompt template.
- `references/personas.md` — 26-persona roster, canonical 20-agent panel, and "when to swap" rules for domain-specific debates.

---

## Installation

Place the `multi-agent-debate/` directory into your Claude skills folder:

```
~/.claude/skills/multi-agent-debate/
```

Restart Claude Code (or reload skills) and the skill becomes invokable.

---

## License

Private. Do not redistribute without permission.

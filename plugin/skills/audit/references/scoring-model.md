# Scoring Model

## Tiers

| Tier | Name | Weight | Items |
|------|------|--------|-------|
| T1 | Foundation | 50% | CLAUDE.md existence, test command, build command, project overview |
| T2 | Protection | 30% | Sensitive file protection, security rules, hook configuration quality |
| T3 | Optimization | 20% | Directory references, CLAUDE.md length, command availability, rules path validation, agent configuration quality, MCP configuration |

## Item Scoring (4-Level Scale)

| Result | Score | When to use |
|--------|-------|-------------|
| PASS | 1.0 | Fully implemented — meets all criteria |
| PARTIAL | 0.6 | Exists but incomplete (e.g., `.env` in deny but `secrets/` missing) |
| MINIMAL | 0.3 | Bare minimum present (e.g., CLAUDE.md exists but no test command or project overview) |
| FAIL | 0.0 | Missing entirely |
| SKIP | — | Not applicable — remove from denominator |

## Formula

```
Tier_Score = Σ(item_score) / count(non-SKIP items in tier)     # Range: 0.0 – 1.0

Final_Score = (T1_Score × 0.50 + T2_Score × 0.30 + T3_Score × 0.20) × 100
```

If an entire tier has all items SKIP, that tier's weight is redistributed proportionally to the remaining tiers.

## Quality Gate

The Quality Gate is a binary pass/fail check independent of the score. ALL conditions must be met:

- CLAUDE.md exists (root or `.claude/CLAUDE.md`)
- Test command is present in CLAUDE.md

If the Quality Gate fails, display **"Gate: NOT READY"** alongside the score. The score is still calculated but the gate signals that the configuration is not functional.

## Penalty Cap

When a T1 (Foundation) item scores FAIL, the final score is capped regardless of other tier scores:

| Condition | Score Cap |
|-----------|-----------|
| CLAUDE.md missing | Max 20 |
| Test command missing | Max 50 |
| 2+ T1 items FAIL | Max 30 |

Apply the lowest applicable cap. Penalty caps prevent inflated scores when fundamentals are missing.

## Grade

| Grade | Score Range |
|-------|------------|
| A | ≥ 80 |
| B | ≥ 60 |
| C | ≥ 40 |
| D | ≥ 20 |
| F | < 20 |

## Maturity Level

Maturity levels are cumulative — each level requires the previous level to be satisfied:

| Level | Condition | Meaning |
|-------|-----------|---------|
| Level 1 — Basic | T1_Score ≥ 0.7 | Claude can work effectively |
| Level 2 — Protected | Level 1 AND T2_Score ≥ 0.6 | Project is safe from common mistakes |
| Level 3 — Optimized | Level 2 AND T3_Score ≥ 0.5 | Configuration is well-organized and maintainable |

If Level 1 is not met, display **"Level 0 — Incomplete"**.

## Output Format

```
Configuration Audit Results
═══════════════════════════

Quality Gate: PASS ✓  (CLAUDE.md ✓, test command ✓)

Foundation (T1):    0.85    █████████████████░░░ 85%
Protection (T2):    0.70    ██████████████░░░░░░ 70%
Optimization (T3):  0.50    ██████████░░░░░░░░░░ 50%

Score: 74/100 (Grade: B)  |  Maturity: Level 2 — Protected

  Breakdown:
  T1: 0.85 × 50% = 42.5
  T2: 0.70 × 30% = 21.0
  T3: 0.50 × 20% = 10.0
  Penalty: none

[Detailed findings per item...]

Suggestions
  • [actionable improvements]

Since last audit (2026-03-31): 65 → 74 (+9). Resolved: missing security rules.
Still open: no MCP configuration, agent model diversity.
```

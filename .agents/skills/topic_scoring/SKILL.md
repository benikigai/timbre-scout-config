---
name: topic_scoring
description: Score a candidate on novelty and voice fit, then combine. Outputs three numbers in [0,1] that map directly onto the Candidate schema.
---

# Topic scoring

For each candidate that survives dedupe, compute three scores:

- `novelty_score` ∈ `[0, 1]`
- `voice_fit_score` ∈ `[0, 1]`
- `combined_score = 0.55 * voice_fit_score + 0.45 * novelty_score` (rounded to 3 decimals)

**Determinism rule:** the scoring path is rule-based. Do not call the model to derive scores — use the rubrics below mechanically. Same input → same scores.

## `novelty_score`

Start from the highest matching tier; stop at the first that applies.

| Condition | Score |
|---|---|
| Canonical URL is in `seen.txt` (shouldn't reach scoring, but defensive) | `0.00` |
| Title or summary fuzzy-matches an entry seen in the last 7 days (e.g. same release announced by 3 outlets) | `0.30` |
| Topic is adjacent to (but not identical to) something covered in `voice_corpus/` or `seen.txt` in the last 14 days | `0.70` |
| Topic is absent from `voice_corpus/` AND from `seen.txt` for the last 7 days | `1.00` |

Subtract `0.10` (floor at 0) if the title uses stock hype phrasing: "revolutionize", "game-changing", "the death of X", "X is eating Y". (See `voice_dna.json` → `forbidden_jargon` for the canonical list.)

## `voice_fit_score`

Additive, then capped to `[0, 1]`. Consult the `voice_profile` skill output first.

| Condition | Add |
|---|---|
| Candidate's topic plausibly carries one of the profile's `tone` tags (e.g. `engineering-first`, `pragmatic`, `slightly-skeptical`) | `+0.40` |
| Candidate intersects a topical anchor extracted from `voice_corpus/*.md` (a domain term that recurs in the founder's past posts) | `+0.30` |
| Candidate's likely technical depth aligns with `technical_depth` (`deep-engineer` candidates want code/benchmarks/mechanism; mismatch → no bonus) | `+0.30` |
| Title or summary contains a term from `forbidden_jargon` | `-0.20` |

After additive scoring, apply the **forbidden-moves caps** from `voice_profile` (generic AI hype / press-release re-tread / outsider topic). These are post-additive clamps, not additive penalties — a candidate stuck in any of those buckets is capped at `0.30` or `0.20` regardless of its raw additive score.

If `voice_corpus/` is empty, the corpus-anchor bullet contributes 0 (don't fabricate anchors). Voice_fit then leans on tone + technical_depth alone.

## `combined_score`

```
combined_score = round(0.55 * voice_fit_score + 0.45 * novelty_score, 3)
```

## Output (per candidate)

Write these directly into the `Candidate` object in `candidates.json`:

```json
{
  "novelty_score": 0.700,
  "voice_fit_score": 0.700,
  "combined_score": 0.700,
  "summary": "<one-sentence why this matters; <=280 chars>"
}
```

(The other Candidate fields — `id`, `url`, `title`, `source`, `published_at`, `raw_excerpt` — come from `source_scanning`, not from this skill.)

## Thresholds

- `combined_score > 0.85` → **alert.** Append to `alerts.json` per `AGENTS.md` §"Alert schema". Use the `summary` (or a tightened version) as the alert's `reason`.
- `combined_score ≥ 0.30` → keep in top 50 (subject to ranking).
- `combined_score < 0.30` → drop from `candidates.json`. Still append the canonical URL to `seen.txt` so it doesn't re-evaluate.

Never alert on `combined_score ≤ 0.85`. Alert precision matters more than recall — the founder's trust in Scout decays fast on false positives.

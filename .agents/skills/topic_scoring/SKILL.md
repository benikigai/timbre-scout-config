---
name: topic_scoring
description: Score a candidate topic on novelty, relevance, and voice fit. Output a number in [0, 1].
---

# Topic scoring

For each candidate that survives dedupe, produce a single score `[0, 1]` that summarizes whether the founder should care.

## Components

The score is the weighted product of three sub-scores. Each is in `[0, 1]`.

### `novelty` (weight: 0.3)

Is this story actually new, or has the topic been covered to death? Use:

- Was the same claim/finding/release covered by any source in `seen.txt` in the last 7 days? If yes, novelty ≤ 0.3.
- Does the title use stock phrases ("X is revolutionizing Y", "the death of Z")? Penalize.
- Does the story include a specific technical claim, dataset, benchmark number, or code reference? Boost.

### `relevance` (weight: 0.4)

Does this match the founder's domain? Consult `voice_profile` for the founder's topical surface area. Score relevance against:

- Topics the founder has written about before (from `voice_corpus/`)
- Adjacent topics that intersect their stated focus
- Stories where they would have a contrarian or insider take

A story that's *interesting in general* but outside the founder's wheelhouse should score relevance < 0.4 even if it's important news.

### `voice_fit` (weight: 0.3)

Could the founder write about this in their voice? Apply the `voice_profile` skill. Discount candidates that:

- Are pure punditry on macro tech trends (low fit — founder writes technically)
- Are press release re-treads (low fit — founder writes original takes)
- Require knowledge the founder demonstrably doesn't have (low fit)

Boost candidates that:

- Connect to something the founder already has a strong opinion about
- Have a technical angle the founder is positioned to explain better than the source

## Output

```json
{
  "score": 0.78,
  "novelty": 0.85,
  "relevance": 0.80,
  "voice_fit": 0.70,
  "reasoning": "One sentence on why this scored where it did."
}
```

Persist this into the entry's `score` field in `candidates.json`. Keep `reasoning` short — it's for human spot-checks during demos.

## Thresholds

- `score > 0.85` → **alert.** Append to `alerts.json`.
- `score > 0.60` → keep in top 50 ranking.
- `score < 0.40` → still record in `seen.txt`, but exclude from `candidates.json`.

Never alert on `score ≤ 0.85`. The founder's trust in Scout depends on alert precision.

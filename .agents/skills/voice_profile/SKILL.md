---
name: voice_profile
description: Load the founder's Voice DNA (tone, jargon, openings, depth) and expose it to topic_scoring + downstream Writer/Voice stages.
---

# Voice profile

This skill is the founder's voice, loaded once per tick and consulted by `topic_scoring`. Downstream Timbre stages (Writer, Voice) load the same data via the orchestrator, not via this skill.

## Sources, in order of authority

1. **`/workspace/voice_dna.json`** — authoritative if present. Matches the `VoiceProfile` schema (`founder_id`, `tone[]`, `sentence_length`, `technical_depth`, `forbidden_jargon[]`, `preferred_openings[]`, `brand{}`, `tts_voice`).
2. **`/workspace/voice_corpus/*.md`** — used as evidence for topical signals (what does the founder actually write about). NOT used to overwrite `voice_dna.json`.
3. If `voice_dna.json` is absent (shouldn't happen — it ships with the repo), reconstruct a best-effort profile from `voice_corpus/`, write it to `voice_dna.json`, and proceed.

Never include PII beyond `founder_id`. Never modify `voice_dna.json` if it already exists — it's version-controlled config.

## Fields you must use

| Field | Used by | Used how |
|---|---|---|
| `tone[]` | `topic_scoring` → voice_fit | If a candidate's topic plausibly carries one of these tones (e.g. "engineering-first" → a deep technical post fits; "pragmatic" → a benchmark or postmortem fits), award the tone match. |
| `forbidden_jargon[]` | `topic_scoring` → both novelty and voice_fit | Penalize candidates whose title/summary contains these terms. The founder doesn't write hype, so hype titles signal source quality. |
| `technical_depth` | `topic_scoring` → voice_fit | Candidate must plausibly admit a post at this depth (`deep-engineer` = needs technical specifics; benchmark numbers, code, mechanism). |
| `preferred_openings` | downstream Writer/Voice (informational here) | Not used during scouting. Listed for context. |

## Topical surface — derived from corpus, not declared

There is no `topical_surface_area` field in the Voice DNA schema. To answer "what does the founder write about," do this once per tick:

1. List `/workspace/voice_corpus/*.md`. If empty, treat topical surface as undefined → voice_fit relies on `tone` + `technical_depth` only.
2. If non-empty, inspect 2–3 files (rotate across ticks via `tick_id % n` so you cover the corpus over time).
3. Extract the dominant nouns / domain terms (e.g. "Postgres replication", "vector store eviction", "Kubernetes pod scheduling"). These become the implicit topical surface for this tick.
4. Use them as evidence when scoring: if a candidate intersects an extracted topic, that's a `tone` match for voice_fit purposes.

This is qualitative, not numeric. Don't emit a separate score for it — fold the judgment into `voice_fit_score` per `topic_scoring/SKILL.md`.

## Forbidden moves (auto-discount)

These are independent of corpus and always apply:

- **Generic AI hype.** "Model X scores Y on Z" with no implementation angle → cap `voice_fit_score` at `0.30`.
- **Press release re-treads.** "Company announces feature" with no analysis hook → cap `voice_fit_score` at `0.30`.
- **Outsider commentary on fields the founder demonstrably doesn't work in** (e.g. cell biology, macro finance — judged via corpus absence) → cap `voice_fit_score` at `0.20`.

The cap applies AFTER additive scoring in `topic_scoring`. So a candidate that would otherwise be 0.90 voice_fit gets clamped to 0.30 if it's pure AI hype.

## Output

This skill doesn't have a separate output — its job is to make the Voice DNA + corpus signals available to `topic_scoring`. After running this skill once per tick, every subsequent `topic_scoring` call has access to:

- The parsed `voice_dna.json` object.
- A small list of topical anchors extracted from corpus (or empty if no corpus).

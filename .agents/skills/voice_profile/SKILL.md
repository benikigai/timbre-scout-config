---
name: voice_profile
description: The founder's voice, topical surface area, and forbidden moves
---

# Voice profile

This skill is consulted by `topic_scoring` (for relevance + voice_fit) and by the downstream Writer / Voice agents in the Timbre pipeline.

Source of truth: the markdown files in `voice_corpus/`. Treat the rules below as a learned summary; if a candidate seems to contradict these rules but matches what the founder actually writes (per `voice_corpus/`), trust the corpus.

## Topical surface area

> _**TODO** (founder to fill in): 5–10 topics the founder writes about. Be specific. "Infrastructure" is too broad; "self-hosted multi-region Postgres" is specific._

## Voice rules

> _**TODO** (founder to fill in or auto-extract from corpus). Examples:_
> - _Never use "leverage" as a verb._
> - _Sentences are short. Paragraphs are 2–4 sentences._
> - _Code examples are mandatory for any technical claim._
> - _No "in today's fast-paced world" openers. No "in conclusion" closers._
> - _Personal anecdotes are fine if they're specific and short._

## Forbidden moves

- **Generic AI takes.** If a story is generic AI hype ("Model X scored Y on benchmark Z"), only score it relevant if the founder can add a take that adds value beyond restating the benchmark.
- **Press release reposts.** The founder doesn't repost — they react. A topic must admit a take, not just a summary.
- **Outsider commentary.** If the founder doesn't have credible standing on a topic (e.g. they're not in the field), discount voice_fit hard.

## How to extract voice from corpus

When `voice_corpus/*.md` exists, Scout should treat each file as a positive example of the voice. For scoring:

1. Inspect 2–3 corpus posts per tick (rotating, to vary exposure).
2. Identify recurring patterns: openers, transition styles, sentence rhythm, technical specificity.
3. Use those as the implicit rubric for `voice_fit` scoring.

Do not produce a single numeric "voice match" score — the comparison is qualitative. Reason about it in `reasoning` in the topic_scoring output.

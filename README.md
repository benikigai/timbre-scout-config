# timbre-scout-config

> **Status:** Companion config repo for [Timbre](https://github.com/benikigai/timbre) — Cerebral Valley × Google DeepMind I/O Hackathon, 2026-05-23. Reference only. It must stay a standalone public repo because the Antigravity managed agent clones it by URL.

Configuration for **Timbre Scout** — the always-on Antigravity agent that monitors the founder's industry, scores candidate topics against a voice profile, and persists state to its sandbox.

This repo is cloned into Scout's Antigravity environment on every tick. The runtime state (`candidates.json`, `seen.txt`, `alerts.json`) is written by the agent and not committed here.

## Layout

```
.
├── AGENTS.md                          # Scout's role + tick protocol (loaded as system instructions)
├── .agents/
│   └── skills/
│       ├── source_scanning/SKILL.md   # how to fetch + dedupe feeds
│       ├── topic_scoring/SKILL.md     # novelty / relevance rubric
│       └── voice_profile/SKILL.md     # founder's voice rules
├── sources.yaml                       # RSS, X handles, HN tags, arXiv categories
└── voice_corpus/                      # founder's past blog posts as markdown
    └── *.md
```

## Wiring

The orchestrator passes this repo as a `repository` source to `agents.create()`:

```ts
base_environment: {
  type: "remote",
  sources: [{
    type: "repository",
    source: "https://github.com/benikigai/timbre-scout-config",
    target: "/workspace"
  }]
}
```

Scout is invoked hourly by an external scheduler (Vercel Cron). Each tick:
1. Clone latest config (this repo)
2. Read `sources.yaml`, fetch new candidates, dedupe against `seen.txt`
3. Score candidates against `voice_corpus/` + skill rubrics
4. Persist top results to `candidates.json`, alerts to `alerts.json`
5. Exit; the orchestrator harvests state via subsequent interactions

## Related

- [benikigai/timbre](https://github.com/benikigai/timbre) — the pipeline, backend and dashboard that register and drive Scout.
- Landing page: [usetimbre.ai](https://usetimbre.ai) (the live API backend is offline; see the Timbre README).

## Credits

Built by [Ben Shyong](https://github.com/benikigai). `voice_corpus/` holds Ben's own published posts, used as the voice reference.

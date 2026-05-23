---
name: source_scanning
description: Fetch, parse, and dedupe entries from the sources listed in sources.yaml
---

# Source scanning

The `sources.yaml` file lists feeds Scout monitors. Each entry has a `type` and source-specific config. This skill defines how to fetch each type.

## Source types

### `rss`

Standard RSS or Atom feed. Fetch via HTTP, parse with a standard library. Extract: `title`, `link`, `pubDate`, `description`, `author`.

```yaml
- type: rss
  name: simonwillison
  url: https://simonwillison.net/atom/everything/
```

### `hn_tag`

Hacker News stories matching a tag or score threshold. Use the Algolia HN API:

```
https://hn.algolia.com/api/v1/search_by_date?tags=story&numericFilters=points>50
```

Filter by `created_at_i` to last 24h.

### `x_user`

X (Twitter) user timeline. Currently no free API access — Scout should treat these as *manual reminders* unless the orchestrator pre-populates a feed. Skip silently if unavailable.

### `arxiv`

arXiv category RSS:

```
https://arxiv.org/rss/cs.AI
```

Parse like RSS. The `description` field contains the abstract — use it for scoring.

## Dedupe

Canonicalize URLs before checking `seen.txt`:
- Strip `utm_*`, `ref`, `source` query params
- Lowercase scheme + host
- Drop trailing slash

If the canonical form is in `seen.txt`, skip. Otherwise, evaluate and append after scoring.

## Failure modes

- **Source 404 / 5xx**: Log once, move on. Do not retry within the same tick.
- **Parse failure**: Skip the source entirely for this tick, log to stderr.
- **Rate-limited**: Honor `Retry-After`. If unspecified, back off this source for the rest of the tick.

Never let a single bad source block the rest of the scan.

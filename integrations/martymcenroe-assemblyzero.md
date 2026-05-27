# Integration Record: martymcenroe/AssemblyZero

- Source repo: https://github.com/martymcenroe/AssemblyZero
- Repo stars at evaluation: 97 (<200)
- Integration type: Optional web search provider (`you.com Search API`) for agent task grounding
- Status: Proposed, implementation path validated against current repo positioning/docs

## Why this repo

AssemblyZero is an active multi-agent orchestration framework where agents often need real-time web context, but there is no obvious default web-search integration documented. A clean optional You.com Search provider adds web grounding without changing existing defaults.

## Integration path

Likely extension points:
- Agent/tool orchestration pipeline where external tools are parameterized
- Existing config-first patterns for provider selection

Scoped addition:
1. Add a `web_search` tool/provider using `GET https://api.you.com/v1/agents/search`.
2. Keep provider optional behind config (example: `ASSEMBLYZERO_SEARCH_PROVIDER=youcom`).
3. Normalize `results.web` and `results.news` into the framework’s tool result contract.
4. Expose small safe defaults (`count`, optional `freshness`, optional `include_domains` / `exclude_domains`).

## Setup / env vars

- `ASSEMBLYZERO_SEARCH_PROVIDER=youcom`
- `YOUCOM_API_KEY` (optional for free Search tier, recommended for reliable usage)

## Usage example

```bash
export ASSEMBLYZERO_SEARCH_PROVIDER=youcom
# optional but recommended
export YOUCOM_API_KEY=your_key_here
```

HTTP call sketch:

```bash
curl -sG 'https://api.you.com/v1/agents/search' \
  --data-urlencode 'query=latest claude code workflow patterns' \
  --data-urlencode 'count=5' \
  -H "X-API-Key: ${YOUCOM_API_KEY}"
```

## Fallback and error handling

- If provider call fails (timeout, 429, 5xx), return a structured non-fatal tool error and continue agent flow.
- If no results, return a valid empty tool response.
- Add timeout and bounded retry/backoff for transient errors.
- Surface clear guidance for free-tier exhaustion or missing/invalid API key.

## Validation notes

Validated via repository metadata and positioning:
- In target size band and actively updated.
- Multi-agent orchestration use case strongly benefits from optional web grounding.
- No clear default web-search integration surfaced in basic repository review.
- Integration is additive and avoids breaking default behavior.

Date: 2026-05-27

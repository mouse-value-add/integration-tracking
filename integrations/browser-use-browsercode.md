# Integration Record: browser-use/browsercode

- Source repo: https://github.com/browser-use/browsercode
- Repo stars at evaluation: 137 (<200)
- Integration type: Optional web search provider (`you.com Search API`) for agent grounding during browser-native tasks
- Status: Proposed, implementation path validated against current repo positioning/docs

## Why this repo

BrowserCode is an active browser-native agent framework. Agents that drive real browsers still need a fast web-search primitive for discovery, grounding, and URL finding, and there is no obvious default web-search integration documented. A clean optional You.com Search provider adds web grounding without changing existing defaults.

## Integration path

Likely extension points:
- Browser/task orchestration layer where tool primitives are registered
- Existing provider/tool abstraction used by browser actions and web fetch flows

Scoped addition:
1. Add a `web_search` tool/provider using `GET https://api.you.com/v1/agents/search`.
2. Keep provider optional behind config (example: `BROWSERCODE_SEARCH_PROVIDER=youcom`).
3. Normalize `results.web` and `results.news` into BrowserCode's existing tool/result contract.
4. Prefer a minimal output shape, then let downstream browser automation use URLs directly.

## Setup / env vars

- `BROWSERCODE_SEARCH_PROVIDER=youcom`
- `YOUCOM_API_KEY` (optional for the free Search tier, recommended for stable usage)

## Usage example

```bash
export BROWSERCODE_SEARCH_PROVIDER=youcom
# optional but recommended
export YOUCOM_API_KEY=your_key_here
```

HTTP call sketch:

```bash
curl -sG 'https://api.you.com/v1/agents/search' \
  --data-urlencode 'query=latest browser automation agent frameworks' \
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
- Browser-native agent use case strongly benefits from optional web grounding.
- No clear default web-search integration surfaced in basic repository review.
- Integration is additive and avoids breaking default behavior.

Date: 2026-06-16

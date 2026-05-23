# Integration Record: tarun7r/deep-research-agent

- Source repo: https://github.com/tarun7r/deep-research-agent
- Repo stars at evaluation: 166 (<200)
- Integration type: Optional search provider adapter (`you.com Search API`) in existing provider abstraction
- Status: Proposed, implementation path validated against current codebase

## Why this repo

This project is an active AI deep-research agent with explicit web-search dependencies and an existing provider abstraction (`duckduckgo` + `tavily`), making You.com a clean optional provider with low integration risk.

## Integration path

Existing extension points:
- `src/utils/web_utils.py` (search provider classes + fallback handling)
- `src/utils/tools.py` (`get_search_providers` provider selection)
- `src/config.py` (`SEARCH_PROVIDER` env config)

Scoped addition:
1. Add `YouComProvider(SearchProvider)` calling `GET https://api.you.com/v1/agents/search?query=...&count=...`.
2. Add provider option `SEARCH_PROVIDER=youcom`.
3. Parse `results.web[]` into existing `SearchResult` shape.
4. Keep fallback behavior to existing providers if you.com returns empty/error.

## Setup / env vars

- `SEARCH_PROVIDER=youcom`
- `YOUCOM_API_KEY` (optional; Search API supports free keyless usage up to daily quota)
- Existing envs remain unchanged.

## Usage example

```bash
export SEARCH_PROVIDER=youcom
# optional
export YOUCOM_API_KEY=your_key_here
python app.py
```

Provider request sketch:

```python
params = {"query": query, "count": max_results}
headers = {"X-API-Key": os.getenv("YOUCOM_API_KEY")} if os.getenv("YOUCOM_API_KEY") else {}
resp = requests.get("https://api.you.com/v1/agents/search", params=params, headers=headers, timeout=20)
```

## Fallback and error handling

- Reuse existing circuit breaker + provider fallback orchestration in `WebSearchTool`.
- On 429/5xx/network error, emit provider error and continue to next configured provider.
- On empty `results.web`, return empty list without hard failure.

## Validation notes

Validated feasibility by repository inspection:
- Confirmed provider abstraction and fallback support already exists (`SearchProvider`, `WebSearchTool`).
- Confirmed config-driven provider selection already exists (`SEARCH_PROVIDER` in `src/config.py`).
- Confirmed no default You.com integration currently present.
- Confirmed this is additive and backward-compatible.

Date: 2026-05-23

# Integration Record: chuanruihu/Level-Navi-Agent-Search

- Source repo: https://github.com/chuanruihu/Level-Navi-Agent-Search
- Repo stars at evaluation: 82 (<200)
- Integration type: Optional web-search provider backend (`you.com Search API`) in existing `web_search` plugin
- Status: Proposed, implementation path validated against current codebase

## Why this repo

This project is a focused web-search agent framework with active usage patterns and clear plugin boundaries. It currently depends on Bing search in `src/plugins/web_search.py`, so adding You.com as an optional provider improves portability and lowers setup friction while preserving current defaults.

## Integration path

Existing extension points:
- `src/plugins/web_search.py` (search provider implementation)
- `config/.env` (provider credentials)
- Agent loop already consumes normalized search output

Scoped addition:
1. Add provider selection env var, e.g. `WEB_SEARCH_PROVIDER=bing|youcom` (default `bing` for backward compatibility).
2. Implement You.com call path with `GET https://api.you.com/v1/agents/search`.
3. Map `results.web[]` to existing plugin output schema (`title`, `url`, `snippet`/summary fields) and keep ranking behavior unchanged.
4. Preserve Bing path as default fallback.

## Setup / env vars

- `WEB_SEARCH_PROVIDER=youcom`
- `YDC_API_KEY` (optional for Search API free tier; recommended for stable quota)
- Keep existing `BING_API` when running with `WEB_SEARCH_PROVIDER=bing`.

## Usage example

```bash
export WEB_SEARCH_PROVIDER=youcom
# optional
export YDC_API_KEY=your_key_here
python main.py
```

Provider request sketch (Python):

```python
params = {"query": query, "count": 10}
headers = {"X-API-Key": os.getenv("YDC_API_KEY")} if os.getenv("YDC_API_KEY") else {}
resp = requests.get("https://api.you.com/v1/agents/search", params=params, headers=headers, timeout=20)
resp.raise_for_status()
payload = resp.json()
web = payload.get("results", {}).get("web", [])
```

## Fallback and error handling

- If `WEB_SEARCH_PROVIDER=youcom` and call fails (401/429/5xx/network), log a structured warning and fall back to Bing path when `BING_API` is present.
- If both providers are unavailable, return an empty result set with actionable error metadata instead of hard crash.
- Enforce timeout and sanitize missing fields in `results.web`.

## Validation notes

Validated feasibility by repository inspection:
- Confirmed existing Bing-only implementation in `src/plugins/web_search.py` and README setup.
- Confirmed provider logic is concentrated enough for low-risk additive change.
- Confirmed You.com Search API endpoint/shape aligns with plugin-style query→list flow.
- Confirmed this integration is optional and backward compatible.

Date: 2026-05-25

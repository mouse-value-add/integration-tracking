# Integration Record: hajirufai/ai-agent-toolkit

- Source repo: https://github.com/hajirufai/ai-agent-toolkit
- Repo stars at evaluation: 0 (<200)
- Integration type: Optional web search tool backend (`you.com Search API`) for toolkit tool registry
- Status: Proposed, implementation path validated against current codebase

## Why this repo

This repo is an active, lightweight agent framework with explicit web-search tooling in examples and project structure, but no fixed default provider implementation documented. It is a good fit for a clean provider-backed `web_search` tool using You.com Search API.

## Integration path

Existing extension points:
- `tools/web_search.py` (search tool implementation point)
- `tools/base.py` and `@tool` decorator (tool registration contract)
- agent/orchestrator examples that already consume `web_search`

Scoped addition:
1. Implement You.com-backed `web_search` in `tools/web_search.py` via `GET https://api.you.com/v1/agents/search`.
2. Add optional provider config/env wiring (`WEB_SEARCH_PROVIDER=youcom`, default unchanged).
3. Map `results.web[]` and/or `results.news[]` into existing string/tool-return format.
4. Preserve backward compatibility by keeping existing default behavior and making You.com opt-in.

## Setup / env vars

- `WEB_SEARCH_PROVIDER=youcom`
- `YOUCOM_API_KEY` (optional; Search API supports free keyless usage up to daily quota)
- Existing model/env settings remain unchanged.

## Usage example

```bash
export WEB_SEARCH_PROVIDER=youcom
# optional
export YOUCOM_API_KEY=your_key_here
python examples/research_agent.py
```

Provider request sketch:

```python
params = {"query": query, "count": max_results}
headers = {"X-API-Key": os.getenv("YOUCOM_API_KEY")} if os.getenv("YOUCOM_API_KEY") else {}
resp = requests.get("https://api.you.com/v1/agents/search", params=params, headers=headers, timeout=20)
resp.raise_for_status()
payload = resp.json()
web = payload.get("results", {}).get("web", [])
```

## Fallback and error handling

- If provider is `youcom` and request fails (429/5xx/network), return a structured tool error and optionally fall back to existing provider logic when available.
- Handle empty `results.web` gracefully by returning an empty/no-results message.
- Enforce request timeout and avoid raising raw exceptions to agent loop.

## Validation notes

Validated feasibility by repository inspection:
- Confirmed explicit `web_search` tool presence in README structure and examples.
- Confirmed no explicit default You.com integration currently documented.
- Confirmed integration can be additive and opt-in with minimal blast radius.
- Confirmed repo activity is recent enough for practical contribution consideration.

Date: 2026-05-24

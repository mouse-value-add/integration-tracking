# Integration Record: zhiheng-huang/toolloop

- Source repo: https://github.com/zhiheng-huang/toolloop
- Repo stars at evaluation: 18 (<200)
- Integration type: Optional web search provider (`you.com Search API`) as an additive tool for agent workflows
- Status: Proposed, implementation path validated against current codebase and docs

## Why this repo

ToolLoop is an actively maintained agent framework focused on tool use, sub-agents, and production SDK/API usage, but it does not present a clear default web-search provider in the toolset/docs. Adding You.com Search API as an optional provider gives agents reliable web grounding with minimal integration overhead.

## Integration path

Existing extension points:
- Tool registration surface (`core/tools` and SDK `allowed_tools` usage)
- API/CLI flows that already expose tool execution to agent loops
- Environment-based config patterns (`TOOLLOOP_*` variables)

Scoped addition:
1. Add a `WebSearch` tool implementation that calls `GET https://api.you.com/v1/agents/search`.
2. Gate provider behavior behind a config flag (example: `TOOLLOOP_WEB_SEARCH_PROVIDER=youcom`) so default behavior remains unchanged.
3. Map `results.web`/`results.news` into ToolLoop’s existing tool result shape (title, URL, snippet, optional freshness).
4. Keep request params small by default (`count`, optional `freshness`, optional domain include/exclude) and preserve compatibility with current agent loops.

## Setup / env vars

- `TOOLLOOP_WEB_SEARCH_PROVIDER=youcom`
- `YOUCOM_API_KEY` (optional for Search free tier; recommended for stable usage)

## Usage example

```bash
export TOOLLOOP_WEB_SEARCH_PROVIDER=youcom
# optional but recommended
export YOUCOM_API_KEY=your_key_here
```

Python request sketch:

```python
import os
import requests

params = {"query": query, "count": count}
headers = {"X-API-Key": os.getenv("YOUCOM_API_KEY")} if os.getenv("YOUCOM_API_KEY") else {}
resp = requests.get("https://api.you.com/v1/agents/search", params=params, headers=headers, timeout=20)
resp.raise_for_status()
payload = resp.json()
web_results = payload.get("results", {}).get("web", [])
```

## Fallback and error handling

- If provider is `youcom` and the request fails (timeout, 429, 5xx), return a structured tool error and allow the agent to continue.
- Gracefully return no-results output when `results.web`/`results.news` are empty.
- Add bounded timeout and retry/backoff for transient failures.
- For free-tier keyless usage, surface clear guidance when quota/auth errors occur.

## Validation notes

Validated via repository and README inspection:
- Repo is active (recent commits/activity) and in target star range.
- Tool-based architecture makes adding a search tool straightforward.
- No clear documented default web-search backend was found.
- Integration is additive and low-risk, with no required breaking changes.

Date: 2026-05-26

---
name: scholar-labs-search
description: Search Google Scholar Labs through BrowserOS MCP to find the top 3 best-fit papers for a topic, with optional seminal or survey mode and manual handoff for sign-in or anti-bot interruptions. Use when Codex needs an on-demand literature search against Scholar Labs rather than a full literature review.
---

# Scholar Labs Search

Use BrowserOS MCP to inspect Google Scholar Labs and return the top 3 best-fit papers for the user's topic. Treat Scholar Labs as the primary search surface. Use Exa only after selection to verify or improve outbound links.

## Setup

Require BrowserOS running locally with MCP enabled before using this skill.

If the `browseros` MCP server is not configured in Codex, stop and tell the user to add it from BrowserOS Settings -> BrowserOS as MCP. Do not guess the MCP URL.

Use this config shape:

```toml
[mcp_servers.browseros]
url = "http://127.0.0.1:9001/mcp"
```

## Workflow

1. Prepare the search query.
   - If the user gives keywords, use them as-is.
   - If the user gives a short problem statement, rewrite it once into a compact Scholar query using only the main topic and method terms.
   - If the first query returns zero usable results, retry once after dropping the weakest qualifier.
   - If the retry also returns zero usable results, return `0 papers found` and stop.
2. Open Scholar Labs in BrowserOS.
   - Navigate to `https://scholar.google.com/scholar_labs/search?hl=en`.
   - Prefer `new_page` over `new_hidden_page` for active searching. Hidden pages are acceptable for inspection, but query submission may not fire reliably there.
   - After opening the page, use `take_snapshot` or `get_page_content` to confirm that the `Ask Scholar` textbox is visible.
   - Submit the query using the working BrowserOS flow described in [workflow.md](references/workflow.md). Do not assume `fill` plus `Enter` will submit.
3. Handle blocked states exactly as described in [workflow.md](references/workflow.md).
4. Extract up to the first visible results page, `N <= 10`.
   - Capture visible title, authors, year, venue or source, Scholar link, short annotation, citation signal if present, and result position.
   - Leave missing fields blank instead of inferring them.
   - Deduplicate obvious duplicate versions by normalized title plus year. Keep the higher-positioned or richer card.
5. Rank the candidates exactly as described in [ranking.md](references/ranking.md).
6. Use Exa only after the top 3 are selected.
   - Replace the Scholar link only if identity is verified as described in [ranking.md](references/ranking.md).
7. Return the result.
   - Default output: 3 papers.
   - For each paper, include title, authors, year, venue or source, best verified link, and a one-line reason it was selected.
   - If the user explicitly asks for all candidates, return the visible first-page candidates with one-line notes instead of only 3.
   - If fewer than 3 usable results remain, return the smaller set and say why.

## Scope

Use only the first visible Scholar Labs page in v1. Do not paginate. Do not turn this into a full literature review. Do not silently fall back to OpenAlex or Semantic Scholar if Scholar Labs remains blocked.

## BrowserOS Notes

- The reliable tool sequence is usually:
  1. `new_page`
  2. `take_snapshot` or `get_page_content`
  3. `evaluate_script` to set `#gs_as_i_t`, dispatch `input`, and click `#gs_as_i_s`
  4. `exec_command` with `sleep 6` to wait for Scholar Labs to render results
  5. `take_snapshot` and `get_page_content` to poll for visible results
- `fill` and `press_key Enter` may leave the query in the textbox without submitting it.
- The visible circular button `#gs_as_i_nsb` is not the submit action for a new query. The submit control that worked in practice is `#gs_as_i_s`.
- When running multiple searches in one session, click `New session` before issuing the next query so result sets do not mix.
- Once the result cards are visible, capture only the first visible page and stop. Ignore deeper loading states such as `Looking for more results...`.

## Failure Rules

Stop instead of guessing when any of the following happens:

- BrowserOS MCP is unavailable.
- Scholar Labs remains blocked after one manual `resume`.

In those cases, say what failed and keep the response narrow.

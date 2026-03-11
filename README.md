# Scholar Labs Search

`scholar-labs-search` is a Codex skill for finding the top 3 best-fit papers for a topic using Google Scholar Labs through BrowserOS MCP.

It is intended for fast, targeted literature lookup, not for full systematic reviews. It is optimized for quick, best-fit paper retrieval from the **first** visible Google Scholar Labs results page. This skill intentionally does not silently switch to another scholarly search engine when Scholar Labs is blocked.

## Requirements

> [!NOTE]
> It is expected that the skill works with any CLI agent that supports local skills. But it was tested only with Codex.

You need:

- CLI agent (Codex/Claude Code) with local skills support
- [BrowserOS](https://www.browseros.com/) installed and running locally with MCP enabled
- Network access to `https://scholar.google.com/scholar_labs/search?hl=en` (it's not available in all regions)
- [Exa MCP](https://exa.ai/mcp) configured if you want verified DOI or publisher links after paper selection

Practical note:

- Google Scholar Labs may occasionally require sign-in or a CAPTCHA. When that happens, the skill pauses and asks the user to complete the page in BrowserOS, then resume.
- You have to run BrowserOS manually to enable the BrowserOS MCP to use this skill.

## Installation

Copy this folder into your local Codex skills directory:

```text
~/.codex/skills/scholar-labs-search
```

The folder should contain:

```text
scholar-labs-search/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── ranking.md
    └── workflow.md
```

After copying the folder, start a new Codex session so the skill is discovered.

## MCP Configuration

### BrowserOS

BrowserOS MCP is required. If it is not configured in BrowserOS, add it in `BrowserOS Settings` -> `BrowserOS as MCP`, then ensure Codex has this entry:

```toml
[mcp_servers.browseros]
url = "http://127.0.0.1:9001/mcp" # or whatever your local BrowserOS MCP URL is
```

See more about configuring BrowserOS MCP in the [BrowserOS documentation](https://docs.browseros.com/features/use-with-claude-code).

### Exa

Exa is used after paper selection to improve outbound links. So, if it is not configured, the skill logic should still be usable for paper selection, but link verification will be weaker.

```toml
[mcp_servers.exa]
url = "https://mcp.exa.ai/mcp"
```

## How To Use

Open the BrowserOS manually and ensure that the `browseros mcp` is up and running. Then, one can either invoke the skill explicitly in a prompt:

```text
Use $scholar-labs-search to find the top 3 best-fit papers for my topic.
```

For example, something like:

```text
Use $scholar-labs-search to find the top 3 best-fit papers on sediment trapping in estuaries.
```

Or the skill can also be used for related prompts where Codex is asked to search Scholar Labs for a few relevant papers rather than run a full literature review.

## Expected Workflow

At a high level, the skill:

1. Rewrites the request into a compact Scholar Labs query if needed.
2. Opens Scholar Labs in BrowserOS.
3. Submits the query.
4. Waits for results to render.
5. Extracts only the first visible results page.
6. Ranks the best-fit papers.
7. Uses Exa to verify better links for the top selections.

If the first query returns no usable results, the skill retries once with a simpler query.

## Manual Handoff

If Scholar Labs is blocked by sign-in or anti-bot checks, the skill uses this handoff:

```text
Scholar Labs is blocked in BrowserOS. Complete the page in BrowserOS until the results list is visible, then reply 'resume'. Reply 'abort' to stop.
```

After `resume`, the skill inspects the current page and continues only if results are visible.

## Tips

- Use compact noun-phrase queries rather than long conceptual questions.
- If a query is too abstract, drop weak qualifiers first.
- When running multiple searches in one tab, Scholar Labs works more reliably if the session is reset before the next query.

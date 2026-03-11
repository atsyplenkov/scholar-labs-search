# Scholar Labs Search

`scholar-labs-search` is a Codex skill that finds the top 3 best fit papers for a given topic using Google Scholar Labs via BrowserOS MCP. For those who have never worked with Google Scholar Labs, it is an LLM-powered search through one of the biggest scientific literature database.

This skill is for fast, targeted literature lookup, not for systematic reviews. It is optimised for quick, best fit retrieval from the first visible Google Scholar Labs results page. It also does not silently switch to another search engine if Google Scholar Labs is blocked or unavailable.

See [Motivation](#motivation) for why this exists.

## Requirements

> [!NOTE]
> The skill should work with any CLI agent that supports local skills, but it has only been tested with Codex.

You need:

- A CLI agent (e.g., Codex or Claude Code) with local skills support
- [BrowserOS](https://www.browseros.com/) installed and running locally, with MCP enabled
- Network access to `https://scholar.google.com/scholar_labs/search?hl=en` (it is not available in all regions)
- Optional: [Exa MCP](https://exa.ai/mcp) if you want verified DOI or publisher links after paper selection

Practical note:

- Google Scholar Labs may occasionally require a sign-in or a CAPTCHA. When that happens, the skill pauses and asks you to complete the page in BrowserOS, then resume.
- You must run BrowserOS yourself for the BrowserOS MCP to be available.

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

BrowserOS MCP is required. If it is not configured in BrowserOS, enable it in `BrowserOS Settings` -> `BrowserOS as MCP`. Then ensure your CLI agent has an MCP entry like this:

```toml
[mcp_servers.browseros]
url = "http://127.0.0.1:9001/mcp" # or whatever your local BrowserOS MCP URL is
```

See more about configuring BrowserOS MCP in the [BrowserOS documentation](https://docs.browseros.com/features/use-with-claude-code).

### Exa

Exa is only used after paper selection to improve outbound links. If you do not configure Exa, the core paper selection still works, but link verification is weaker.

```toml
[mcp_servers.exa]
url = "https://mcp.exa.ai/mcp"
```

## How To Use

Open BrowserOS and confirm the BrowserOS MCP server is running. Then you can invoke the skill explicitly:

```text
Use $scholar-labs-search to find the top 3 best fit papers for my topic.
```

For example, something like:

```text
Use $scholar-labs-search to find the top 3 best fit papers on sediment trapping in estuaries.
```

You can also use the skill in a workflow where you need a few targeted citations rather than a full literature review. One practical pattern is to leave `(REFERENCE!)` placeholders in a manuscript and ask the agent to replace them with suitable citations. For example:

```text
In @manuscript.md there are three places where a literature reference is needed (marked with "REFERENCE!"). Using $scholar-labs-search, find relevant studies to cite for each place. Add the citations to the text and update the reference list.
```

## Motivation
For the last half a year I do most of my day-to-day work in Codex or Claude Code, but I still do not want an agent to write my papers end-to-end. I also think it is important to be careful about the ethics of LLM-assisted writing.

That said, there are many moments where I only need a surgical change, such as adding a citation that supports a specific statement, especially after major revisions. Finding that exact paper can take a long time, even when you already know the topic well.

Google Scholar Labs makes this much faster for targeted searching, but it does not offer a public API. This skill is a practical workaround that lets an agent use Scholar Labs through BrowserOS and bring back a short, high-quality shortlist.

## Alternatives

If you do not need Scholar Labs specifically, there are strong alternatives. Semantic Scholar and OpenAlex provide APIs that can be enough for keyword and metadata-based searching.

I have also seen a [scientific-literature-researcher subagent](https://github.com/VoltAgent/awesome-claude-code-subagents/blob/main/categories/10-research-analysis/scientific-literature-researcher.md) that uses the `bgpt` MCP. In my own use, Scholar Labs gave better results for the kinds of targeted lookups this skill is meant to support.

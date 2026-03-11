# Scholar Labs Search

`scholar-labs-search` is a Codex skill for finding the top 3 best-fit papers for a topic using Google Scholar Labs through BrowserOS MCP. For those who have never worked with Google Scholar Labs, it is an LLM-powered search through one of the biggest scientific literature database.

It is intended for fast, targeted literature lookup, not for full systematic reviews. It is optimized for quick, best-fit paper retrieval from the **first** visible Google Scholar Labs results page. This skill intentionally does not silently switch to another scholarly search engine when Scholar Labs is blocked.

See Motivation to learn more why this exists.

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

Or the skill can also be used for related prompts where Codex is asked to search Scholar Labs for a few relevant papers rather than run a full literature review. My usual workflow is to leave "(REFERENCE!)" placeholders trhoughout the manuscript and then run Codex asking to replace them with relevant citations. For example:

```text
In the @manuscript.md there are three places where a literature reference is needed (marked with a "REFERENCE!" tag). USing the $scholar-labs-search find a relevant studies one can cite there to support the logic. Add them to text and reference list
```

## Motivation
Even though for the last half a year, I have been doing everyhting in Codex/Claude Code, I am still not ready to handle the article writing to them. And from ethical POV, I am not sure if I should. However, sometimes there is a need to "surgically" make a change in the text, to support a statement with a relevant citation, especially after Major Revisions. I am talking not about the backbone previous papers of your research, but more about small... But the process of finding that exact CITATION is sometimes very time consuming. And I need to acknowledge The Google Scholar Labs launched in November 2025 was a great helper, helping to reduce the amount browsing thought the endelss Google Scholar catalog. They were not pioneer in this space (LIST ALTERNATIVES) but from my perspective produced one of the best products in this field. So in one moment I thought, why shouldnt I delegate this process to the agents? Since the Google Scholar Labs doesnt have an API yet and I doubt that we can dream about the MCP server, this workaround came to mind. 

## Alternatives

Apart from obvious Semantic Scholar, OpenAlex APIs, which does only allow you to search through the word matching in abstract, I found only one [scientific-literature-researcher subagent](https://github.com/VoltAgent/awesome-claude-code-subagents/blob/main/categories/10-research-analysis/scientific-literature-researcher.md) that uses the `bgpt` mcp. But frankly, the bgpt was not as good as Google Scholar Labs.

*

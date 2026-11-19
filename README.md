# Box skills for Gemini

Gemini plugin that ships a subset of the Box agent skills. Gemini loads every skill under `skills/`, and this repository contains only the skills meant for the Gemini connector.

## Skills

| Skill | When to use it |
|---|---|
| [`box-core`](skills/box-core/SKILL.md) | Box MCP guardrails and tool guidance for search, files, metadata, Box AI, Hubs, and Doc Gen. |
| [`box-mcp`](skills/box-mcp/SKILL.md) | Operate Box through the active MCP connection, using the tools exposed in the session. |
| [`box-legal-workflows`](skills/box-legal-workflows/SKILL.md) | Shared legal reference for the intake, contract, and M&A skills. |
| [`box-legal-workflows-intake`](skills/box-legal-workflows-intake/SKILL.md) | Check intake submissions, extract matter details, and route attorney review. |
| [`box-legal-workflows-contract`](skills/box-legal-workflows-contract/SKILL.md) | Compare contracts with approved templates and capture cited variances. |
| [`box-legal-workflows-ma`](skills/box-legal-workflows-ma/SKILL.md) | Build an M&A data room and answer due diligence questions with citations. |

The intake, contract, and M&A skills use `box-core` and `box-legal-workflows`. Firms supply the checklists, templates, and review criteria.

## Plugin layout

`plugin.json` is the Gemini manifest. `mcp_config.json` points at `https://mcp.box.com`. The Gemini client handles authentication. Format: [Gemini plugin docs](https://antigravity.google/docs/build-plugins/).

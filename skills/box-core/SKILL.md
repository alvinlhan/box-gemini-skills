---
name: box-core
description: "Rules, guardrails, and tool guidance for Box MCP tools: search, files, folders, metadata, Box AI Q&A/extraction, Hubs, Doc Gen, and troubleshooting. Use for Box content workflows, metadata search, AI extraction, or error recovery."
---

# Box Core (MCP)

## Overview

This is the foundation skill for operating Box through the active Box MCP connector. Use it to:

- Verify the acting Box identity via `who_am_i`.
- Route requests to the right Box MCP domain reference below.
- Apply mandatory Box MCP guardrails for sharing, content display, Box AI, Hubs, and Doc Gen.
- Diagnose common Box MCP errors (`401`/`403`/`404`/`409`/`429`, missing content, missing tools, or wrong actor).

Only Box MCP tools are available in Sobi. Do not instruct the user to run Box CLI commands, `curl`, or REST scripts.

## Domain Routing

Choose which reference file to read based on what the user needs:

| If the user needs... | Read first | Pair with | Minimal verification |
| --- | --- | --- | --- |
| Uploads, folders, listings, content reads, previews, moves, or metadata | `references/content-workflows.md` | `references/guardrails.md` | Read-after-write call using the same actor |
| Auditing collaborators or creating/updating shared links | `references/collaboration.md` | `references/content-workflows.md`, `references/guardrails.md` | Call `list_item_collaborations` (chunked at 10 items) before and after changes |
| Finding files or folders (keyword, folder name, metadata search) | `references/mcp-search.md` | `references/guardrails.md` | Confirm item details (`get_file_details` / `get_folder_details`) before acting |
| Box AI Q&A, summarization, extraction, or agent queries | `references/ai-and-retrieval.md` | `references/mcp-search.md`, `references/guardrails.md` | Verify candidate file set and citations before answering |
| Box Hubs — creating, curating, or querying a hub | `references/mcp-hubs.md` | `references/ai-and-retrieval.md`, `references/guardrails.md` | Confirm hub items after changes |
| Document generation from templates (Doc Gen) | `references/mcp-doc-gen.md` | `references/content-workflows.md`, `references/guardrails.md` | Confirm template, fields, and destination before batch |
| Organizing or moving many files/folders; bulk metadata tagging | `references/bulk-operations.md` | `references/content-workflows.md`, `references/ai-and-retrieval.md` | Inventory source, confirm batch cap, verify target counts |
| `401`, `403`, `404`, `409`, `429`, missing content, missing tools, or wrong-actor bugs | `references/troubleshooting.md` | `references/mcp-search.md` | Confirm acting user (`who_am_i`), object ID, and error message |

## MCP Usage

### Availability check

- Call `who_am_i` to confirm the active Box user identity and login.
- If `who_am_i` fails with an authentication error, ask the user to reconnect their Box connector in Sobi and retry.

### Tool availability and documentation

The maintained list of Box MCP tools is documented at https://docs.box.com/en/box-mcp/tools. If an expected tool is missing or unavailable, see `references/troubleshooting.md` (MCP tool missing section) — never invent or call a tool that is not in the live tool list.

### General guidelines

- When listing folder contents with `list_folder_content_by_folder_id`, paginate if needed and summarize large directories rather than printing every item.
- Prefer reading file or folder details (`get_file_details` / `get_folder_details`) before mutating an object — it confirms the object exists and reveals its current path, permissions, and shared links.
- For exploratory or demo usage, work inside a dedicated folder rather than across the entire Box account.
- Apply least privilege — only search and access the folders and files the task requires.

### MCP capability map

| MCP category | Reference |
| --- | --- |
| Mandatory safety & confirmation rules | `references/guardrails.md` |
| Files, folders, text uploads, content reads, previews, metadata | `references/content-workflows.md` |
| Search (keyword, folder name, metadata) | `references/mcp-search.md` |
| Collaborator audits (read-only) and shared links | `references/collaboration.md` |
| Box AI Q&A, structured extraction, and Box AI agents | `references/ai-and-retrieval.md` |
| Box Hubs creation, item management, and hub-level Q&A | `references/mcp-hubs.md` |
| Box Doc Gen templates and document generation batches | `references/mcp-doc-gen.md` |
| Multi-file organization, folder trees, and rate-limit handling | `references/bulk-operations.md` |
| Error diagnosis and missing-tool troubleshooting | `references/troubleshooting.md` |

## Guardrails

> **Read and follow `references/guardrails.md` in every session.** It covers Sobi write approvals, content display preferences, hub modifications, file comments, Doc Gen output locations, externally shared folders, and Box AI governance.

- In Sobi, **every write tool call is shown to the user for approval automatically**. Never ask the user whether future writes can skip approval or run unattended, and never promise to bypass approvals.
- Distinguish file IDs, folder IDs, shared links, metadata template keys/scopes, and hub IDs.
- Treat shared links, collaborator exposure, hub updates, and metadata writes as permission-sensitive changes. Confirm audience, scope, and least privilege before invoking write tools.
- Require explicit confirmation before widening external access or retrieving broad document sets.
- When a task requires understanding document content, try Box AI (`ai_qa_*`, `ai_extract_*`) first — it runs server-side without exporting raw content. See `references/ai-and-retrieval.md` for the full preference order.
- Pace Box AI calls at least 1–2 seconds apart. For classifying many files, use the sample-first strategy in `references/bulk-operations.md`.
- Do not attempt to download raw file bytes locally when Box-native methods (Box AI, search, metadata, `get_file_content`) can answer the question; if a file cannot be read through MCP, share its Box link.
- Request only the fields needed for the task, and reuse returned Box IDs instead of reconstructing paths.
- Keep AI retrieval narrow for search and Q&A tasks: search and filter first, then retrieve only the files needed for the answer.

## Verification

- Verify writes with a read-after-write call (`get_file_details`, `get_folder_details`, `list_folder_content_by_folder_id`, or `list_item_collaborations`) using the same actor, and include the affected Box object IDs and URLs in your summary.
- For AI flows, confirm the candidate file set and preserve returned citations before formatting the answer.

## References

- `references/guardrails.md`: mandatory safety, Sobi approval, and scope rules for Box MCP usage
- `references/content-workflows.md`: files and folders — text uploads, reads, previews, folder trees, moves, metadata
- `references/collaboration.md`: collaborator role matrix, read-only audits (`list_item_collaborations`), and shared links
- `references/mcp-search.md`: keyword, folder-name, and parameterized metadata search
- `references/ai-and-retrieval.md`: Box AI Q&A, structured extraction (up to 50 files/call), `ask_default_agent` / `ask_pro_agent`, and content understanding order
- `references/mcp-hubs.md`: Box Hubs creation, item management (50 items/call), and hub-level Q&A
- `references/mcp-doc-gen.md`: Box Doc Gen template registration and document generation batches
- `references/bulk-operations.md`: MCP-native multi-file organization, folder-level moves, batch caps, and rate-limit handling
- `references/troubleshooting.md`: common Box MCP failure modes (`401`/`403`/`404`/`409`/`429`), search issues, and missing tools

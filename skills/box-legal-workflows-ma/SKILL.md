---
name: box-legal-workflows-ma
description: "Set up and run an M&A data room in Box: build due-diligence folder trees, plan and audit access, and run cross-document Q&A with Box AI. Use for M&A, deal rooms, data rooms, VDRs, or due diligence."
---

# M&A Deal Room Management

> **PREREQUISITES:** Use the `box-core` skill for Box MCP tool mechanics and guardrails, and the `box-legal-workflows` skill for Box collaborator role definitions, Box AI usage boundaries, and reusable confirmation phrasings.

Build and run an M&A data room *in Box*: create the folder hierarchy, plan least-privilege role-based access for the deal lead to apply in Box, audit permissions with `list_item_collaborations`, and answer due-diligence questions with Box AI plus citations. This skill is the deal-room-specific recipe; the underlying Box tool mechanics live in the capability references below. Deal risk, materiality, and terms are attorney calls. Not legal advice.

## Box capability references

Reach for these for tool mechanics rather than restating them here:

- The `box-core` skill's `references/content-workflows.md` — create the folder hierarchy, upload/copy, classify-and-file submissions
- The `box-core` skill's `references/collaboration.md` — collaborator role matrix, permission audits (`list_item_collaborations`, max 10 items per call), and confirmed shared links
- The `box-core` skill's `references/mcp-search.md` — locate documents, folder-scoped search
- The `box-core` skill's `references/ai-and-retrieval.md` — due-diligence Q&A and term extraction with citations

## Folder structure

Create folders using `create_folder` (top-down, parent before child; names max 255 characters, no slashes; reuse the existing folder on a `409` name conflict). Show the full proposed tree and get one OK in chat first. Because every `create_folder`, `upload_file`, and `copy_file` call triggers a separate Sobi write approval, create only the top-level deal room plus the specific subfolders the user needs right now.

Example numbered structure — numeric prefixes keep ordering consistent and segregate external submissions:

```
[Deal Name] M&A Deal Room/
├── 01 - Financial Statements/
├── 02 - Legal Documents/
├── 03 - HR & Employment/
├── 04 - Intellectual Property/
├── 05 - Commercial Contracts/
├── 06 - Real Estate & Assets/
├── 07 - IT & Cybersecurity/
└── 08 - External Submissions/
```

## Access model

You **cannot** add or update named Box collaborators through MCP on Sobi. Scope access least-privilege and folder-specific rather than root (role capabilities and external-sharing rules are in the `box-core` skill's `references/collaboration.md` and `box-legal-workflows`), present the proposed mapping below as a table, and ask the deal lead or folder owner to invite each party in Box. After they confirm, verify access with `list_item_collaborations` (chunked at max 10 items per call).

Example deal-room mapping (confirm with the user):

- **Internal:** Deal Lead → Editor/Co-Owner on root; Finance → Viewer on `01 - Financial Statements`; Legal → Editor on `02 - Legal Documents`.
- **External:** External Counsel → Uploader on `08 - External Submissions`; Auditors → Viewer on `01 - Financial Statements`; Prospective Buyer → Viewer on a curated subset, not the full room.

## Tool selection

| Deal-room task | Tool | Notes |
|------|------|-------|
| Create folders | `create_folder` | Confirm full tree first; each folder is a separate Sobi approval — create only requested folders now |
| Add files | `upload_file` (text only) / `copy_file` | Binary uploads must be done in Box; copy existing Box files with `copy_file` |
| Grant named access | Manual in Box (not available via MCP) | Present proposed role table; ask deal lead/owner to invite in Box, then verify |
| Shared link | `add_folder_shared_link` | Use only for a confirmed link share; confirm audience/expiration first |
| Audit/verify access | `list_item_collaborations` | Before and after changes; max 10 items per call — chunk folder-tree audits |
| Find docs | `search_files_keyword` | Scope with `ancestor_folder_id` |
| Due-diligence Q&A | `ai_qa_multi_file` | Cross-document; surface citations |
| Extract terms | `ai_extract_structured_from_fields` | Default; use `_enhanced` only if asked or incomplete (higher cost); persist with `set_file_metadata` |
| Classify submissions | `ai_qa_single_file` | Then `copy_file` into the right folder |

## Workflow

1. **Setup**: present the proposed folder tree and get one OK in chat, create the top-level room plus the subfolders requested now with `create_folder`, and present the proposed internal access table for the deal lead to apply in Box. After they confirm, verify with `list_item_collaborations`. **[CONFIRM: folder scope, proposed emails/roles]**
2. **Populate**: `copy_file` existing Box documents (or `upload_file` for text/Markdown files; binary files must be uploaded in Box); classify submissions with `ai_qa_single_file`, then `copy_file` into the right category folder.
3. **External access**: audit current access with `list_item_collaborations` (chunked in batches of 10 items) → **[CONFIRM: who, folders, permission, expiration]** → present the access table and ask the deal lead/folder owner to invite named external parties in Box (or call `add_folder_shared_link` only if a confirmed link share is requested) → re-verify with `list_item_collaborations`.
4. **Due diligence**: `search_files_keyword` (folder-scoped with `ancestor_folder_id`) → `ai_qa_multi_file` → present the answer with citations; use `ai_extract_structured_from_fields` for terms (use `_enhanced` only if explicitly requested or standard results are incomplete, noting higher cost) → `set_file_metadata` to persist.

## Legal guardrails

Box mechanics (external-sharing confirmation, shared-link settings, AI pacing/limits/citations) are governed by the capability references above and `box-legal-workflows`. Specific to deal rooms:

- Deal risk, materiality, and term interpretation are attorney calls, never the agent's.
- Validate permissions with `list_item_collaborations` (chunked at 10 items per call) before *and* after external changes — a folder grant exposes everything inside it, including files added later.
- Audit trail: record returned folder/file IDs and audited collaboration summaries, and write due-diligence summaries back to Box (`create_box_note` or `upload_file` as `.md`).

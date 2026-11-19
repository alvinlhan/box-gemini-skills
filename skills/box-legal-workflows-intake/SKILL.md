---
name: box-legal-workflows-intake
description: "Client intake and onboarding in Box: check intake documents against the firm checklist, extract matter metadata, route to an attorney, and generate engagement letters. Use for client intake, onboarding, or engagement letters."
---

# Client Intake & Onboarding

> **PREREQUISITES:** Use the `box-core` skill for Box MCP tool mechanics and guardrails, and the `box-legal-workflows` skill for Box collaborator role definitions, Box AI usage boundaries, and reusable confirmation phrasings.

Do client intake *in Box*: inventory the intake folder, check completeness with Box AI against the firm's checklist, extract intake data into Box metadata, route via file comments and owner-applied Box access, and generate the engagement letter with Box DocGen. This skill is the intake-specific recipe; the underlying Box tool mechanics live in the capability references below. The firm supplies the required-document checklist and risk criteria; conflict, PEP, and sanctions determinations run through the firm's screening system — not the agent or Box AI. Not legal advice.

## Box capability references

Reach for these for tool mechanics rather than restating them here:

- The `box-core` skill's `references/content-workflows.md` — inventory the folder, metadata templates, `set_file_metadata`, file comments
- The `box-core` skill's `references/ai-and-retrieval.md` — completeness checks and data extraction; pacing, limits, citations
- The `box-core` skill's `references/mcp-doc-gen.md` — generate the engagement letter from a template
- The `box-core` skill's `references/collaboration.md` — role reference, permission audits (`list_item_collaborations`), and confirmed shared links
- The `box-core` skill's `references/mcp-search.md` — find/inspect the firm's metadata template

## Box metadata model

Record the intake outcome as file metadata so submissions stay searchable. Find or create the firm's template using the `box-core` skill's `references/mcp-search.md` and its `references/content-workflows.md`.

- **Representative fields** (confirm the firm's actual set): `client_name`, `matter_name`, `practice_area`, `matter_owner`, `jurisdiction`, `matter_value`, `intake_status` (complete/incomplete), `risk_rating`, `assigned_attorney`, `decision`, `decision_date`.
- Store the firm/attorney's rating and decision, not the agent's.

## Tool selection

| Intake task | Tool | Notes |
|------|------|-------|
| Inventory submission | `list_folder_content_by_folder_id` | All files in the intake folder |
| Check completeness | `ai_qa_multi_file` | Present/complete/valid vs. firm checklist, with citations |
| Extract intake data | `ai_extract_structured_from_fields` | Default; use `_enhanced` only if asked or incomplete (higher cost) |
| Write metadata | `set_file_metadata` | Record status, rating, decision, attorney |
| Leave note for attorney | `create_file_comment` | Addressed by name (max 1,000 chars); does not notify — tell user to message attorney |
| Check / grant access | `list_item_collaborations` + manual in Box | Cannot invite collaborators via MCP; ask folder owner to invite in Box, then verify |
| Generate engagement letter | `create_docgen_batch` | Only if the firm approves |
| Share with client | `add_folder_shared_link` (or owner invite in Box) | Use `add_folder_shared_link` only for a confirmed link share (confirm audience/expiration) |

## Workflow

1. **Inventory**: `list_folder_content_by_folder_id`. **[CONFIRM: intake folder ID]**
2. **Completeness**: `ai_qa_multi_file` — is each required doc (per the firm's checklist) present, complete, and valid, with citations? **[CONFIRM: firm's required-document list]**
3. **Extract intake data**: `ai_extract_structured_from_fields` (`client_name`, `matter_name`, `practice_area`, `jurisdiction`, `matter_value`, …). Use the `_enhanced` variant (`ai_extract_structured_from_fields_enhanced`) only if the user explicitly asks or the standard result is incomplete, and note that enhanced extraction costs more.
4. **Surface risk indicators (facts only)**: `ai_qa_multi_file` to surface the facts the firm's criteria reference (value, jurisdiction, named parties) with citations. Do **not** use AI to determine conflicts, PEP status, or sanctions — pass names/entities/locations to the firm's screening system.
5. **Persist**: `set_file_metadata` with the firm-confirmed `risk_rating`, `intake_status`, and `decision`.
6. **Route**: `create_file_comment` to leave a factual summary addressed to the attorney by name (max 1,000 characters; comments do not send notifications — tell the user to message the attorney directly). Check current access with `list_item_collaborations`; if the attorney needs access, share the Box link and ask the folder owner to invite them in Box. **[CONFIRM: who, access level]**
7. **Engagement letter (only if the firm approves)**: `create_docgen_batch` against the firm's template → output to the confirmed folder → for a confirmed link share use `add_folder_shared_link` (checking runtime schema for expiration/password), or ask the folder owner to invite the client directly in Box. **[CONFIRM: DocGen template, destination folder, share method/expiration]**

## Legal guardrails

Box mechanics (DocGen template/tag requirements, external-sharing confirmation, AI pacing/limits/citations, metadata writes) are governed by the capability references above and `box-legal-workflows`. Specific to intake:

- Conflict, PEP, and sanctions clearance are firm screening + attorney/compliance calls — the agent surfaces the underlying text only and never clears them.
- Risk rating and routing/staffing are firm policy + attorney calls, never the agent's.
- Prompt Box AI to surface facts and cite the source, not to determine risk.

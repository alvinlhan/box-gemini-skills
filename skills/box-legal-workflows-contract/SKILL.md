---
name: box-legal-workflows-contract
description: "Review and monitor Box contracts: find new or expiring contracts, compare against firm templates, write contract metadata, and report variances with citations. Use for NDA/MSA review, renewals, expiration dates, or variance reports."
---

# Contract Review Agent

> **PREREQUISITES:** Use the `box-core` skill for Box MCP tool mechanics and guardrails, and the `box-legal-workflows` skill for Box collaborator role definitions, Box AI usage boundaries, and reusable confirmation phrasings.

Do contract review *in Box*: find contracts with Box search, compare against the firm's template with Box AI, persist results as Box metadata so they stay searchable, and monitor dates with metadata search. This skill is the contract-specific recipe; the underlying Box tool mechanics live in the capability references below. Materiality, risk, and favorability are firm-supplied criteria confirmed by an attorney — the agent extracts facts, stores them, and routes. It does not provide legal advice or decide risk.

## Box capability references

Reach for these for tool mechanics rather than restating them here:

- The `box-core` skill's `references/mcp-search.md` — find contracts; metadata vs. keyword search, folder scoping, `from` formatting, and template schema lookup
- The `box-core` skill's `references/ai-and-retrieval.md` — compare to template and extract fields; pacing, text-rep/file limits, citations
- The `box-core` skill's `references/content-workflows.md` — metadata templates, `set_file_metadata`, text/Markdown report creation, file comments
- The `box-core` skill's `references/collaboration.md` — role reference and read-only permission audits (`list_item_collaborations`; named invites must be applied in Box)

## Box metadata model

Persist review results as file metadata so contracts stay searchable. Find or inspect the firm's template using the `box-core` skill's `references/mcp-search.md`; create one using its `references/content-workflows.md` if none exists.

- **Representative fields** (confirm the firm's actual set): `counterparty_name`, `contract_type`, `execution_date`, `effective_date`, `expiration_date`, `auto_renewal`, `notice_period_days`, `contract_value`, `governing_law`, `status` (active/expired/terminated/under_negotiation), `risk_rating`, `review_date`, `next_review_date`, `expiration_alert_date`. Link to matters with `matter_id`, `practice_area`, `matter_owner`.
- The `risk_rating` value is the firm/attorney's determination — store it, don't decide it.

## Contract search recipes

Once contracts carry metadata, use `search_files_metadata` (see the `box-core` skill's `references/mcp-search.md`; fall back to `search_files_keyword` with date filters if no metadata exists). Always include `from` (`<scope>_<enterpriseId>.<templateKey>` from `list_metadata_templates`) and pass values through `query_params` rather than embedding literals:

- **New since last review:**
  - `from`: `"enterprise_<enterpriseId>.<templateKey>"`
  - `query`: `"execution_date >= :start AND execution_date <= :end"`
  - `query_params`: `{"start": "YYYY-MM-DD", "end": "YYYY-MM-DD"}`
- **Expiring window:**
  - `from`: `"enterprise_<enterpriseId>.<templateKey>"`
  - `query`: `"expiration_date >= :start AND expiration_date <= :end AND status = :status"`
  - `query_params`: `{"start": "YYYY-MM-DD", "end": "YYYY-MM-DD", "status": "active"}`
- **By counterparty / rating:**
  - `from`: `"enterprise_<enterpriseId>.<templateKey>"`
  - `query`: `"counterparty_name = :party"` (`query_params`: `{"party": "Acme Corp"}`) or `query`: `"risk_rating = :rating"` (`query_params`: `{"rating": "High"}`)

Date value formats follow the metadata template's field type (e.g., RFC3339 timestamp vs. `YYYY-MM-DD` string) — run a quick one-row test (`limit: 1`) before a full search.

## Tool selection

| Contract task               | Tool                                           | Notes                                                                                                       |
| --------------------------- | ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Find new contracts (date)   | `search_files_metadata`                        | Pass `from`, parameterized `query`, and `query_params`; fall back to `search_files_keyword` if no metadata |
| Find expiring               | `search_files_metadata`                        | Parameterized `expiration_date` window (30/60/90 days) with `from`                                          |
| Compare to template         | `ai_qa_multi_file`                             | Contract + firm template in one call                                                                        |
| Extract metadata (template) | `ai_extract_structured_from_metadata_template` | If a template exists                                                                                        |
| Extract metadata (custom)   | `ai_extract_structured_from_fields`            | Default; use `_enhanced` only if asked or incomplete (higher cost)                                          |
| Write metadata              | `set_file_metadata`                            | Persist extracted/confirmed fields                                                                          |
| Create variance report      | `create_box_note` (or `upload_file` as `.md`)  | Text/Markdown only; cannot generate `.docx` or `.pdf`                                                       |
| Leave note for attorney     | `create_file_comment`                          | Addressed by name (max 1,000 chars); does not notify — tell user to message attorney                        |
| Check / grant access        | `list_item_collaborations` + manual in Box     | Cannot invite collaborators via MCP; ask folder owner to invite in Box, then verify                         |

## Workflow

1. **Find**: `search_files_metadata` (with `from`, parameterized `query`, `query_params`) or `search_files_keyword`. **[CONFIRM: folder ID, date range/cadence]**
2. **Compare to template**: `ai_qa_multi_file` (contract + firm template) → extract clauses that differ, with citations. **[CONFIRM: template file ID]**
3. **Extract metadata**: `ai_extract_structured_from_metadata_template`, or `ai_extract_structured_from_fields` if no template exists. Use the `_enhanced` variant (`ai_extract_structured_from_fields_enhanced`) only if the user explicitly asks or the standard result is incomplete, and tell the user it costs more.
4. **Persist**: `set_file_metadata`, including the firm-confirmed `risk_rating`. **[CONFIRM: which fields]**
5. **Report**: `create_box_note` (or `upload_file` as a `.md` file) — factual differences + citations + firm-confirmed rating (observations for attorney review, not recommendations). Reports are text/Markdown only; you cannot create `.docx` or `.pdf` via `upload_file`.
6. **Route**: `create_file_comment` to leave a concise note addressed to the attorney by name (max 1,000 characters; comments do not trigger notifications — tell the user to message the attorney directly). Check access with `list_item_collaborations`; if the attorney lacks access, provide the Box link and ask the folder owner to invite them in Box. **[CONFIRM: attorney, access]**
7. **Monitor expirations**: `search_files_metadata` on the expiration window → `create_file_comment` reminders (and tell the user to notify the responsible attorney) → `set_file_metadata` (`expiration_alert_sent: yes`).

## Legal guardrails

Box mechanics (external-sharing confirmation, AI pacing/limits/citations, metadata writes) are governed by the capability references above and `box-legal-workflows`. Specific to contracts:

- Risk, materiality, and favorability are firm + attorney calls, never the agent's. If the firm has no documented criteria, route to an attorney rather than rating.
- Prompt Box AI to extract differences and cite section/page — not to judge materiality or legal risk.
- Record decisions in Box as the audit trail: the firm-confirmed rating in metadata plus the uploaded Markdown/Box Note summary.

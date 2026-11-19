# AI and Retrieval

## Table of Contents

- General
  - Search-first strategy
  - Content understanding preference order
  - Box AI pacing
  - Retrieval guardrails
- MCP
  - Tools
  - Security guidelines
  - Tool selection
  - Workflows
  - Error handling
- Verification checklist
- Primary docs

## General

### Search-first strategy

- Use Box search (`search_files_keyword`, `search_folders_by_name`, `search_files_metadata`) before recursive folder traversal.
- Narrow the candidate set with ancestor folders, object type, filenames, owners, or metadata filters whenever possible.
- Return stable IDs and lightweight metadata first, then retrieve content only for the final shortlist.

### Content understanding preference order

When the task requires understanding what a document contains (classification, extraction, summarization, Q&A), prefer Box-native methods in this order:

1. **Box AI Q&A or Extract** — keeps content server-side, no raw text export needed.
2. **Metadata inspection** — check existing Box metadata templates or properties (`get_file_details`, `search_files_metadata`).
3. **Text representation** — use `get_file_content` to pull the text representation of the file (up to 50MB).

If the first Box AI call fails with a `403` or feature-not-available error, switch to the next method immediately rather than retrying AI for remaining files. If none of the above can read the file, tell the user the file cannot be processed here and provide the Box link (`url`). Do **not** attempt to download files locally.

For content-based classification of many files, use the sample-first strategy in `references/bulk-operations.md` to minimize AI calls.

### Box AI pacing

Box AI endpoints have tighter per-user/per-app rate limits than standard content API calls. Space Box AI tool calls at least 1–2 seconds apart. For bulk classification workflows, use the sample-first strategy in `references/bulk-operations.md` to minimize total AI calls.

Box AI responses include citations — surface them whenever available so the user can verify answers.

### Retrieval guardrails

- Avoid pulling raw file text (`get_file_content`) when metadata or Box AI answers are sufficient.
- Keep retrieval scoped to the smallest relevant set of files.
- Preserve traceability with file IDs, names, Box links, or citations when auditability matters.
- Confirm with the user before broad retrieval across large folders or sensitive content sets.

## MCP

### Tools

- Q&A: `ai_qa_single_file`, `ai_qa_multi_file`, `ai_qa_hub`
- Extraction: `ai_extract_freeform`, `ai_extract_structured_from_fields`, `ai_extract_structured_from_metadata_template`
- Enhanced extraction (more powerful, higher cost): `ai_extract_structured_from_fields_enhanced`, `ai_extract_structured_from_metadata_template_enhanced`
- Agents: `ask_default_agent`, `ask_pro_agent`

### Security guidelines

For `ai_qa_*` tools, always provide citations with links back to the files in Box when answering. If possible, name the section in the file containing the relevant information. For Hubs (`ai_qa_hub`), specify which file the information came from.

### Tool selection

- When extracting data from a document, check first whether a matching metadata template exists (`list_metadata_templates`). If not, ask whether the user wants to create one (`create_metadata_template`) or extract with ad-hoc fields (`ai_extract_structured_from_fields`).
- Prefer `ai_qa_*` for simple Q&A or freeform lookups.
- Even when an extraction prompt is stated in natural language, prefer `ai_extract_structured_from_fields` with explicit typed fields (`key`, `type` in `string | float | date | enum | multiSelect`, and `options[].key` for enums).
- **Use `ask_default_agent`** when the task is interpretive/narrative ("assess the themes here") or open-ended across files, folders, and hubs rather than scoped to a known file set. Use **`ask_pro_agent`** only when the user explicitly requests deeper analysis. Check the live runtime tool schema for parameters before calling either agent tool.
- **Only use enhanced extract tools** (`*_enhanced`) if the user explicitly asks or the standard extraction result is incomplete — inform the user that enhanced extraction is more powerful on complex documents but costs more.
- Always tell the user which metadata template was used for a template-based extraction call.
- Always prefer `ai_qa_hub` over calling `ai_qa_multi_file` on every file inside a hub.

### Workflows

#### Single-file Q&A

1. Identify the target file (via `search_files_keyword` or a known file ID).
2. Call `ai_qa_single_file` with the file ID and a specific question.

Best suited for direct lookups ("what's the termination clause in this MSA") rather than open-ended summarization — keep questions narrow.

#### Cross-document Q&A across a known file set

1. Gather the relevant file IDs (e.g., a contract plus amendments and addenda) via search or folder listing.
2. Call `ai_qa_multi_file` with the file IDs and a comparative or synthesizing question ("which of these contracts has the shortest termination notice period").

Keep the file list deliberate and small rather than dumping in an entire folder, since relevance degrades as noise increases.

#### Structured extraction against a metadata template

1. Confirm the template exists via `list_metadata_templates` or `get_metadata_template_schema`; create one with `create_metadata_template` if needed.
2. Call `ai_extract_structured_from_metadata_template` (or the `_enhanced` variant only if requested) with the target `file_ids`, `scope` (plain string, e.g. `enterprise_<enterpriseId>`), and `template_key`.
3. Call `set_file_metadata` to persist the extracted structured values onto the file (note `set_file_metadata` uses enum `enterprise` or `global` for `scope`), making the data queryable later via `search_files_metadata`.

Extraction tools (`ai_extract_structured_*`) accept up to **50 files per call** (`file_ids` array, 1–50 items). For larger sets, chunk requests (files 1–50, then 51–100, etc.) and pace calls 1–2 seconds apart.

#### General-purpose Box AI agent query

1. Call `ask_default_agent` for queries that don't map cleanly to single-file, multi-file, or hub Q&A — e.g., open-ended reasoning across loosely structured Box content.
2. Use `ask_pro_agent` only if the user explicitly asks for deeper analysis.

Treat `ask_default_agent` / `ask_pro_agent` output as interpretive/narrative — better for high-level synthesis than pulling discrete structured fields, where `ai_extract_*` is the better fit.

### Error handling

- `ai_qa_*` tools use a file's text representation and support up to **1MB** of text rep. If a file exceeds 1MB, scope `ai_qa_multi_file` to fewer files, use `get_file_content` (up to 50MB) or targeted search/metadata queries, or tell the user the file exceeds Box AI's text-rep limit and provide the Box link. Do **not** attempt to download the file locally.
- `ai_extract_structured_*` tools have a **50-file max per request**. Chunk batches into groups of 50 (`1–50`, `51–100`, etc.) and space calls 1–2 seconds apart.

## Verification checklist

- **Retrieval quality:** Confirm search filters and candidate file IDs match the intended documents.
- **Answer grounding:** Cite specific file IDs, names, and Box URLs (and returned Box AI citations).
- **Access control:** Confirm the acting identity (`who_am_i`) sees only content authorized for the task.

## Primary docs

- Search reference: https://developer.box.com/reference/get-search/
- Box AI guides: https://developer.box.com/guides/box-ai/
- Box AI with objects: https://developer.box.com/guides/box-ai/use-box-ai-with-box-objects/

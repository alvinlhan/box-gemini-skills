# Bulk Operations (MCP)

## Table of Contents

- When this applies
- Constraints
- Workflow: inventory, classify, plan, execute, verify
- Step 1 — Inventory
- Step 2 — Classify (when content-based sorting is needed)
- Step 3 — Plan the target hierarchy
- Step 4 — Create folders
- Step 5 — Move or copy items
- Step 6 — Verify
- Rate-limit and backoff handling
- Partial failure recovery

## When this applies

Use this reference when a task involves organizing, moving, copying, or tagging more than a handful of Box files or folders via MCP:

- Organizing or reorganizing files across folders (by type, date, project, etc.)
- Moving content from a flat folder into a structured hierarchy
- Creating a folder tree for a classification or filing scheme
- Bulk-tagging files with metadata (`set_file_metadata`)
- Migrating content between folder structures

## Constraints

### Sobi write approvals and batch caps

In Sobi, **every mutative Box MCP call (`create_folder`, `move_file`, `move_folder`, `copy_file`, `copy_folder`, `set_file_metadata`) triggers a separate user approval**.

- **Prefer folder-level moves/copies (`move_folder` / `copy_folder`) over per-file moves (`move_file`)** whenever an entire subfolder can move together — moving a folder moves all of its contents recursively in one approval.
- **Ask the user to confirm and cap the batch size** before starting per-file writes (e.g., process in small batches of 5–15 files or confirm the exact subset to handle now).
- For very large reorganizations (hundreds of individual file moves), warn the user about per-write approval overhead and offer to organize only the top-priority subset or let them drag-and-drop in Box after you produce the inventory and classification table.

### Box API rate limits

Box enforces per-user and per-app rate limits. Sending requests too quickly returns `429 Too Many Requests`. Honor any retry delay and space Box AI calls at least 1–2 seconds apart.

### Folder name uniqueness

Box enforces unique names within a parent folder (`create_folder` names max 255 characters, no slashes). Creating a folder that already exists returns `409 Conflict`. Check existing folders via `list_folder_content_by_folder_id` first, or handle `409` by looking up the existing folder and reusing its ID.

## Workflow: inventory, classify, plan, execute, verify

Do not skip ahead — moving files without a verified plan leads to misplaced content that is painful to undo.

```
Inventory → Classify (if needed) → Plan & Cap Batch → Execute (serial) → Verify
```

Skip the classify step when files can be sorted by filename, extension, or existing metadata alone.

## Step 1 — Inventory

List everything in the source folder(s) using `list_folder_content_by_folder_id`. Paginate through all pages — do not assume a single page covers all items.

Capture each item's `id`, `name`, and `type` (`file` or `folder`) into a working list before proceeding.

## Step 2 — Classify (when content-based sorting is needed)

Skip this step if files can be categorized by filename, extension, or existing metadata. Use it when unstructured document content determines the category.

For the full content-understanding preference order (Box AI → metadata → `get_file_content`) and limits, see `references/ai-and-retrieval.md`.

### Sample-first strategy

Do not run Box AI on every file up front — Box AI has tighter rate limits than standard metadata reads:

1. **Pick a small sample** (5–10 representative files).
2. **Classify the sample** with `ai_extract_structured_from_fields` (supports up to 50 `file_ids` per call) or `ai_qa_single_file` to validate the category set and prompt.
3. **Check for cheaper signals.** Determine whether filename patterns, extensions, or existing metadata can sort remaining files without additional AI calls.
4. **Classify the remainder** in batches (up to 50 files per `ai_extract_structured_*` call), pacing calls at least 1–2 seconds apart.
5. **Record each classification** (`file_id` → category) as it completes so an interrupted run can resume cleanly.

### Handling classification failures

- If the first Box AI call returns `403` or feature-not-available, stop AI calls immediately and switch to metadata inspection or `get_file_content`.
- If an individual file fails (unsupported format, empty content, timeout), log it and continue.
- On `429`, wait for the retry period and retry the same request before moving on.

## Step 3 — Plan the target hierarchy

Decide the target folder structure before creating or moving anything:

1. Define the classification rule (filename pattern, extension, date, metadata, or Box AI category).
2. Map each inventoried item to its target folder path.
3. Identify which target folders already exist and which need to be created.
4. Check whether any target parent folder is externally shared (`get_folder_details` with `fields: ["shared_link"]` and `list_item_collaborations`, max 10 items per call).
5. Present the plan as a structured table — target folder path, parent ID, create yes/no, and item count/IDs — and **confirm the plan and batch size cap** with the user before executing any writes.

Example plan:

```
Target folder          | Parent ID | Needs creation | Items
-----------------------|-----------|----------------|------------------
/SEC Filings/10-K      | 0         | yes            | 3 files (111, 112, 113)
/SEC Filings/10-Q      | 0         | yes            | 3 files (211, 212, 213)
/Research/AI           | 0         | yes            | 2 files (311, 312)
```

## Step 4 — Create folders

Create target folders **top-down, parent before child**, calling `create_folder` serially and recording each returned folder ID for child folders and moves.

If `create_folder` returns `409 Conflict`, call `list_folder_content_by_folder_id` on the parent to get the existing folder's ID and reuse it instead of failing the plan.

## Step 5 — Move or copy items

- **When moving a whole folder:** call `move_folder` (or `copy_folder`) once rather than moving individual files inside it.
- **When moving individual files:** call `move_file` (or `copy_file`) serially up to the agreed batch cap.
- After each successful call, record the completed item ID. If an item fails (e.g., `409 Conflict` name collision in destination), log the item ID and error and continue with the remaining items — do not abort the entire batch.

## Step 6 — Verify

After writes complete:

1. Call `list_folder_content_by_folder_id` on each target folder and confirm the expected item IDs and counts.
2. Call `list_folder_content_by_folder_id` on the source folder and confirm only expected items remain.
3. Report a concise summary: completed count, target folder IDs/links, any failed items with errors, and any remaining items deferred by the batch cap.

## Rate-limit and backoff handling

When Box returns `429 Too Many Requests`:

1. Check for a retry delay (`Retry-After` if surfaced) and wait before retrying the same request.
2. Do not send other requests during the wait — limits are typically per-user.
3. Resume serial execution with slower pacing once the retry succeeds.

## Partial failure recovery

- Keep track of completed item IDs as you go.
- On interruption or partial failure, report what succeeded, what failed, and what remains.
- Resume using the inventory list minus completed IDs. Moving a file to a folder it is already in is safe/idempotent, though check current state (`get_file_details`) if an outcome was ambiguous.

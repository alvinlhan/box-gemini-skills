# Troubleshooting (Box MCP)

## Table of Contents

- Debugging checklist
- 401 or 403
- 404
- 409
- 429
- Search quality problems
- MCP server not connected
- MCP tool missing

## Debugging checklist

Before retrying or escalating, capture these facts:

- Acting user identity (from `who_am_i`)
- Exact MCP tool name and arguments
- Box object type (`file`, `folder`, `hub`, `metadata_template`) and ID
- Response status or error message

Most Box MCP failures reduce to one of these causes: wrong actor, wrong object ID, missing Box permission/entitlement, or an unverified schema parameter. For writes, resolve uncertainty before retrying — do not repeat a mutative call blindly.

## 401 or 403

- Acting user does not have access to the target file, folder, or feature (e.g., Box AI or Doc Gen not enabled on the account tier)
- OAuth session expired or missing required scope — ask the user to reconnect their Box connector in Sobi
- Box Admin Console disabled the specific MCP tool or Box AI capability for the enterprise
- Attempting an edit-only operation (e.g., `create_docgen_template`, `set_file_metadata`) on an item where the user only has Viewer access

## 404

- Wrong file, folder, hub, or template ID (or file ID passed where folder ID is expected)
- Object exists but is not visible to the acting Box user (`who_am_i`) — Box returns `404` for inaccessible items
- Metadata template key or `from` (`<scope>_<enterpriseId>.<templateKey>`) does not match `list_metadata_templates`

## 409

- File or folder name conflict on `create_folder`, `upload_file`, `move_file`, or `copy_file` — list the parent folder to find the existing item and reuse its ID or rename
- Metadata template or instance write conflicts with existing state — read current state (`get_file_details` / `get_metadata_template_schema`) before retrying

## 429

- Rate limit or burst traffic (especially on Box AI calls, which have tighter per-user limits)
- Excessive search or listing requests without pagination controls or `limit`
- Honor any surfaced `Retry-After` delay, wait before retrying, reduce concurrency, and pace Box AI calls at least 1–2 seconds apart (see `references/bulk-operations.md`)

## Search quality problems

- Missing `ancestor_folder_id`, `file_extensions`, owner, or metadata filters
- Querying as the wrong Box user (`who_am_i`) or expecting search to return folders the user cannot access
- Using `search_folders_by_name` without calling `get_folder_details` (`fields: ["path_collection", "owned_by", "modified_at"]`) to disambiguate duplicate folder names
- Missing `from` (`<scope>_<enterpriseId>.<templateKey>`) or unparameterized literal values in `search_files_metadata` (see `references/mcp-search.md`)

## MCP server not connected

Box MCP tools do not appear in the session, or `who_am_i` fails with an authentication error:

- Ask the user to disconnect and reconnect the Box connector in Sobi to refresh their OAuth session.
- Confirm the Box enterprise admin has enabled the Box MCP server integration in the Box Admin Console (https://docs.box.com/en/box-mcp/admin-controls).
- Verify whether the user is signed into the intended Box account via `who_am_i` after reconnecting.

## MCP tool missing

If the user asks for a Box capability whose MCP tool is not in your live tool list:

- Do **not** invent tool names or fall back to shell/CLI/`curl` commands.
- Check the documented Box MCP tool catalog at https://docs.box.com/en/box-mcp/tools.
- Explain clearly that the tool is not exposed in this Sobi connector (it may be disabled in the Box Admin Console at https://docs.box.com/en/box-mcp/admin-controls, not enabled in the Sobi connector configuration, or not available for the account).
- Give the user the direct Box link to perform the action in the Box web UI (for example, inviting named collaborators or uploading binary files).

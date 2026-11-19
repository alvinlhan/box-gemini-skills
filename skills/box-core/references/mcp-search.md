# Box Search

## MCP

### Tools

- Search: `search_folders_by_name`, `search_files_keyword`, `search_files_metadata`
- Metadata schema lookup: `list_metadata_templates`, `get_metadata_template_schema`
- Confirm before acting: `get_file_details`, `get_folder_details`

### Security guidelines

- Use least-privilege search scopes. When the user names a folder or project area, resolve it with `search_folders_by_name` and pass `ancestor_folder_id` to later searches instead of searching the whole Box account.
- Do not expose file contents from search results unless the user asks and content-display preferences are clear. Summarize search results by name, type, path, owner, modified date, and Box URL.
- Request only the fields needed for the task. Avoid pulling permissions, shared link, metadata, or collaboration fields unless needed for access review, sharing analysis, or disambiguation. Note field-casing differences: `search_files_keyword` uses camelCase field names (e.g., `pathCollection`, `sharedLink`), while `get_file_details` and `get_folder_details` use snake_case field names (e.g., `path_collection`, `shared_link`).

### Tool selection

- Use `search_folders_by_name` when the user names a folder, workspace, client, project, or department. Note that `search_folders_by_name` has no `fields` parameter and returns only `id`, `type`, and `name`. If multiple folders match, call `get_folder_details` on each candidate with `fields: ["path_collection", "owned_by", "modified_at"]` and ask the user to choose.
- Use `search_files_keyword` for natural-language, filename, extension, or broad content searches (max `limit`: 200). Scope with `ancestor_folder_id`, `file_extensions`, date ranges, and a low initial `limit`.
- Use `search_files_metadata` when the user's criteria map to structured metadata fields such as contract value, status, department, matter type, expiration date, or risk rating (max `limit`: 100).
- Use `list_metadata_templates` and `get_metadata_template_schema` before metadata search so you use the exact `from`, field keys, and field types.
- Use `get_file_details` or `get_folder_details` after search to confirm the item, path, classification, permissions, metadata, or sharing state before taking follow-up action.

### Search workflows

#### Folder-scoped search

1. Resolve the folder with `search_folders_by_name`.
2. If several folders match, call `get_folder_details` on each with `fields: ["path_collection", "owned_by", "modified_at"]` and let the user choose.
3. Search inside the selected folder by passing `ancestor_folder_id`.
4. Fetch `get_file_details` for the final target item before acting on it.

#### Metadata search

1. Call `list_metadata_templates` with `scope: "enterprise"` or `"global"`.
2. Call `get_metadata_template_schema` for the selected template to get exact field keys and types.
3. Call `search_files_metadata` with required `from` and `query`:
   - `from` has the format `<scope>_<enterpriseId>.<templateKey>` (for example, `enterprise_123456.contractTemplate`), taking the scope and template key from `list_metadata_templates`.
   - Note scope differences across tools: `list_metadata_templates` and `set_file_metadata` take enum `"enterprise"` or `"global"`, whereas `ai_extract_*_from_metadata_template` takes `scope` as a plain string such as `"enterprise_123456"`.
4. Use parameterized `query` placeholders (e.g., `execution_date >= :start AND status = :status`) and pass values in the `query_params` object (e.g., `{"start": "2026-01-01", "status": "active"}`) instead of embedding literal values directly in the query string. Verify date value formats against the template schema (e.g., RFC3339 vs. `YYYY-MM-DD`) with a small test query (`limit: 1`–`5`) before a full search.

#### Broad keyword search

1. Start with a small `limit` (max 200).
2. Add `file_extensions`, `ancestor_folder_id`, or RFC3339 date ranges when results are too broad.
3. Ask a clarifying question before expanding to enterprise-wide search if the query may expose too many unrelated results.

### Error handling

- If search returns no results, broaden one dimension at a time: remove date filters, remove file extension filters, search by partial folder name, or try `search_files_keyword` instead of `search_files_metadata`.
- If search returns too many results, narrow with `ancestor_folder_id`, `file_extensions`, date range, metadata filters, or a smaller `limit`.
- If `search_files_metadata` fails, verify `from` (`<scope>_<enterpriseId>.<templateKey>`), field keys, parameter names in `query_params`, and date formats using `list_metadata_templates` and `get_metadata_template_schema`.
- If permissions block access, explain that Box returns only accessible content and ask the user for a different folder, file, or a collaborator with access.
- If results are ambiguous, do not assume the intended file. Present the top candidates with name, type, path, modified date, owner, and Box URL.

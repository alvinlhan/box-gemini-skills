# Content Workflows

## Table of Contents

- Tools
- Security guidelines
- Tool selection
  - Upload constraints (`upload_file` and `upload_file_version`)
  - Reading file contents (`get_file_content`)
  - Choosing between `upload_file` and `upload_file_version`
- Workflows
  - Uploading files
  - Creating folders
  - Building folder trees
- Error handling

For collaborator audits and shared links, see `references/collaboration.md`. For multi-file reorganization and batch caps, see `references/bulk-operations.md`.

## Tools

- Upload / create text content: `upload_file`, `upload_file_version`, `create_box_note`
- Read / preview: `get_file_content`, `get_file_preview`, `get_preview_page`
- Inspect: `get_file_details`, `get_folder_details`, `list_folder_content_by_folder_id`
- Organize: `create_folder`, `move_file`, `move_folder`, `copy_file`, `copy_folder`
- Properties / metadata: `update_file_properties`, `update_folder_properties`, `set_file_metadata`, `set_folder_metadata`, `create_metadata_template`, `update_metadata_template`
- Comments / tasks: `create_file_comment`, `list_file_comments`, `list_tasks`

## Security guidelines

When using any write tool that affects files or folders, check whether the target object (or destination parent folder) has external collaborators or shared links:

- Use `get_file_details` or `get_folder_details` with `fields: ["shared_link"]` to check shared links.
- Use `list_item_collaborations` (max 10 items per call — chunk larger checks) to inspect collaborators.

If the object is externally or publicly shared, explicitly state who the external collaborators are and/or the shared link permissions before invoking the write tool. Note that in Sobi, every write tool call requires user approval — include whether the object is shared internally or externally so the user has full context when approving.

## Tool selection

### Upload constraints (`upload_file` and `upload_file_version`)

`upload_file` (`file_name`, `file_content`, `parent_folder_id`) and `upload_file_version` (`file_id`, `file_content`) take **text content only** (under 50MB). Only these file extensions preserve their type:

`txt`, `md`, `boxnote`, `html`, `svg`, `xml`, `csv`, `json`, `js`, `ts`, `py`, `sh`

Any other file extension is saved as a `.txt` file. For readable Markdown-backed notes, you can also use `create_box_note` (`file_name`, `file_content`, `parent_folder_id`).

You **cannot** upload binary files (`.pdf`, `.docx`, `.xlsx`, `.pptx`, images, audio, video, archives) or files over 50MB through Box MCP. Tell the user to upload those files directly in Box and provide them the destination folder link.

### Reading file contents (`get_file_content`)

Use `get_file_content` to pull the text representation of a file (up to 50MB file size limit).

If `get_file_content` fails or the file has no text representation, tell the user the file cannot be read directly here and share the Box link (`url` from search results or `get_file_details`). Do **not** attempt to download the file locally or run `curl`.

### Choosing between `upload_file` and `upload_file_version`

If a user is iterating on a text file and uploading it multiple times to Box in the same session, use `upload_file_version` to upload a new version instead of uploading a duplicate file with a different name.

## Workflows

### Uploading files

Confirm which destination folder the user wants to upload the file to. If the folder name was provided ("Upload this file to the Sales Folder"), resolve it via `search_folders_by_name` (disambiguating with `get_folder_details` if multiple matches exist) and confirm if ambiguous. If no specific folder was requested, confirm whether to upload to root (`parent_folder_id: "0"`) or a named folder; you can move it afterward with `move_file`.

### Creating folders

Subfolders in Box inherit the sharing and collaboration of their parent folder. Before calling `create_folder` (names max 255 characters, no slashes), check whether the parent folder has external collaborators or shared links. If the parent is externally or publicly shared, disclose the external collaborators or shared link permissions before creating the folder. Handle `409 Conflict` by listing the parent folder and reusing the existing folder ID.

### Building folder trees

When building multi-folder hierarchies, present the full folder tree diagram in chat and get confirmation first. Because every `create_folder` call is a separate Sobi write approval, confirm the exact subset of folders to create right now before executing. Create parent folders before child folders (top-down) and record each returned folder ID.

## Error handling

- Prefer `get_file_content` and sharing the Box `url` link when referencing a file. Only call `get_file_preview` if the user explicitly asks for an interactive preview (`get_file_preview` works on documents up to 3MB and only renders on clients that support MCP Apps widgets).
- `get_file_content` pulls the text representation of a file and has a 50MB max file size limit.
- `update_file_properties` / `update_folder_properties`: name max 255 characters, description max 256 characters, up to 100 tags.
- `create_file_comment`: `file_id` and `message` (max 1,000 characters); comments do not trigger automatic @mention notifications.
- `set_file_metadata` / `set_folder_metadata`: read the template schema first (`list_metadata_templates` + `get_metadata_template_schema`) and verify field keys and types before writing.

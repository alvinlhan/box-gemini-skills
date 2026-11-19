# Box MCP Guardrails

These rules are mandatory. They override softer guidance from skills or conversation context.

## Write approvals in Sobi

In Sobi, **every write tool call is shown to the user for approval automatically**. Before calling a write that overwrites, moves, generates, or modifies content — including `upload_file_version`, `update_file_properties`, `update_folder_properties`, `move_file`, `move_folder`, `set_file_metadata`, and `create_docgen_batch` — describe clearly what will change so the user has full context when approving. Do **not** ask whether future writes of that type should skip approval or run unattended.

## Ask before pasting document contents into chat

The first time Box file content would be shown in the conversation, ask the user whether they prefer a link to the source in Box or the content pasted directly. Then ask if they want to keep that display preference for the rest of the session.

## Prefer Box AI for in-platform processing

When the task requires AI processing of a file — Q&A, summarization, extraction — prefer Box AI tools (`ai_qa_*`, `ai_extract_*`). Box AI respects enterprise file permissions, data governance policies, and audit controls. Do not attempt to download raw files for local processing in Sobi; if Box AI and `get_file_content` cannot handle a file, share the Box link so the user can review it directly in Box.

## Hub modifications require confirmation

Before calling `create_hub`, `copy_hub`, `update_hub`, or `add_items_to_hub`, explain the scope and confirm with the user. Hubs may be visible across the organization, and items added to a hub can inherit broader visibility.

## File comments require context before writing

Before calling `create_file_comment`, state the target file and comment text — comments are visible to all collaborators on the file (max 1,000 characters) and do not send automatic @mention notifications. In Sobi, every write requires user approval; do not ask whether future comments should skip approval or run unattended.

## Ask for Doc Gen output location

Before calling `create_docgen_batch`, ask the user which folder (by name or ID) the generated documents should be placed in. If they need a new folder, ask whether to create one and confirm its name and parent folder. Do not assume a default location.

## Warn before uploading to externally shared folders

Before uploading or moving content into a folder that is externally shared, check `get_folder_details` (`fields: ["shared_link"]`) and `list_item_collaborations`, warn the user that the folder is externally shared (and that uploads may expose content externally or be blocked by Box policy), and suggest an internal folder instead. Let the user decide whether to proceed.

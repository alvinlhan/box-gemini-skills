# Collaboration and Sharing

## Table of Contents

- Tools
- Box Collaborator Roles (reference)
- Security guidelines
- Managing collaborators (read-only on Sobi)
- Shared links workflows
- Error handling
- Primary docs

## Tools

- Collaborations: `list_item_collaborations` (read-only). Takes an `items` array of `{id, type: "file" | "folder"}` with **at most 10 items per call** — chunk larger folder-tree audits into batches of 10.
- Shared links: `add_file_shared_link`, `add_folder_shared_link`

## Box Collaborator Roles (reference)

Use this matrix as reference when explaining Box roles or proposing an access table for a folder owner to apply in Box:

| Capability | Co-owner | Editor | Viewer Uploader | Previewer Uploader | Viewer | Previewer | Uploader |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Upload | ✓ | ✓ | ✓ | ✓ | — | — | ✓ |
| Download | ✓ | ✓ | ✓ | — | ✓ | — | — |
| Preview | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| Edit & rename | ✓ | ✓ | — | — | — | — | — |
| Delete & move | ✓ | ✓ | — | — | — | — | — |
| Invite collaborators | ✓ | ✓ | — | — | — | — | — |
| Advanced folder settings | ✓ | — | — | — | — | — | — |

**Plan note:** Editor and Viewer are available on all accounts. All other roles require Business/Enterprise and apply to folders only.

## Security guidelines

- In Sobi, **every write tool call is shown to the user for approval automatically**. Never tell the user a write can proceed without confirmation.
- Before generating or updating an external or open shared link, describe what is inside the file or folder and confirm audience, password, and expiration settings with the user.
- Treat folder shared links with extra caution — they expose every file inside the folder, including content added later.
- Before overwriting an existing shared link, read the current settings first (`get_file_details` or `get_folder_details` with `fields: ["shared_link"]`), show the existing settings to the user, and confirm whether they want to replace them.

## Managing collaborators (read-only on Sobi)

You **cannot** invite users or groups or change collaborator roles (including Co-owner) through Box MCP on this connector — only read-only auditing via `list_item_collaborations` is exposed.

When a user asks to invite collaborators or change roles:

1. Use `list_item_collaborations` (chunked at 10 items per call) to inspect current access if helpful.
2. Present the proposed collaborator/role mapping as a clear table following least privilege.
3. Ask the user (or folder owner) to apply the invite or role change directly in Box, and provide the direct Box link to the file or folder.
4. Once they confirm the change is done, re-run `list_item_collaborations` to verify effective access.

## Shared links workflows

### Create or update a shared link on a file or folder

1. Read current state first with `get_file_details` or `get_folder_details` (`fields: ["shared_link"]`) so you know whether a link already exists.
2. Check the live runtime tool schema for `add_file_shared_link` or `add_folder_shared_link` to verify the exact parameter names and allowed enum values (such as access level, password, expiration, or custom URL slug) before calling.
3. Check whether the enterprise enforces password or expiration policies before assuming an open link will be accepted.
4. If tightening access on an existing link (e.g., from open to company-only), confirm that no external party currently depends on the old link.
5. Verify the resulting link by calling `get_file_details` / `get_folder_details` (`fields: ["shared_link"]`) after the write.

## Error handling

- **Inviting users/groups or changing roles (including Co-owner):** Not available via Box MCP on Sobi. Direct the user to manage collaborators on box.com in the browser and provide the item link.
- **Enterprise link policies / unsupported shared-link options:** If `add_file_shared_link` or `add_folder_shared_link` rejects a setting (for example, a permanent open link when the enterprise requires an expiration window, or an option not exposed in the runtime schema), direct the user to configure the link directly in Box and provide the item link.

## Primary docs

- Collaborations: https://developer.box.com/reference/post-collaborations/
- Shared links (files): https://developer.box.com/reference/put-files-id/
- Shared links (folders): https://developer.box.com/reference/put-folders-id/

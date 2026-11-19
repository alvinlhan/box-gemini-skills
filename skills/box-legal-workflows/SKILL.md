---
name: box-legal-workflows
description: "Shared legal reference loaded by box-legal-workflows-contract, box-legal-workflows-intake, and box-legal-workflows-ma: Box collaborator roles, Box AI boundaries, and confirmation patterns. Do not trigger on its own."
---

# Shared Legal Concepts

> **PREREQUISITE:** Use the `box-core` skill for Box MCP tool mechanics, guardrails, and base workflows.

Shared building blocks used by the legal skills (`box-legal-workflows-ma`, `box-legal-workflows-intake`, `box-legal-workflows-contract`): Box collaborator role definitions, the boundaries of where Box AI must not be the decision-maker, and reusable confirmation phrasings. Risk frameworks, metadata fields, workflows, and decision-transparency requirements live in the individual legal skills.

## Box capability references

Reach for these for the underlying Box tool mechanics rather than restating them here:

- The `box-core` skill's `references/collaboration.md` — collaborator role capabilities, read-only collaboration audits (`list_item_collaborations`), shared links, and external-sharing rules
- The `box-core` skill's `references/ai-and-retrieval.md` — Box AI Q&A, extraction, and structured metadata tools, with pacing/limits/citations

---

## Box Collaboration Roles

**[CONFIRM: What permission level is appropriate?]**

For the full breakdown of each Box collaborator role (Co-owner, Editor, Viewer Uploader, Previewer Uploader, Viewer, Previewer, Uploader) and the exact capabilities each grants, see the role matrix in the `box-core` skill's `references/collaboration.md`. Use that matrix to recommend which role fits each collaborator; because named collaborator invites cannot be executed via Box MCP on Sobi, present the proposed role table for the folder owner to apply in Box and verify afterward with `list_item_collaborations`.

Apply least privilege: default to the most restrictive role that still lets the collaborator do their job, prefer specific folders over root access, and set expiration dates on external sharing.

---

## Box AI Boundaries

Box AI informs; a human attorney decides. Do **NOT** use Box AI for:

- Final legal advice or decisions (human attorney only)
- Access-control decisions (human approves permissions)
- Client conflict checks (use the firm's conflict system)
- Privilege determinations (attorney judgment)
- Settlement negotiations or strategy

Box AI is appropriate for *informing* humans — completeness checks, risk-factor flagging, metadata extraction, contract comparison, and due-diligence Q&A — always with citations surfaced and a human making the final call.

For details on the Box AI tools available and how to use them, see the `box-core` skill's `references/ai-and-retrieval.md`.

---

## Common Confirmation Patterns

Reusable phrasings for the human-in-the-loop confirmations the legal skills require.

### Risk Rating
"Here are the factors I found: [...], with citations from Box. Under your firm's criteria, which rating applies?" (The agent presents facts; the firm's criteria and attorney determine the rating.)

### Permissions
"Here is the recommended access mapping: [person] → [role] on [folder/file] (grants [permissions]). Named collaborator invites must be applied in Box — please invite them using the folder link, or confirm if you want a scoped shared link instead, and I can verify access with `list_item_collaborations` afterward."

### Routing
"Based on [practice area/risk/type], I recommend routing to [attorney]. Correct, or assign to someone else?"

### Document Completeness
"Firm requires: [list]. Found: [list]. Missing: [list]. Proceed with assessment?"

### Metadata Template
"Do you have a Box metadata template for [type]? If yes, scope and template key?"

### Thresholds
"What is your firm's threshold for [matter value/expiration alert/risk escalation]?"

### External Sharing
"Before sharing with [external party], confirm: (1) Permission level? (2) Folders? (3) Expiration? (4) Owner-applied Box invite or scoped shared link?"

---
name: artemis-setup
description: Connects Claude Code to Artemis and checks access with a small project read. Use when setting up Artemis or diagnosing missing Artemis tools.
---

# Connect Artemis

Connect only when the user requests setup.
Use an existing Artemis connection when one is available.
MCP, the Model Context Protocol, lets the assistant call Artemis tools with the user's permission.

## Workflow

1. Check whether Artemis tools are available in this session.
   If they are, skip installation and continue with the project read below.
2. If not, inspect `claude mcp list` from the terminal.
   For an installed plugin, sign in with `claude mcp login plugin:artemis:artemis`.
   For an existing direct connection, use `claude mcp login artemis`.
   If both appear, do not add another connection or remove either without asking.
3. If no connection exists, choose one method only.
   The plugin installs the connection and all five skills:

   ```sh
   claude plugin marketplace add monalee-inc/artemis-skills
   claude plugin install artemis@artemis
   claude mcp login plugin:artemis:artemis
   ```

   The direct method installs only the connection:

   ```sh
   claude mcp add --transport http --scope user artemis https://sales-api.artemis.solar/external/v2/mcp
   claude mcp login artemis
   ```

4. Let the user sign in through the browser and check the account and organization.
   Suggest **Read Only**, then **Allow access**.
   Never ask for passwords, API keys, tokens, or browser sign-in URLs in chat.
5. Check `claude mcp list` for **Connected**.
   If tools remain unavailable in this session, ask the user to start a new Claude Code session and stop.
6. Call `list_projects` with `limit: 3` through the available Artemis tool.
   Report success only after a successful tool response, including an empty list.
   Show up to three names, addresses and statuses, then suggest pipeline review, call preparation, or roof measurements.

For an error, report the failed step and safe error text without credentials.
Do not read credential files or bypass permissions to diagnose sign-in.
Do not write project data to test access.
Codex and Cursor CLI setup awaits tested sign-in fixes.
Do not substitute Claude app, ChatGPT, or Cursor desktop instructions.

## Artemis safety rules

- Use only the connection's available tools and permissions.
  Browser consent grants access, not approval for an action.
- Before a write, summarize its target and change, then get an explicit yes in the current conversation.
  Follow any additional confirmation steps in the tool description.
- A retry with the same intent, identical inputs, and the same idempotency key needs no new approval.
  An idempotency key identifies one write so retrying does not create a second operation.
- Reuse `headers.etag` from a create or update as `ifMatch` for the next write.
  An ETag is the resource's version marker.
  Read the resource only when no ETag is available.
  After a `412` conflict, read it again before retrying.
- Poll the read tool while background work is `pending` or `processing`.
  Never create another resource to retry that work.
  Report unfinished or failed work honestly.
- Resolve an address, project name, or customer name/email with `list_projects` and `filters.search`.
  Use the selected result's `id` as `projectId`.
  Ask the user to choose among matches or refine an empty search.
  Never invent an identifier or require the user to know one.
- Prefer `open_project_document_upload` for a local file when available.
  Chat attachments are not automatically available to Artemis.
  Never put file bytes in tool arguments.

Details: [Artemis setup guide](https://docs.artemis.solar/overview/ai-agents).

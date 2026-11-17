---
name: artemis-roof-measurements
description: Reads saved roof measurements for an Artemis project, including squares, pitch, and edge lengths. Use when preparing a roofing estimate or reviewing an existing roof.
---

# Read roof measurements

Example request:

```text
Using Artemis, give me the roof measurements for the project at [address]:
total squares, predominant pitch, and the ridge, hip, valley and rake lengths.
```

This workflow reads the saved design.
It does not generate a new design, change roof geometry, or create an estimate.

## Workflow

1. Check that Artemis tools are available.
   If not, direct the user to `artemis-setup` and stop.
2. Find the project with `list_projects` and `filters.search`.
   Ask the user to select a match or refine an empty search.
3. Call `get_project_design` with the selected `projectId`.
   Start with the summary response, without requesting full geometry.
   Check `autoDesignStatus` before interpreting measurements.
4. If work is pending, poll the read tool and report its actual state.
   If design generation failed, report the failure rather than treating missing totals as zero.
   If `roofing` is null or unavailable, say measurements are unavailable to this connection.
   Do not infer an entitlement problem from a null value alone.
5. Report `roofing.totalSquares`, `predominantPitch`, and `edges.ridgeFt`, `hipFt`, `valleyFt`, and `rakeFt`.
   A roofing square means 100 square feet.
   Keep lengths in feet and preserve the returned pitch notation.
   Include the project address and saved-design date when available.
   Label a missing pitch or measurement unavailable, never zero by assumption.

Roof measurements describe the saved design, not unsaved edits in an editor.
Do not calculate replacement measurements from a picture.
Do not promise roof previews or a CSV export through MCP, the assistant tool connection.
Those operations are not exposed by the current Artemis MCP tool catalog.

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

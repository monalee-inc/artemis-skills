---
name: artemis-call-prep
description: Summarizes an Artemis project before a customer call, including status, system size, pricing, and financing options. Use for call preparation or a project briefing.
---

# Prepare for a call

Example request:

```text
Using Artemis, summarize the project at [address]: status history, system size, price, and financing options.
Keep it short enough to read before a call.
```

This workflow reads one project and drafts a briefing.
It does not change pricing, select financing, or submit an application.

## Workflow

1. Check that Artemis tools are available.
   If not, direct the user to `artemis-setup` and stop.
2. Search with `list_projects` and `filters.search` using the supplied address or project name.
   Ask the user to choose when several projects match.
   If none match, ask for a more specific search and stop.
3. Use the selected project ID with `get_project`, `list_project_status_history`, and `get_project_design`.
   Read `get_project_customer` only if the briefing needs customer details.
4. Read `get_project_pricing` and `list_project_financing_options` when those tools are available.
   Preserve the returned currency, units, option names, and term lengths.
   Do not invent monthly payments, eligibility, approvals, rates, or savings.
5. Return a short briefing: project and address, recent status history, system size, price, financing choices, and suggested questions.
   Label recommendations separately from recorded facts.
   State missing permissions, unavailable values, pending design work, or failed reads in the relevant section.

Follow pagination when needed for the requested history or financing list.
Label a partial list rather than claiming to have compared every option.
Pending design work is not a completed zero-sized system.
Share only details needed for this call.

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

---
name: artemis-pipeline-review
description: Reviews an Artemis sales pipeline and ranks follow-ups from project facts. Use for recent projects, pipeline reviews, or deciding whom to follow up with.
---

# Review a pipeline

Example request:

```text
Using Artemis, list my 10 most recently updated projects with customer name, address and status.
Then tell me which 3 need a follow-up first, and why.
```

This workflow reads projects and recommends actions.
It does not send follow-ups or change statuses.

## Workflow

1. Check that Artemis tools are available.
   If not, direct the user to `artemis-setup` and stop.
2. Establish whose projects and which time window to review.
   For “my projects,” resolve the user from available team or user records before setting `filters.assignedSalesRepId`.
   If that lookup is unavailable or ambiguous, ask which team or projects to review.
   Do not silently equate all visible projects with the user's assigned projects.
3. Call `list_projects` with supported filters for the agreed scope.
   Follow `page.nextCursor` to cover that scope before claiming the ten most recently updated projects.
   Sort the collected records by `updatedAt`, newest first.
   The default list order is creation time, not update time.
   If paging stops early, label the result partial and state the reviewed scope.
4. Read `get_project_customer` for the selected projects when customer names are needed.
   Use `list_project_status_history` where it clarifies a follow-up recommendation.
   Keep missing fields unavailable rather than guessing them.
5. Show up to ten projects with customer, address, status, and latest update time.
   Recommend up to three follow-ups with the supporting project facts.
   Distinguish recommendations from recorded facts.
   No status change is not proof that no one contacted the customer.

If tools deny access, report what could not be checked rather than treating hidden records as absent.
Do not export customer data to another service or create a recurring task as part of this review.

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

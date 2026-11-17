---
name: artemis-team-check
description: Checks Artemis installer readiness, active team pricing, and pending invites. Use for an administrator's team review or to identify missing setup.
---

# Check team setup

Example request:

```text
Using Artemis, check which installers are not ready and list what each one is missing.
```

Other supported requests:

```text
Using Artemis, list every team with its active pricing, and point out teams that have no pricing set.
```

```text
Using Artemis, list invites that are still pending, with the person and the date sent.
```

This workflow reads setup information.
It does not assign pricing, complete onboarding, send invites, or resend invites.

## Workflow

1. Check the available Artemis tools and establish which check the user wants.
   If no Artemis tools are available, direct the user to `artemis-setup` and stop.
   If only admin tools are missing, explain that the connection cannot perform that check.
   Never claim a team is missing setup because access was denied.
2. For installer readiness, call `list_installers`, then `get_installer_readiness` for the requested installers.
   Show each installer's returned readiness state and missing requirements.
   Do not invent additional requirements.
3. For team pricing, call `list_teams` and each team's `list_team_pricings` using the current tool schema.
   Distinguish available pricing catalogs from an enabled catalog with an `activePricingVersion`.
   Name enabled catalogs and active versions without treating a draft as active.
   Call `get_team_pricing` only if the list lacks details needed for the conclusion.
4. For invites, call `list_invites` and show returned pending invite records.
   Use the returned email when no name is available.
   Label `createdAt` as the creation date, not confirmed delivery or the last resend date.
5. Follow `page.nextCursor` for lists before claiming to cover every accessible record.
   Report partial checks, denied access, and failed reads separately from missing setup.
   Summarize findings and suggested next steps without performing writes.

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

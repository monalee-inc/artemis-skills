# Artemis skills

Connect Claude Code to Artemis to review projects, prepare for calls, read roof measurements, and check team setup.
This package includes five skills and the Artemis MCP server connection.
MCP, the Model Context Protocol, lets an assistant use Artemis tools with your permission.

## Connect in Claude Code

Use Claude Code in your terminal, not the Claude web or desktop app.
You need a working Claude Code account and an Artemis account.
Your organization may restrict which plugins you can install.

Choose either the plugin or the direct connection, not both.
If Artemis is already connected, keep that connection rather than adding another.

### Plugin: connection and five skills

Run these commands in your terminal:

```sh
claude plugin marketplace add monalee-inc/artemis-skills
claude plugin install artemis@artemis
claude mcp login plugin:artemis:artemis
```

In the browser, sign in to Artemis.
Check the account and organization shown under **Signed in as**.
Choose **Read Only** to start without granting changes, then select **Allow access**.
Never paste your password, an API key, or a sign-in token into a chat.

The browser should show **Authentication successful**.
Check the connection in your terminal:

```sh
claude mcp list
```

Look for `plugin:artemis:artemis` with **Connected**.
Then start a new Claude Code session and ask:

```text
Using Artemis, list up to 3 projects I can access.
Show their names, addresses and statuses. Do not change anything.
```

A successful answer must come from an Artemis tool call, not an example.
An empty list can be a successful read when your account has no visible projects.
If tools are unavailable, stop and report the connection error instead of inventing project data.

### Direct connection: no skills installed

The direct connection works without this repository:

```sh
claude mcp add --transport http --scope user artemis https://sales-api.artemis.solar/external/v2/mcp
claude mcp login artemis
claude mcp list
```

Follow the same browser steps, then look for `artemis` with **Connected**.
Start a new Claude Code session and use the same first question.

## Skills

| Command in Claude Code | What it does |
| --- | --- |
| `/artemis:artemis-setup` | Checks the connection and reads up to three projects. |
| `/artemis:artemis-pipeline-review` | Reviews a rep's projects and suggests three follow-ups. |
| `/artemis:artemis-call-prep` | Summarizes one project before a customer call. |
| `/artemis:artemis-roof-measurements` | Reads saved roof measurements, when available to your organization. |
| `/artemis:artemis-team-check` | Checks installer readiness, active team pricing, or pending invites. |

These workflows read data and suggest next steps.
They do not send messages, change designs, assign pricing, or resend invites.
Tool availability depends on your Artemis permissions and the access you grant.
Missing access is not proof that a record does not exist.

## Sign in again or disconnect

If the plugin connection needs authentication, run:

```sh
claude mcp login plugin:artemis:artemis
```

For a direct connection, use `claude mcp login artemis` instead.

To disconnect the plugin, sign out before uninstalling:

```sh
claude mcp logout plugin:artemis:artemis
claude plugin uninstall artemis@artemis
```

To disconnect the direct connection:

```sh
claude mcp logout artemis
claude mcp remove --scope user artemis
```

Start a new Claude Code session after disconnecting.
Removing a configuration alone is not a guarantee that issued access tokens were revoked.
Your Artemis account and projects remain unchanged.

## Scope of verification

Claude Code 2.1.325 completed direct and plugin sign-in against an Artemis test environment on November 14, 2026.
The plugin test used a local marketplace, not this public repository.
A read using the resulting credentials succeeded.
These five skills have not been tested in model conversations.
Production sign-in, live-session reload, and public-repository installation have not been tested with this package.

Codex and Cursor CLI guides will follow after their sign-in fixes are released and tested.
Claude app and ChatGPT setup are not covered by this guide.
Cursor desktop is outside this first version.
Installing skill files in another assistant does not connect that assistant to Artemis.

## Documentation and privacy

- [Artemis setup guide](https://docs.artemis.solar/overview/ai-agents), the live Claude Code guide (HTTP 200 on November 17, 2026).
- [Artemis documentation](https://docs.artemis.solar).
- [Privacy policy](https://artemispower.com/privacy-policy).
- [Terms of use](https://artemispower.com/terms-of-use).
- Support: [support@artemis.solar](mailto:support@artemis.solar).

The live setup guide still says this repository is unavailable.
Use this README for plugin installation.
The assistant-tab guide merged into `dev` in https://github.com/monalee-inc/artemis/pull/7971 but is not yet live in production.

Artemis receives tool requests, not your full conversation automatically.
Your assistant receives the records its tool calls return.
Share only the information needed for your task.
The package adds no separate collection service.
Existing Artemis usage and billing rules still apply.

## Validate the package

From the repository root, run:

```sh
claude plugin validate .
claude plugin validate --strict --profile directory .
```

Passing the first command confirms package structure, not sign-in or skill behavior.
Directory validation is not directory approval.
Strict directory validation reports `ICON_MISSING` because this package does not include a directory icon.
An icon is required before directory submission, not for this public repository.

## License

See [LICENSE](LICENSE).
Released under the MIT License.

# Limitry from a terminal: the `limitry` CLI

For a coding agent with a shell (Claude Code, Codex or another coding agent). Read SKILL.md first.

- **Install and sign in:** `npm install -g @limitry/cli`, then `limitry login` (the person approves it in the browser). Without a browser, the person sets `LIMITRY_TOKEN` to a personal access token.
- Add `--json` to EVERY command: output is the API's JSON, errors are JSON on stderr, exit code 0 means success.
- Find the workspace first (`limitry workspace list --json`); pass `--workspace <slug>`.
- Pass `--yes` to a command that cannot be undone only after the person's explicit go-ahead.
- Next page: `--cursor <nextCursor>`. Retrying a write: `--idempotency-key <key>`, the same both times; the CLI already retries a failed attempt with the same key.
- **Approvals:** `limitry workspace approval-actions list --json`; `limitry workspace approval-requests create --action <id> --input '{…}' --reason "…" --json` (show the person the `approveUrl`). For an action that creates a secret, `limitry workspace approval-requests redeem <id> --wait --write-env .env` waits for the decision and writes the secret into the file WITHOUT printing it.

## Commands

| Command | MCP tool | What it does |
| --- | --- | --- |
| `limitry check <kind>:<id> <action>` | `checks_create` | Check whether a subject may act |
| `limitry limits archive <limitId>` | `limits_archive` | Archive a limit |
| `limitry limits create` | `limits_create` | Create a limit |
| `limitry limits credits list <limitId>` | `limits_credits_list` | List a balance's credits |
| `limitry limits get <limitId>` | `limits_get` | Get a limit |
| `limitry limits list` | `limits_list` | List limits |
| `limitry limits restore <limitId>` | `limits_restore` | Restore an archived limit |
| `limitry sync [file]` | `limits_sync` | Sync limits from a list |
| `limitry limits update <limitId>` | `limits_update` | Change a limit |
| `limitry reservations commit <reservationId>` | `reservations_commit` | Commit a reservation |
| `limitry reservations extend <reservationId>` | `reservations_extend` | Extend a reservation |
| `limitry reservations release <reservationId>` | `reservations_release` | Release a reservation |
| `limitry subjects get <subjectKind> <subjectId>` | `subjects_get` | Get a subject |
| `limitry subjects reset <subjectKind> <subjectId>` | `subjects_reset` | Reset a subject's limits |
| `limitry subjects update <subjectKind> <subjectId>` | `subjects_update` | Set a subject's billing date |
| `limitry usage query` | `usage_query` | Get a limit's usage over time |
| `limitry usage standing` | `usage_standing` | Get where a subject stands |
| `limitry usage subjects` | `usage_subjects` | List who is using a limit now |
| `limitry user me` | — | Get the current user |
| `limitry user revoke-token` | — | Revoke the token making this request |
| `limitry workspace approval-actions list` | `workspace_approval_actions_list` | List the actions you can ask approval for |
| `limitry workspace approval-requests create` | `workspace_approval_requests_create` | Ask a person to approve an action |
| `limitry workspace approval-requests get <requestId>` | `workspace_approval_requests_get` | Check an approval request |
| `limitry workspace audit-events list` | `workspace_audit_events_list` | List workspace audit events |
| `limitry workspace event-types list` | `workspace_event_types_list` | List event types |
| `limitry workspace get` | `workspace_get` | Get the current workspace |
| `limitry workspace invitations list` | `workspace_invitations_list` | List pending invitations |
| `limitry workspace members list` | `workspace_members_list` | List workspace members |
| `limitry workspace teams list` | `workspace_teams_list` | List teams |
| `limitry workspace usage` | `workspace_usage` | Get your plan's usage this period |
| `limitry workspace webhook-deliveries list <endpointId>` | `workspace_webhook_deliveries_list` | List an endpoint's deliveries |
| `limitry workspace webhook-deliveries redeliver <endpointId> <deliveryId>` | `workspace_webhook_deliveries_redeliver` | Redeliver an event |
| `limitry workspace webhook-endpoints get <endpointId>` | `workspace_webhook_endpoints_get` | Get a webhook endpoint |
| `limitry workspace webhook-endpoints list` | `workspace_webhook_endpoints_list` | List webhook endpoints |
| `limitry workspace webhook-endpoints test <endpointId>` | `workspace_webhook_endpoints_test` | Send a test event |

`limitry <command> --help` lists each command's inputs; `limitry api <method> <path>` calls any endpoint directly.

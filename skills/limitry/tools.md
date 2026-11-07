# Limitry through its tools: the MCP server

For an agent with Limitry's MCP tools: Claude, ChatGPT, an always-on agent or another client connected to `https://mcp.limitry.com/mcp`. Connecting opened the person's browser to sign in, pick ONE workspace and the access, so the tools you see are exactly what they allowed. Read SKILL.md first.

- A tool that changes something takes an `idempotencyKey`: pass the same one when you retry it. The tools already retry a failed attempt with the same key.
- When a tool is missing, the person did not grant that access. Tell them what it would take; do not look for another way.
- **Approvals:** `workspace_approval_actions_list` shows what can be asked and its input; `workspace_approval_requests_create` files it with your reason. Show the person the returned `approveUrl`; `approval_requests_get` tells you when it is decided.

## Tools

| Tool | What it does |
| --- | --- |
| `checks_create` | Check whether a subject may act |
| `limits_archive` | Archive a limit |
| `limits_create` | Create a limit |
| `limits_credits_list` | List a balance's credits |
| `limits_get` | Get a limit |
| `limits_list` | List limits |
| `limits_restore` | Restore an archived limit |
| `limits_sync` | Sync limits from a list |
| `limits_update` | Change a limit |
| `reservations_commit` | Commit a reservation |
| `reservations_extend` | Extend a reservation |
| `reservations_release` | Release a reservation |
| `subjects_get` | Get a subject |
| `subjects_reset` | Reset a subject's limits |
| `subjects_update` | Set a subject's billing date |
| `usage_query` | Get a limit's usage over time |
| `usage_standing` | Get where a subject stands |
| `usage_subjects` | List who is using a limit now |
| `workspace_approval_actions_list` | List the actions you can ask approval for |
| `workspace_approval_requests_create` | Ask a person to approve an action |
| `workspace_approval_requests_get` | Check an approval request |
| `workspace_audit_events_list` | List workspace audit events |
| `workspace_event_types_list` | List event types |
| `workspace_get` | Get the current workspace |
| `workspace_invitations_list` | List pending invitations |
| `workspace_members_list` | List workspace members |
| `workspace_teams_list` | List teams |
| `workspace_usage` | Get your plan's usage this period |
| `workspace_webhook_deliveries_list` | List an endpoint's deliveries |
| `workspace_webhook_deliveries_redeliver` | Redeliver an event |
| `workspace_webhook_endpoints_get` | Get a webhook endpoint |
| `workspace_webhook_endpoints_list` | List webhook endpoints |
| `workspace_webhook_endpoints_test` | Send a test event |

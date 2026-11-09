---
name: limitry
description: Use when working with Limitry (limitry.com): reading or changing a workspace in Limitry, from a chat app such as Claude or ChatGPT through its MCP tools, from a terminal through the `limitry` CLI or through its API. Covers signing in, choosing the workspace, every operation, approvals and errors.
---

# Limitry

Limitry is the limits, credits and budgets layer for the software you build: give each of your users, teams, tenants, API keys or agents an allowance, and before any costly request one call tells your code whether it may happen.

Everything belongs to a **workspace**. You act as the person who signed
you in, with their role in that workspace and only the access they
granted. If they could not do something in the app, neither can you.

## Where you are

- **With Limitry's tools** (Claude, ChatGPT, or an agent that runs on its own; tool names like `workspace_get`): read [tools.md](tools.md) (online: https://limitry.com/docs/skill/tools.md). Use only the tools there.
- **In a terminal** (Claude Code, Codex or another coding agent): read [cli.md](cli.md) (online: https://limitry.com/docs/skill/cli.md). If your client also has the MCP tools, prefer them.
- **In code you are writing**: the HTTP API `https://api.limitry.com/v1` with `Authorization: Bearer <token>`; the OpenAPI document is `https://api.limitry.com/v1/openapi.json`.

## Rules

- Find out which workspace the person means, and ask when it is unclear. Never guess one.
- Read before you change anything, and say what you will change before you do.
- An operation that cannot be undone (a delete) needs the person's explicit go-ahead.
- When nobody is there to answer (you run on a schedule, in the background, or while the person is away): never wait at a sign-in or for an approval. File the approval request, carry on with what does not depend on it, and report what is waiting and why. Never treat silence as a yes.
- Lists return one page: pass `nextCursor` as `cursor` for the next.
- To retry a write after an error or timeout, send the same idempotency key both times so it happens once. Without one, a retried write can happen twice.
- Errors carry `code` and `message`: `unauthorized` → sign in again; `forbidden` → this sign-in lacks the access or role (tell the person what is missing, never work around it); `rate_limited` → wait for `Retry-After`; `invalid_request` → fix the input from `details`.

## For people only

Never do these yourself, even if you could. Ask for approval (below) or tell the person to do it in the app, and why:

- Add credits to a balance: Adding credits gives value away; an agent asks a person (limit.credit approval)
- Redeem an approved request: Returns a secret. Secrets never enter an agent's conversation. The CLI writes them to a file instead.
- Add a webhook endpoint: Changes where workspace data is sent. A person configures webhook endpoints (Settings → Developers in the app).
- Remove a webhook endpoint: Changes where workspace data is sent. A person configures webhook endpoints (Settings → Developers in the app).
- Re-enable a webhook endpoint: Changes where workspace data is sent. A person configures webhook endpoints (Settings → Developers in the app).
- Update a webhook endpoint: Changes where workspace data is sent. A person configures webhook endpoints (Settings → Developers in the app).

## When a person must approve

Some actions are never yours to take alone: creating an API key, adding or removing a webhook endpoint, and others Limitry lists. ASK instead, and wait for a person. You file a request with your reason and show the person its approval link; they decide in Limitry. A secret it creates goes to the person, never to you, and you never ask them to paste one. Requests expire after 24 hours; a denied request is final, so ask the person what they want instead. How: tools.md or cli.md.

## More

- Documentation for agents: https://limitry.com/docs/llms.txt (index) and https://limitry.com/docs/llms-full.txt (everything)
- Errors: https://limitry.com/docs/errors/

## What Limitry is

Limitry decides, at request time, whether a **subject** (a user,
workspace, API key or agent of the person's own product) may do an
**action** at a **cost**. The answer is allowed or not allowed, with what
remains — a "not allowed" is a normal answer, not an error to retry.

Two kinds of people ask you for Limitry work:

- **Developers** wiring Limitry into their product: "Adding Limitry to
  a codebase" below. Writing code, `limitry.json` and `limitry sync`
  need a terminal (the CLI); the rest works through the tools too.
- **Product and finance people** asking about their customers' usage:
  "Questions about usage" below. Every step there works through the
  tools (Claude, ChatGPT, another chat app with MCP, or an always-on
  agent); no CLI needed. They name
  customers ("Acme"), not ids, and mean their real customers: live.

## Test and live (environments)

Every workspace has two environments, `test` and `live`, each with its
own API keys, limits, counts, balances, reservations and usage. They
never share anything: a limit id or reservation from one is not found
in the other, and names are unique per environment.

- **API keys are bound:** `lim_test_…` acts in test, `lim_live_…` in
  live, always. Sending `X-Environment` with a key that names the other
  is 403.
- **Your credential (personal token) defaults to TEST.** Live must be
  named: CLI `--environment live`, `LIMITRY_ENVIRONMENT=live` or
  `limitry environment use live`; HTTP `X-Environment: live`; SDK
  `createClient({ token, environment: "live" })`. An unknown name is 400.
- **An MCP connection works where the person chose** when they
  connected you ("It works in, by default": Test or Live, starting on
  Test). `?environment=live` on the connection address overrides that
  choice. To change it, the person reconnects and picks the other.
- **Every answer names its environment:** the CLI prints
  `Environment: <name>` on stderr, every MCP tool result ends with
  `Environment: <name>`, and /v1 responses carry an `X-Environment`
  header. ALWAYS tell the person which environment you acted in, and
  never describe a test result as live.
- **Do the work in test.** Create and try limits in test. Act in live
  only when the person asks for it explicitly: a live limit refuses
  their real customers.
- **Going live:** limits move with Copy to live — a person's step in the
  app (the Limits page), or ask for the `limits.copyToLive` approval
  (input: optional `limitIds`; all test limits when absent). It creates
  missing names in live and updates same-named ones; never credits,
  balances or usage. There is no /v1 operation for it. The production
  app then needs a live key: ask for `workspace.apiKey.create` with
  `"environment": "live"` (an API key request MUST name `test` or
  `live`).
- **Test allowance:** test checks have their own monthly allowance (10%
  of the plan's checks, not billed, no grace). Past it, test checks
  answer 402 `plan_limit_reached`; live is unaffected.
- **A test key cannot change workspace-wide things** (webhook endpoints,
  members): 403. Use a live key or a personal token.
- **Webhooks:** an endpoint belongs to test or live (added in the app
  with that environment selected) and receives only its events;
  `limit.reached` also carries `environment`.
- `limit.credit` (adding credits, an approval) takes `environment`;
  absent, the credits go to test. A negative `amount` takes credits back
  (a refund or chargeback) and needs a `note`; the balance may go below
  zero, and checks are refused until more credits are added.

## Limits

A limit says how much of an action a subject may use per window
(`limits list|get|create|update|archive|restore`, or the `limits_*`
tools; `archive` stops a limit and keeps its history).
Only `subject.kind`, `amount` and `window` are required: `action`
defaults to `*`, `unit` to `units`, and `name` to the sentence the limit
reads as ("100 credits per user per month"; a 409 means that name is
taken: send a `name`). Before creating one, confirm with the person:
which subjects (`subject: { kind }`, plus `id` for one), which action
(or `*`), how many per `minute`, `hour`, `day` or `month` (`rolling:
true` for "in any 24 hours" — no burst at midnight), as a balance
(`total`) or at once (`concurrent`), and whether their product should
allow or refuse when Limitry is unreachable (`failMode`, default `open`
= allow). Every count is stored before a check answers. Creating and
changing limits needs an owner's or admin's token.

What a limit counts is FIXED once created: `subject`, `action`, `unit`,
`window`, `every`, `start`, `rolling`. Changing one is refused (400);
create a new limit and archive the old one. Only `name`, `amount`,
`failMode` and `enabled` change. A limit whose `lastCheckedAt` is null
or over 7 days old is not reached by any check: check that the subject
kind and action match what the code sends.

**Limits in code:** when the person wants their limits kept in their
repository (reviewed in pull requests, applied by CI), write them to a
`limitry.json` at the repository root — `{ "limits": [ … ] }`, each
entry the fields of `limits create` plus a required `name` — and run
`limitry sync --preview`,
then `limitry sync` (`POST /v1/limits/sync`, the `limits_sync` tool,
`limits.sync` in the SDKs). Names are the key: it creates missing
names, changes limits that differ (an omitted field means its default)
and leaves the rest; `--archive-missing` archives active limits the
file doesn't name. The whole file is validated first: one bad entry
changes nothing (`details` names it). A changed subject, action, unit
or window under an existing name is refused: rename the entry instead. Once CI syncs the file, edits
made in the app to those limits are overwritten on the next sync —
say so. GitHub Actions recipe:
https://limitry.com/docs/guides/sync-from-github/

## Usage

To answer "how much is X using?": `usage overview` (`GET /v1/usage/overview`)
gives every active limit at once — its total and top subjects, in the
current window or a `period`; `usage subjects` (`GET /v1/usage/subjects?limitName=`)
lists one limit's subjects — used, left, refused — and `usage query`
(`GET /v1/usage?limitName=`) gives its series by day or hour, with a `total`.
Every operation on one limit (`limits get|update|archive|restore`,
`limits credits list|create`, the `limit.credit` approval) takes the
limit's name wherever it takes its id.
This is Limitry's usage of the person's limits — not the person's own
Limitry plan (`workspace usage`). Reports trail live checks by up to a
minute.

To answer "how much does this user have left?" exactly, now:
`usage standing --subject user:<id> [--shared team:<id>]`
(`GET /v1/usage/standing`, the `usage_standing` tool). It lists every
limit on the subject and its shared subjects with `used`, `remaining`
and `resetsAt`; an empty `limits` list means none applies. It is not a
check: nothing is counted.

Warnings: the webhook event `limit.threshold_crossed` fires when a
subject first reaches 80% and 100% of a limit (a balance: 20% left, then
none; concurrent limits never warn). Limitry never contacts the
customer's users; their own server does, from the webhook.

## Questions about usage

For product and finance people. All but the last only read, so go
ahead without asking, and end each one with what you found and the
choice it leaves the person.

**First, every time: the environment.** Questions about customers mean
**live**. If your results say `Environment: test`, say these are test
figures and stop: in a chat app, the person reconnects Limitry and
picks **Live** when it asks where you work (or connects
`https://mcp.limitry.com/mcp?environment=live`); in a terminal, pass
`--environment live`. Never present test figures as customers' usage.

**Finding the customer.** Limitry knows subjects by the kind and id the
person's product sends (`team:t_8f2`, `user:usr_123`), not by name. Ask
the person for the id, or which limit to look in, then match it in
`usage_subjects`. If you can't tell which subject is "Acme", ask; never
guess.

**One call each.** Answer every question below with ONE call; never
call `limits_list` or `limits_get` first to find a limit or read its
unit. Every usage answer carries each limit's `limitName`, `unit`,
`amount` and `window`. Ranges: a `period` (`today`, `this_week`,
`last_week`, `this_month`, `last_month`; UTC, weeks from Monday), or
`from` and `to`.

- The person names no limit ("on AI", "a limit", "our usage"):
  `usage_overview`. It returns every active limit; pick the one the
  question means from the names, and say which you read.
- The person names one ("the AI budget"): `usage_query` or
  `usage_subjects` with `limitName`. A name matches an active limit,
  exactly or else ignoring case; a 400 names the limits it could mean.

**"How much did we spend on AI last month?"**

`usage_overview` with `period: "last_month"`: read the AI limit's
`total` (`used`, `allowed`, `refused`); never sum points yourself. Named
("the AI budget"): `usage_query` with `limitName` and the period. Give
units as the limit names them (`cents` is money: 12,345 cents is
$123.45).

**"Who spent the most last month?"**

`usage_overview` with `period: "last_month"` (each limit's top
subjects, most used first; `top` up to 20), or `usage_subjects` with
`limitName` and the period. In a past period `remaining` is null.

**"Which customers are close to their limit?"**

1. `usage_overview` with no period: each limit's current window, its
   top subjects with used, remaining and refused, and `resetsAt`.
   (`usage_subjects` with `limitName` for one limit's longer list.)
2. Report the subjects at 80% or more of `amount` (a balance, window
   `total`: 20% or less left), with the limit's name and when it
   resets. Skip `concurrent` limits: they count slots in use now, not
   spend.
3. Stop there. Raising a limit, adding credits or contacting the
   customer is the person's call.

**"How much did Acme use in October?"**

1. Find the subject (above). `usage_query` with `limitName`, `subject`,
   and `from` and `to` (ISO 8601, UTC: October is
   `2026-10-01T00:00:00Z` to `2026-11-01T00:00:00Z`; or `period:
"last_month"`); `granularity: "day"` for day by day. Report `total`.
   If you don't know which limit, `usage_standing` with `subject` lists
   the limits on it.
2. Usage older than the workspace's plan keeps (Free: 7 days) isn't
   returned, and `from` in the answer says where it starts: say so,
   never report it as zero.

**"Who was refused, and why?"**

`usage_overview` with `sort: "refused"` (and a `period` for a past
range) lists, under each limit, only the subjects it refused, the most
refused first. A refusal is the limit working. Say which limit refused
and how often, and leave whether to raise it to the person.

**"Which teams bought credits this month?"**

Credits are added to a balance (a limit with window `total`), not used:
`usage_overview` with `period: "this_month"` gives each balance's
`credits` in the period: `added`, `takenBack` and the `latest`, newest
first, with subject, amount, note and date. For the full history,
`limits_credits_list` with the balance's id or name. `usage_subjects`
shows what each subject has used and has left, not what it bought.

**"A monthly usage summary"**

`usage_overview` with `period: "last_month"`: every limit's total and
its busiest subjects in one answer. Write a short table per limit with
use: total used, checks allowed and refused, the busiest subjects; say
which limits had none. Add `workspace_usage` only if asked about
Limitry's own bill: it is the workspace's Limitry plan, not the
customers' usage.

**"Acme paid for more credits. Add them."**

You never add credits yourself. Find the balance (a limit with window
`total`) and the subject, confirm the amount and the environment (live)
with the person, then file the `limit.credit` approval with `limitId` (its id or name),
`subject`, `amount` and `environment: "live"`, and show the approval
link. An owner or admin approves it in Limitry. Raising a limit's
`amount` instead (`limits_update`) changes it for every subject on that
limit: say so, and do it only on the person's explicit go-ahead.

## Checks

`checks create` (`POST /v1/checks`, the `checks_create` tool) asks
whether a subject may do an action at a cost, and — in the default
`consume` mode — uses it. The answer is ALWAYS 200: read `allowed`; a
`false` is a normal answer to act on (show `limits[].remaining`, wait
`retryAfterSeconds`; `reason.code` is `limit_reached` with
`reason.limitId`, or `too_many_reservations`), never an error to retry. Use `mode: "preview"` to
ask without using. Send an `Idempotency-Key` when the caller may retry.
From the CLI, `limitry check user:usr_123 ai-call [--cost N] [--shared
team:team_42]` is the short form of `limitry checks create` (one-line
answer; `--json` for the decision; exit 0 for a refusal too).

When users share an allowance (a team's pool), pass the team as
`shared: [{ kind: "team", id }]` on each user's check — define the pool
as a limit on the team. Prepaid credits are a limit with window `total`
(a balance); never add credits yourself — ask the person (the
`limit.credit` approval). An agent's budget is a limit in `cents` on
`agent`, action `*`, with each check's `cost` the price (e.g. 500 cents
a day; each step checked as action `tool`).

When the cost is known only afterwards (the tokens of an AI reply in the app), use
`mode: "reserve"` with what it will probably cost, then `reservations
commit` with the actual cost — or `reservations release` if the work
did not happen. Always commit or release (a `finally`): an open
reservation blocks its cost until it expires. For "at most N at once"
use a `concurrent` limit: each reservation takes one slot until
committed or released; for work longer than its `ttlSeconds`, call
`reservations extend` as a heartbeat.

## The usual shapes

Five patterns cover most products. Match the person's job to one, and
reuse its names unless they have their own:

| Job                                                               | Subject                 | Action        | Limit                                                      | Call                                  |
| ----------------------------------------------------------------- | ----------------------- | ------------- | ---------------------------------------------------------- | ------------------------------------- |
| The product's AI features, paid per token, for each of your users | `user`                  | `ai-call`     | 200 `cents` a `month`                                      | `reserve` a `cost`, then the real one |
| Credits a whole team spends                                       | `user`, `shared` `team` | `report`      | `total` balance on `team`; credits on payment; report = 25 | `guard` with `cost`                   |
| Rate limits for the product's own API                             | `apikey`                | `api-request` | 60 per `minute` and 1,000 per `hour`, both `rolling`       | `guard`; refusal → 429 Retry-After    |
| The product's AI agents, spending on models and paid tools        | `agent`                 | `tool`        | 500 `cents` a `day` on action `*`, `failMode: closed`      | `reserve` a `cost`, then the real one |
| Heavy jobs such as exports                                        | `workspace`             | `export`      | 3 `concurrent`                                             | `reserve` (extend for long jobs)      |

## Adding Limitry to a codebase

When the person asks you to add Limitry (or to limit, meter or budget
something), work in this order:

1. **Find what costs money or can be abused** — LLM and other paid API
   calls, image/audio generation, emails and SMS, exports and reports,
   anything a user can trigger in a loop. Note who triggers it (their
   user, API key, agent, team) and what one call costs (1, tokens, cents).
2. **Propose limits in a table** — name, subject kind (and pool:
   `shared` team/org), action, amount, window (per
   minute/hour/day/week/month/year; `every` for a multiple, such as
   `window: "hour", every: 5`; `start`: `calendar` (default),
   `billing_date` for each subject's billing date, or `first_use`;
   `rolling` for "in any 24 hours"; `total` for prepaid credits;
   `concurrent` for "at most N at once"), and fail mode (`open` keeps
   their product working if Limitry is unreachable; `closed` for spend
   that must never run unchecked). **Ask the person** to confirm or
   change it — a wrong limit blocks their real users. Never create
   limits they have not agreed to.
3. **Create the agreed limits in test** (`limits create`, or the
   `limits_create` tool) with an owner's or admin's token. Your token
   acts in test unless told otherwise; confirm the output says
   `Environment: test`.
4. **Wrap the code** with the SDK (`npm install @limitry/sdk`):

   ```ts
   import { createClient, LimitReached } from "@limitry/sdk";
   const limitry = createClient(); // reads LIMITRY_API_KEY

   // Known cost: one request against a per-key rate limit
   await limitry.guard(
     { subject: { kind: "apikey", id: key.id }, action: "api-request" },
     () => handle(request),
   );

   // Cost known afterwards (cents), or at most N at once
   await limitry.reserve(
     { subject: { kind: "user", id: user.id }, action: "ai-call", cost: 5 },
     async (reservation) => {
       const r = await callModel();
       reservation.cost(priceCents(r.usage));
       return r;
     },
   );
   ```

   Put it right before the costly work; pass the REAL subject id and the
   same action name as the limit. Handle `LimitReached` where the
   product answers its user (a 429 with `retryAfterSeconds`, a message,
   an upgrade prompt) — never swallow it. `Unavailable` is separate: a
   limit that fails closed while Limitry is unreachable.

5. **Add a test** that a refusal is handled (mock the SDK or point it at
   a stub returning `allowed: false`), and tell the person which env var
   to set (`LIMITRY_API_KEY`, a workspace API key with `checks:write`):
   a test key (`lim_test_…`) for development, a live key (`lim_live_…`)
   in production.
6. **Going live** is the person's decision: offer the `limits.copyToLive`
   approval and a live key (`workspace.apiKey.create` with
   `"environment": "live"`), and say that until then everything is in
   test.

**"Add a usage page for my users"** (or "show users how much they have
left", "warn users before they run out"), after the steps above:

- **Server route:** for the signed-in user only, call
  `limitry.standing({ subject: { kind: "user", id: user.id }, shared })`
  with the same subjects their checks use, and return the JSON. The API
  key stays on the server; never expose it or call Limitry from a
  browser.
- **Usage bar:** copy the `UsageBar` component from
  https://limitry.com/docs/guides/show-usage/ into their React app and
  render it on the page they choose. Show "credits left" for a balance
  (window `total`, `remaining` may exceed `amount`); render nothing for an
  empty list.
- **Warnings** (if they want emails): a webhook endpoint for
  `limit.threshold_crossed` (ask the person to add it in Settings →
  Webhooks and set the secret), a handler that verifies the signature,
  skips `environment: "test"` in production, sends one email per event
  `id`, and uses their own email provider
  (https://limitry.com/docs/guides/warn-users/).

Past the workspace's plan (10% grace for live; none for the test
allowance), checks answer **402 `plan_limit_reached`** — treat it like unreachable (fail mode applies),
and tell the person to upgrade; never retry in a loop. The SDK honours
each limit's fail mode by itself: it remembers the last
decision's `failMode` and, if Limitry cannot be reached within 1.5 s,
lets the work run (`open`) or throws `Unavailable` (`closed`) — never
`LimitReached`. A 429 or 5xx from Limitry counts as unreachable.

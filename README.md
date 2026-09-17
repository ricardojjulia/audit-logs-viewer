# Audit Logs Viewer

Audit Logs Viewer is an unofficial Dynatrace app for exploring audit events in a Dynatrace environment. It provides separate views for settings audit events, classic API audit events, and API gateway audit events, with built-in filtering, timeframe selection, and per-record detail views.

> **Status:** Personal project by a Dynatrace employee, not an official Dynatrace-supported product. Please use GitHub issues for feedback, but support and updates are not guaranteed.

## Product overview

This app helps Dynatrace users answer questions such as:

- What configuration changes were made in the environment?
- Which API activity happened in a given time window?
- Which user, token, client, app, or source was involved in an audit event?
- What changed before and after a settings update?

The UI is optimized for quick investigation:

- dedicated pages for three audit-log sources
- selectable relative timeframes from 30 minutes to 365 days
- table filtering and multi-select filters
- row selection with a side sheet for full event inspection
- masked token display in tables
- IAM-based user enrichment for full name and email when available

## Supported audit-log sources

The current implementation supports these sources from `dt.system.events`:

| Source | Provider filter | Purpose |
| --- | --- | --- |
| Settings audit logs | `event.provider == "SETTINGS"` | configuration and settings changes |
| Classic audit logs | `event.provider == "CLASSIC_API"` | classic API activity |
| API gateway audit logs | `event.provider == "API_GATEWAY"` | platform/API gateway activity |

All three views also filter on `event.kind == "AUDIT_EVENT"` and sort newest events first.

## Architecture

This repository contains a frontend-only Dynatrace App Toolkit project.

| Area | Details |
| --- | --- |
| Runtime | Dynatrace App Toolkit (`dt-app`) |
| UI stack | React 18 + TypeScript |
| UI components | Dynatrace Strato components and icons |
| Routing | `react-router-dom` with one route per audit source |
| Data access | Dynatrace Query SDK polling async DQL execution |
| User enrichment | Dynatrace IAM client fetches user names/emails for `user.id` UUIDs |
| Detail inspection | JSON viewers and diff rendering for before/after settings payloads |

Relevant files:

- `ui/app/App.tsx` - route registration
- `ui/app/pages/SettingsLogs.tsx`
- `ui/app/pages/ClassicAuditLogs.tsx`
- `ui/app/pages/GatewayLogs.tsx`
- `ui/app/hooks/useSettingsAuditLogs.ts`
- `ui/app/hooks/useClassicAuditLogs.ts`
- `ui/app/hooks/useGatewayAuditLogs.ts`
- `ui/app/hooks/useUserMap.ts`

## Supported workflows

### 1. Review settings changes

Open **Settings Audit Logs** to inspect schema IDs, scope information, object summaries, and before/after value changes for settings events.

### 2. Investigate classic API activity

Open **Classic Audit Logs** to review classic API events by event type, resource, user organization, and authentication metadata.

### 3. Investigate API gateway activity

Open **API Gateway Audit Logs** to review app IDs, client IDs, grant types, origin metadata, and user context for platform/API gateway events.

### 4. Drill into individual records

Select rows and open the detail sheet to inspect:

- audit metadata
- authentication metadata
- settings scope metadata when applicable
- JSON patches
- side-by-side previous/new values for changed settings content

## Prerequisites

- Node.js `>=16.13.0`
- npm
- Dynatrace App Toolkit (`dt-app`)
- Access to a Dynatrace environment where you can deploy and run custom apps

Install the toolkit if needed:

```bash
npx dt-app@latest
```

## Installation

```bash
git clone https://github.com/ricardojjulia/audit-logs-viewer.git
cd audit-logs-viewer
npm install
```

## Configuration

Update `app.config.json` before running or deploying:

1. Set `environmentUrl` to your Dynatrace Apps environment URL.
2. Review the application metadata and requested scopes.

Current configured app scopes:

- `storage:logs:read`
- `storage:buckets:read`
- `storage:system:read`
- `environment-api:audit-logs:read`
- `iam:users:read`

These scopes are used to read audit events from platform storage and enrich records with IAM user information.

## Usage

### Start local development

```bash
npm run start
```

This runs `dt-app dev`.

### Build the app

```bash
npm run build
```

### Deploy to your Dynatrace environment

```bash
npm run deploy
```

### Remove the deployed app

```bash
npm run uninstall
```

## How-to examples

### View the last 24 hours of settings changes

1. Start the app.
2. Open **Settings Audit Logs**.
3. Keep the default 24-hour timeframe or choose another preset.
4. Filter by schema ID, scope type, user organization, or event type.
5. Select one or more rows and open the detail sheet for JSON patch and diff inspection.

### Find API gateway events for a specific app

1. Open **API Gateway Audit Logs**.
2. Use the timeframe selector.
3. Filter by **App Id** or event type.
4. Review the selected row details for client ID, grant type, origin session, and user context.

### Inspect classic API activity for a resource

1. Open **Classic Audit Logs**.
2. Filter by **Resource** or **User Organization**.
3. Use the table action menu to copy values from relevant cells.

## Development, test, and build commands

| Command | Purpose |
| --- | --- |
| `npm run start` | run local development mode with `dt-app dev` |
| `npm run build` | build the app for production |
| `npm run deploy` | build and deploy to the configured environment |
| `npm run uninstall` | uninstall the app from the configured environment |
| `npm run lint` | run ESLint |
| `npm run test` | run Jest tests for the query hooks |
| `npm run update` | update Dynatrace packages with toolkit migrations |
| `npm run info` | print App Toolkit environment info |
| `npm run help` | show toolkit help |
| `npm run generate:function` | scaffold a function |
| `npm run generate:action` | scaffold an action |

## Security and privacy considerations

- Audit logs can contain sensitive operational and identity metadata.
- The UI masks long authentication tokens in table cells, but detailed record views may still expose sensitive fields present in the source event.
- The app requests `iam:users:read` to enrich user UUIDs with names and email addresses.
- Do not point this app at environments unless the operators are authorized to view audit and IAM data.
- Do not commit real tenant URLs, tokens, customer data, or exported audit events to the repository.
- Review `CHECK_IN_POLICY.md` before contributing changes.

## Troubleshooting

### No data is shown

- Confirm the app is deployed to the intended environment.
- Verify the configured `environmentUrl`.
- Check that the signed-in user and app installation have the required scopes.
- Confirm that the selected timeframe actually contains audit events for the chosen source.

### User names or emails are missing

- `user.id` enrichment only works when the value is a UUID and the IAM lookup succeeds.
- If `iam:users:read` is unavailable, the app can still show the raw `user.id` where present.

### Build or start fails

- Verify the Node.js version satisfies `>=16.13.0`.
- Run `npm install` again if dependencies are missing.
- Use `npm run info` to inspect the local App Toolkit setup.

## Contributing

Contributions are welcome, but please keep changes focused and reviewable.

1. Open an issue describing the problem or enhancement.
2. Make minimal, well-scoped changes.
3. Run the existing checks before submitting a pull request:

```bash
npm run lint
npm run test
npm run build
```

4. Avoid committing secrets, tenant-specific configuration, or captured customer data.

## Changelog, releases, and update guidance

This repository does not currently include a formal `CHANGELOG.md` or published Git tags.

Recommended maintenance practice:

- document user-visible changes in pull requests
- tag releases when publishing meaningful updates
- add a changelog once release cadence becomes regular
- run `npm run update` before dependency refreshes to apply supported Dynatrace toolkit migrations

## Licensing status

This repository now includes an ISC license file that matches the existing `license: "ISC"` declaration in `package.json`.

See `LICENSE`.

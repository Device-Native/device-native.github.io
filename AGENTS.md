This repository is the DeviceNative Android client SDK.
It syncs with `dn-api` for config, ads, and metadata; sends stats back to the backend; and handles on-device ad selection, caching, impression registration, and click routing.

## Runtime and deployment notes

- Android library module built with Gradle Kotlin DSL.
- Main SDK code lives under `app/src/main/java/com/devicenative/dna`.
- `DNANetworkRequestSync` and `DNANetworkRequestStats` talk to the backend sync/stats APIs, while click/link handling bridges into `dn-click-router` behavior and partner URLs.
- Multiple agents may run at once. Do not revert unrelated local changes.

## Repo role in the shared analytics stack

- Client sync requests pull remote config, ad bundles, metadata, and timing parameters from `dn-api`.
- Stats, impression, and click code in this repo produces the raw client-side events that later land in Athena and downstream analytics rollups.
- Debugging SDK issues often requires comparing device behavior with `dn-api` logs, click-router outcomes, and the finalized analytics numbers shown in the dashboard.

## Connected repo directory

These repositories are part of the same DeviceNative sync/click/analytics loop and often need to be debugged together:

- [`dn-sdk`](../dn-sdk): Android client SDK. Syncs with `dn-api`, renders/caches on-device ads and content, and emits stats, impression, and click activity upstream.
- [`dn-api`](../dn-api): Core Spring API. Serves sync/config payloads, ingests `/v2/stats`, and handles partner postbacks.
- [`dn-click-router`](../dn-click-router): Spring click-routing service. Resolves redirect chains, caches stable redirect outcomes, and logs click behavior.
- [`dn-analytics-processor`](../dn-analytics-processor): Scheduled analytics processor. Builds rollups from Athena data, updates analytics tables, and finalizes daily JSON in S3.
- [`dn-frontend`](../dn-frontend): Next.js/Amplify dashboard and admin UI. Displays analytics and manages content, offers, settings, and operational tooling.
- [`dn-jsads`](../dn-jsads): JavaScript targeting-algorithm repo. Houses the targeting code-version logic plus test data and runnable framing used to validate selection behavior locally.

## Google Workspace access

For Google Docs, Sheets, Slides, Drive, Gmail, or Calendar work in this repo, use the DeviceNative-specific wrapper so you do not overwrite the other `gws` login on this machine.

- Preferred command: `/bin/zsh ./scripts/gws-devicenative`
- Shared config dir: `/Users/alex/.config/gws-devicenative`
- DeviceNative OAuth is already configured for `alex@devicenative.com` on this machine for Drive, Docs, and Sheets access
- Normal Doc/Sheet work should not require another login unless `auth status` shows the token is invalid
- Setup and guardrails: `docs/agents/google-workspace.md`

## AWS access (for agent debugging)

Goal: let agents audit production/staging state (logs + read-only data) while debugging SDK/backend integration issues against live services.

Assumption: an AWS CLI profile exists (default recommended: `devicenative-codex-agent`) with least-privilege access to:
- CloudWatch Logs for the backend services this SDK talks to
- DynamoDB read access for analytics/config tables
- S3 read access for analytics JSON + Athena output
- Athena query execution + Glue metadata read

### Suggested IAM resource scope

CloudWatch Logs:
- `logs:DescribeLogGroups`, `logs:DescribeLogStreams`
- `logs:GetLogEvents`, `logs:FilterLogEvents`
- `logs:StartQuery`, `logs:GetQueryResults`, `logs:StopQuery`
- Scope:
  - `/ecs/*DNA*`
  - `/ecs/*Analytics*`
  - `/ecs/*DeviceNative*`

DynamoDB (read/debug):
- `dynamodb:ListTables`
- `dynamodb:DescribeTable`
- `dynamodb:GetItem`
- `dynamodb:BatchGetItem`
- `dynamodb:Query`
- `dynamodb:Scan` (use sparingly)
- `dynamodb:PartiQLSelect`
- Scope:
  - `DeviceNativeParquetProcessingState`
  - `DailyAnalyticsFinalizationState`
  - `DeviceKey-7ytrvc7eybecretmsaot5aupfe-prod`
  - `SessionReporting-7ytrvc7eybecretmsaot5aupfe-prod`
  - `UsageReporting-7ytrvc7eybecretmsaot5aupfe-prod`
  - `AdReporting-7ytrvc7eybecretmsaot5aupfe-prod`
  - `AggAdReporting-7ytrvc7eybecretmsaot5aupfe-prod`
  - `Advertisement-7ytrvc7eybecretmsaot5aupfe-prod`

Athena + Glue:
- Athena:
  - `athena:StartQueryExecution`
  - `athena:GetQueryExecution`
  - `athena:GetQueryResults`
  - `athena:ListWorkGroups`
  - `athena:GetWorkGroup`
- Glue:
  - `glue:GetDatabase`
  - `glue:GetDatabases`
  - `glue:GetTable`
  - `glue:GetTables`
  - `glue:GetPartition`
  - `glue:GetPartitions`
- Repo defaults:
  - Workgroup: `primary`
  - Database: `devicenative`
  - Athena output: `s3://dna-stats-log/AthenaOutput/`
  - Main tables: `dna_stats_partitioned`, `dna_postbacks_partitioned`

S3:
- Read/list on:
  - `dna-aggregate-stats` (daily JSON files)
  - `dna-stats-log` (Athena output)
- Write required for Athena output only:
  - `s3://dna-stats-log/AthenaOutput/*`

## Before using AWS CLI (per terminal / per agent run)

- `export AWS_PROFILE=devicenative-codex-agent`
- `export AWS_REGION=us-east-1`
- `export AWS_PAGER=""`
- If SSO profile: `aws sso login --profile "$AWS_PROFILE"`
- Verify identity: `aws sts get-caller-identity`

## Quick command cookbook

CloudWatch:
- `aws logs describe-log-groups --log-group-name-prefix "/ecs/" --max-items 100`
- `aws logs tail "/ecs/<log-group>" --since 1h --follow`

Athena:
- `aws athena start-query-execution --work-group primary --query-execution-context Database=devicenative --result-configuration OutputLocation=s3://dna-stats-log/AthenaOutput/ --query-string "SELECT 1"`
- `aws athena get-query-execution --query-execution-id <id>`
- `aws athena get-query-results --query-execution-id <id>`

DynamoDB:
- `aws dynamodb describe-table --table-name SessionReporting-7ytrvc7eybecretmsaot5aupfe-prod`
- `aws dynamodb query --table-name SessionReporting-7ytrvc7eybecretmsaot5aupfe-prod --index-name byDeviceKeyByDate --key-condition-expression "deviceKey = :dk AND #td = :d" --expression-attribute-names '{"#td":"timeDay"}' --expression-attribute-values '{":dk":{"S":"<deviceKey>"},":d":{"S":"<YYYY-MM-DD>"}}'`

S3:
- `aws s3api list-objects-v2 --bucket dna-aggregate-stats --prefix daily-analytics/<deviceKey>/ --max-items 20`
- `aws s3 cp s3://dna-aggregate-stats/daily-analytics/<deviceKey>/<YYYY-MM-DD>.json -`

## Guardrails

- Default to read-only for debugging.
- Do not run write/update/delete AWS actions unless explicitly requested.
- Keep log windows and query ranges narrow (cost + speed).
- Avoid broad DynamoDB scans on hot/large tables.

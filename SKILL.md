---
name: mh2-project-data-skill
description: Use this skill when the user wants to query or generate ThinkingData/数数 SQL and model analysis requests for the MH2 project, using the 2026-08-21 tracking workbook and service-side ID mapping catalog. Supports event analysis, retention analysis, funnel analysis, distribution analysis, path analysis, interval analysis, attribute analysis, and ID/name mapping lookup.
---

# MH2 TA 数据查询 / 数数模型分析 Skill

## Core Workflow

Use this skill to turn natural-language MH2 analysis questions into executable ThinkingData SQL, model-analysis JSON request bodies, or task Markdown files.

1. Identify the requested analysis type: SQL, event analysis, retention, funnel, distribution, path, interval, attribute, or event-user-list drilldown.
2. Read `resources/project/verified-reporting-contracts.md` for verified MH2 defaults and response rules, then resolve the remaining metric口径 from the current user request and bundled resources.
3. When the user asks for MH2 "首日分析", Stage1, 小秘境推进, or 小秘境驻留, read `resources/project/stage1-retention-report.md` and the "Stage1 小秘境关卡推进指标" section of `resources/project/verified-reporting-contracts.md` before selecting requests. A user-only "首日分析" must combine retention/final-progress results with the required per-`dungeon_id` small-dungeon progression chain; final-progress distribution alone is incomplete. Use a compatible configured ThinkingData connector for read-only requests, or generate the complete SQL/model-request plan when no connector is available. Do not claim a generated plan is a live report, and do not assume any private runner or Tool exists.
4. Look up exact event names, property names, and enum meanings in `resources/project/mh2-events.md`.
5. Search `resources/project/mh2-event-index.csv` and `resources/project/mh2-event-properties.csv` when the event or property name needs exact spelling.
6. Search `resources/project/mh2-mapping-tables.md` and `resources/project/mapping_exports/` when an event property contains server-side IDs such as dungeon, item, career, skill, equipment, pet, reason, or sub_reason.
7. Generate SQL or model-analysis JSON using the matching template under `resources/templates/`.
8. If a live ThinkingData API call is unavailable, return the request body or SQL and explain what endpoint/config is needed to run it.

## Canonical Resources

- `resources/project/mh2-events.md`: event tracking workbook converted from `代号：MH2_埋点方案_20260821.xlsx`; use it as the event/property dictionary for MH2.
- `resources/project/mh2-event-index.csv`: compact event index with event name, display name, tag, description, and property count.
- `resources/project/mh2-event-properties.csv`: flat event-property index for exact property lookup.
- `resources/project/mh2-mapping-tables.md`: static server-side mapping catalog plus registered, separate small-dungeon binding/unlock mapping resources; it identifies each mapping domain and its join key.
- `resources/project/verified-reporting-contracts.md`: verified MH2 reporting defaults, live-query result rules, and Stage1 small-dungeon progression metrics. Read this before selecting a default identity, metric, or user-facing response format.
- `resources/project/stage1-retention-report.md`: portable Stage1 analytical contract. It defines the required inputs, calculations, and evidence rules, but does not include a private production runner or OpenClaw Tool.
- `resources/project/tracking-dictionary-reproduction.md`: scope and limitations of reproducing the core event/field set from the raw tracking workbook.
- `scripts/build_tracking_dictionary.py`: regenerate a readable event/field dictionary from `resources/project/raw/代号：MH2_埋点方案_20260821.xlsx`; it does not generate mapping tables or templates.
- `resources/project/mapping_exports/`: CSV exports of each mapping workbook sheet for offline lookup.
- `resources/project/raw/`: raw source workbooks preserved for deeper inspection when needed.
- `resources/templates/`: ThinkingData SQL and model-analysis task templates copied from the existing project skill.
- `resources/model-api/thinkingdata-model-api.xlsx`: ThinkingData model API parameter reference copied from the existing project skill.
- `resources/config/ta.example.json`: public example config.
- `resources/config/ta.local.example.json`: local private config template. Do not commit real tokens.

## Project Defaults

- Default project: MH2, `projectId = 41`.
- Default event table: `ta.v_event_41`.
- Default user table: `ta.v_user_41`.
- Default partition field: `"$part_date"`.
- Default event-name field: `"$part_event"`.
- Default event-time field: `"#event_time"`.
- The default reporting and retention identity is `#account_id`; see `verified-reporting-contracts.md`. Use a different identity only when the user explicitly requests a different grain or supplies a conflicting authoritative口径.
- Every query against `ta.v_event_41` must include a `"$part_date"` partition filter in each CTE/subquery that reads the event table.

## Default Metric Definitions

- **新增 / 新增活跃 (default)**: count distinct `"#account_id"` for `"$part_event" = 'role_create_success'` on the requested date. A successful role creation is the default MH2 new-user event and also qualifies as same-day activity.
- **纯新用户 / 勇1新用户**: this is not a synonym for the default 新增. It means an MH2 creator who is **not** in the imported Hero1 old-user cluster `cohort_20260824_202104`, whose native label is `不属于 勇1老用户`. Read `resources/project/verified-reporting-contracts.md` before selecting the native grouping or describing this population.
- **勇1老用户 source of truth**: use the project-41 imported cluster only: `user_result_cluster_41`, `cluster_name = 'cohort_20260824_202104'`, with `#varchar_id` matched to `ta.v_user_41.r2uid`. Do not rebuild this population from project-4 phone or ID-card matching in a report query.
- **勇1标签时间 is mandatory**: this imported cluster is manually followed up every day. When a user explicitly requests data for `勇1老用户`, `纯新用户`, or `勇1新用户`, label the output with the cluster tag/snapshot date actually used, for example `勇1老用户标签时间：2026-08-24`. The event query period is not a substitute for this label time.
- When reporting a metric, table, or chart as `纯新用户` or `勇1新用户`, state the population definition alongside the data: `纯新用户：MH2 当日创角且不属于勇1老用户（cohort_20260824_202104）`. Do not relabel an all-creator result as 纯新用户. If the request combines this population with a dungeon/progression drilldown, preserve the requested population definition and return `EVIDENCE_REQUIRED` rather than claiming a compound query is verified without a successful live result.
- **Do not use** `account_login` with `first_login = 1` for the default 新增 metric. That event can have an empty `"#account_id"` and represents first account login / activation rather than the fixed new-role metric.
- Use `account_login` with `first_login = 1` only when the user explicitly asks for 首次账号登录、账号激活, or activation-chain analysis. State that this is an activation metric, not the default 新增 metric.
- **活跃 (default)**: count distinct `"#account_id"` for `"$part_event" = 'login'` on the requested date.

## Date Placeholders

When generating reusable SQL templates, prefer these placeholders:

```text
${PartDate:today}
${PartDate:yesterday}
${PartDate:last7days}
${PartDate:last30days}
${PartDate:2026-08-01}
${PartDate:2026-08-01..2026-08-07}
```

Use concrete dates only when the user explicitly asks for fixed dates or the execution environment does not support placeholders.

## Supported Actions

Use these action names in task configs:

- `sql`
- `event-analyze`
- `event-analyze-download`
- `event-user-list`
- `event-user-list-download`
- `retention-analyze`
- `funnel-analyze`
- `distribution-analyze`
- `path-analyze`
- `interval-analyze`
- `attribute-analyze`

Map live API calls to these paths when a ThinkingData connector/config is available:

```text
sql                         POST /querySql?token=<TA_USER_TOKEN>
event-analyze               POST /open/event-analyze?token=<TA_USER_TOKEN>
event-analyze-download      POST /open/event-analyze?token=<TA_USER_TOKEN>
event-user-list             POST /open/event-user-list?token=<TA_USER_TOKEN>
event-user-list-download    POST /open/event-user-list?token=<TA_USER_TOKEN>
retention-analyze           POST /open/retention-analyze?token=<TA_USER_TOKEN>
funnel-analyze              POST /open/funnel-analyze?token=<TA_USER_TOKEN>
distribution-analyze        POST /open/distribution-analyze?token=<TA_USER_TOKEN>
path-analyze                POST /open/path-analyze?token=<TA_USER_TOKEN>
interval-analyze            POST /open/interval-analyze?token=<TA_USER_TOKEN>
attribute-analyze           POST /open/attribute-analyze?token=<TA_USER_TOKEN>
```

Live execution is optional. Use it only when the installing user has intentionally supplied a compatible endpoint and token. Never look for a drive-specific project directory, a private Tool, cached production output, or a historical report as a substitute for that configuration.

## Missing Information Rules

Before producing a final query or request, verify:

1. Analysis target and metric口径.
2. Time range and timezone.
3. User identity grain, using `#account_id` unless the request explicitly requires a different grain.
4. Required event names and property names from bundled resources.
5. Filters, cohorts, grouping dimensions, and output format.
6. Whether the user wants only SQL/JSON or an actual API call/export.

If the user asks for a simple draft and the identity grain is unclear, use a clearly marked assumption and keep the query easy to adjust.

## Token Handling

- Never write a real token into generated SQL, JSON, Markdown, README, examples, or committed config.
- Prefer `resources/config/ta.local.json` only when the user has intentionally created it locally.
- Use `resources/config/ta.example.json` or `ta.local.example.json` as structure references.
- If a token is needed and not available, ask the user for it only for the live API call; otherwise return the runnable request body.

## Resource Snapshot

- Events extracted: 34
- Event tags extracted: 13
- Mapping sheets exported: 10

## Output Style

- Give directly usable SQL, JSON request bodies, or task Markdown.
- Mention which bundled resource supplied the口径 when there may be ambiguity.
- Preserve ThinkingData quoted fields exactly, such as `"$part_date"`, `"$part_event"`, and `"#event_time"`.
- Do not invent event/property names. If a field cannot be found, say what was searched and ask for the missing口径.

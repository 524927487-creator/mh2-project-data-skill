---
name: mh2-project-data-skill
description: Use this skill when the user wants to query or generate ThinkingData/数数 SQL and model analysis requests for the MH2 project, using the 2026-08-21 tracking workbook and service-side ID mapping catalog. Supports event analysis, retention analysis, funnel analysis, distribution analysis, path analysis, interval analysis, attribute analysis, and ID/name mapping lookup.
---

# MH2 TA 数据查询 / 数数模型分析 Skill

## Core Workflow

Use this skill to turn natural-language MH2 analysis questions into executable ThinkingData SQL, model-analysis JSON request bodies, or task Markdown files.

## Formal Execution Route

For a new MH2 query, use this default route. Do not choose a local script merely because it matches a word in the request:

```text
MH2 Skill and its verified project resources
-> formal MH2 request definition or approved SQL
-> F:\api数据提取\adapters\thinkingdata_analysis.py
-> ThinkingData native API or /querySql
-> request, raw response, normalized result, and any required local report
```

### Entry Status Taxonomy

- `CANONICAL`: the default route for a new MH2 query. It uses verified MH2 request definitions or approved SQL and the shared ThinkingData adapter.
- `SPECIALIZED_CASE`: a fixed business contract or report path. It remains valid only for its documented case and is not a generic query entrypoint.
- `VERIFIED_EXPERIMENT`: real request evidence exists, but the workflow is not part of MH2 formal routing.
- `DEPRECATED/HISTORICAL`: retained only for audit traceability. Do not use it as a new-task execution entrypoint.

- **CANONICAL / retention:** select only a `READY` grouping from `mh2-segment-registry.md`, then use `F:\Projects\data-analysis-agent\src\mh2_retention_config.py:retention_request` with the shared `retention_analyze` adapter.
- **CANONICAL / event, funnel, distribution, and approved SQL:** reuse the matching verified request definition or approved SQL in the MH2 formal configuration/report code, then call the shared adapter (`event_analyze`, `funnel_analyze`, `distribution_analyze`, or `query_sql`). Do not substitute SQL when a native model request has failed semantically.
- **CANONICAL report path:** `scripts/run_mh2_retention_report.py` is the report path for the existing Stage1 report. It is also the direct existing-template route for the named intent `SMALL_DUNGEON_FULL_ANALYSIS`; it is not a default route for a new metric or an ad hoc query.
- **SPECIALIZED_CASE:** `scripts/mh2_tool.py` is a fixed D1-career contract. Repeat-equipment, small-dungeon quality, old-user export, and career-ranking scripts serve their documented cases only; they are not generic query entrypoints.
- **VERIFIED_EXPERIMENT:** `scripts/retention_tool.py`, `scripts/retention_segment_profile.py`, and `retention-segment-analysis` preserve real candidate-discovery, classification, confirmation, retention, and safe-ranking evidence. They are not active MH2 formal routing and must not be promoted from a successful historical run.
- **DEPRECATED/HISTORICAL:** `F:\Projects\data-analysis-agent\其他项目参考\` and `runtime/hero2_*` are audit evidence. Do not execute their generated SQL as a new-query substitute.

### Shared ThinkingData Filter Contract

For generic ThinkingData filter and response validation, use the existing
`F:\api数据提取\docs\thinkingdata_analysis_data_skill.md`. This MH2 Skill
owns project definitions and registry metadata; it does not duplicate the
generic filter schema. A Tag or cohort filter request uses its `READY`
registry metadata together with that DataSkill's witnessed-object and
reconciliation validation.

If the existing formal route has no verified executor for an action, do not infer that an endpoint template makes it ready. Resolve the missing business definition or return `EVIDENCE_REQUIRED`; do not create a competing wrapper or silently route through a specialized script.

1. Identify the requested analysis type: SQL, event analysis, retention, funnel, distribution, path, interval, attribute, or event-user-list drilldown.
2. Read `resources/project/verified-reporting-contracts.md` for locally verified MH2 defaults and response rules, then resolve the remaining metric口径 from the current user request and bundled resources.
3. When the user asks for MH2 "首日分析", Stage1, 小秘境推进, 小秘境驻留, or `看/看看/看一下 [日期] 小秘境数据`, read `resources/project/stage1-retention-report.md` and the "Stage1 小秘境关卡推进指标" section of `resources/project/verified-reporting-contracts.md` before selecting requests. The latter phrase is the named `SMALL_DUNGEON_FULL_ANALYSIS` intent: execute the existing full small-dungeon template without asking the user to choose metrics. If its date is absent, resolve only the existing time-range requirement; do not turn it into a question about metrics or business focus. A user-only "首日分析" must combine the existing Stage1 retention/final-progress results with the required per-`dungeon_id` small-dungeon progression chain; final-progress distribution alone is incomplete. This is a data-query request, not authorization to change code: use the existing shared Tool or a direct read-only request shape from the contract and derive the required report fields in the current session. These resources define the existing Stage1 chain; do not create a second Stage1 query framework.
4. For small-dungeon player-behavior questions about first success, pre-success challenge, or post-success replay, read the "Player Behavior" section of `resources/project/verified-reporting-contracts.md` with the Dungeon contract before selecting requests.
5. For equipment acquisition, yellow/precious equipment, set, wearability, or source questions, read the "Equipment" section of `resources/project/verified-reporting-contracts.md` with the item/equipment/reason mapping exports.
6. Keep Player Behavior and Equipment as independent analysis domains. Do not imply a required progression -> equipment -> retention workflow; combine them only when the user asks a specific compound question and its live evidence supports that combination.
7. When the request needs a user layer, tag, cohort, filter, or any `groupBy`, read `resources/project/mh2-segment-registry.md` before selecting request metadata. Only its `READY` entries may be used as reusable native groupings or filters. Apply its 标签与分群时效 Gate: record the behavior/cohort date, return window, actual `specifiedClusterDate`, last successful tag update time, and freshness verdict. For a native Tag/cohort filter, apply the shared DataSkill's witnessed-object and filtered-versus-grouped reconciliation validation. For a numeric tag, also select a registered analysis tier scheme; never infer interval boundaries from the tag name or a prior report.
8. Look up exact event names, property names, and enum meanings in `resources/project/mh2-events.md`.
9. Search `resources/project/mh2-event-index.csv` and `resources/project/mh2-event-properties.csv` when the event or property name needs exact spelling.
10. Search `resources/project/mh2-mapping-tables.md` and `resources/project/mapping_exports/` when an event property contains server-side IDs such as dungeon, item, career, skill, equipment, pet, reason, or sub_reason.
11. Generate SQL or model-analysis JSON using the matching template under `resources/templates/`.
12. If a live ThinkingData API call is unavailable, return the request body or SQL and explain what endpoint/config is needed to run it.

## Canonical Resources

- `resources/project/mh2-events.md`: event tracking workbook converted from `代号：MH2_埋点方案_20260821.xlsx`; use it as the event/property dictionary for MH2.
- `resources/project/mh2-event-index.csv`: compact event index with event name, display name, tag, description, and property count.
- `resources/project/mh2-event-properties.csv`: flat event-property index for exact property lookup.
- `resources/project/mh2-mapping-tables.md`: static server-side mapping catalog plus registered, separate small-dungeon binding/unlock mapping resources; it identifies each mapping domain and its join key.
- `resources/project/verified-reporting-contracts.md`: locally verified MH2 reporting defaults, live-query result rules, Stage1 progression metrics, and independent Player Behavior / Equipment analysis rules. Read this before selecting a default identity, metric, or user-facing response format.
- `resources/project/mh2-segment-registry.md`: canonical registry of reusable MH2 grouping items, tags, and cohorts. It owns business definitions and approved `groupBy` metadata; only `READY` entries can be used in a live request.
- `F:\Projects\data-analysis-agent\scripts\mh2_segment_stability_gate.py`: read-only local stability gate for saved tag/cohort metadata, behavior grouping, and retention conservation; it returns `READY` only when freshness and conservation checks pass.
- `resources/project/stage1-retention-report.md`: the current deployed Stage1 report and OpenClaw Tool contract. Read it only for that existing report/Tool; it adds report-specific rules without changing the generic identity.
- `resources/project/tracking-dictionary-reproduction.md`: scope and limitations of reproducing the core event/field set from the raw tracking workbook.
- `scripts/build_tracking_dictionary.py`: regenerate a readable event/field dictionary from `resources/project/raw/代号：MH2_埋点方案_20260821.xlsx`; it does not generate mapping tables or templates.
- `resources/project/mapping_exports/`: CSV exports of each mapping workbook sheet for offline lookup.
- `resources/project/raw/`: raw source workbooks preserved for deeper inspection when needed.
- `resources/templates/`: ThinkingData SQL and model-analysis task templates copied from the existing project skill.
- `resources/model-api/thinkingdata-model-api.xlsx`: ThinkingData model API parameter reference copied from the existing project skill.
- `resources/config/ta.example.json`: public example config.
- `resources/config/ta.local.example.json`: local private config template. Do not commit real tokens.

## Query Knowledge Index

Use this index after classifying the request, before asking the user to repeat
an existing MH2 definition. It routes only to existing Skill resources; it is
not a second knowledge base or execution layer.

| Business question | Read first | Definition status and action |
| --- | --- | --- |
| DAU, new roles, daily duration, D1, career/channel, Hero1 old/new | `verified-reporting-contracts.md`, then `mh2-segment-registry.md` when a layer is requested | Reuse the verified metric and population definition. |
| Complete first-day / Stage1 report or its fixed G2.2 activation funnel | `stage1-retention-report.md` | Reuse the fixed report contract only for that report. G2.2 is an installation-day activation funnel, not a Guild or creator-cohort funnel. |
| `看/看看/看一下 [日期] 小秘境数据` | `stage1-retention-report.md` section `Stage1 2.4 小秘境全量关卡分布`, then `verified-reporting-contracts.md` section `Stage1 小秘境关卡推进指标` | `KNOWN`: set `QUERY_INTENT=SMALL_DUNGEON_FULL_ANALYSIS` and use the existing full template, including its required B-line. Do not ask the user to select metrics or return only event counts. A missing date is only a time-range item, not a metric-choice question. |
| Dungeon / small-dungeon success, result states, progression, residency, or first success | `verified-reporting-contracts.md` sections `Dungeon 通用分析边界` and `Stage1 小秘境关卡推进指标` | Reuse the applicable verified submetric; keep entry, settlement, and data-quality facts separate. |
| "小秘境卡关" | `verified-reporting-contracts.md` section `小秘境业务意图选择` | It has several mature business interpretations but no default single metric. Ask the smallest business question needed to distinguish failure/repeated pre-success challenge from post-success lack of progress; do not ask for SQL, joins, denominators, or a technical window. Replay is not a 卡关 option. |
| "小秘境复刷" / "成功后复刷" | `verified-reporting-contracts.md` section `Player Behavior` | `KNOWN`: this is the independent post-success replay definition. Do not mix it with defeat, residency, or post-success failure to enter the next level. |
| Guild / Guild onboarding / Guild funnel | `mh2-events.md`, `mapping_exports/reason_sub_reason.csv`, then this index and verified contracts | `guide_start` and `guide_finish` exist with `guide_id`; static `prop_flow` mappings record Guild creation cost/refund reasons. Neither establishes a Guild mapping nor a formal Guild funnel chain. State that known boundary and ask only which Guild objective/node and business flow should be measured. Do not relabel G2.2 as Guild. |
| Equipment, acquisition, source, sets, or wearability | `verified-reporting-contracts.md` section `Equipment`, then mappings | Reuse acquisition/source rules; do not infer stage attribution from `prop_flow`. |

Do not promote a dated report result, a local runtime result, or an unverified
field-name association into this index. Historical evidence can justify a
definition only after its stable business meaning and live-query contract have
been established in the formal resources.

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
- **在线时长 (default)**: role-level daily cumulative online duration. `角色当日累计在线时长` means the sum of valid online-duration observations for the same `#account_id` within the requested natural day. Do not treat one `logout.online_time` event value as the role's final daily duration. Use a single-session metric only when the user explicitly asks for `单次登录在线时长`、`单次 Session 在线时长` or `每次登录时长`.
- A future daily-duration distribution must first use an existing formal request or approved SQL and retain its live request/raw evidence. The definition above does not itself prove a particular aggregation request has been live-verified.

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

### 新分组稳定性验收

For a newly discovered user tag or cluster, do not promote it from the registry to `READY` based on a single successful response. Save the metadata and the baseline/grouped behavior and retention responses, then run:

```powershell
python scripts/mh2_segment_stability_gate.py --input <evidence-contract.json> --output <run-dir>\stability_summary.json
```

The input must include `freshness` (`behavior_date`, `specified_cluster_date`, `tag_last_updated_at`, `freshness_verdict`), `behavior` (`status`, `baseline_users`, mutually exclusive `groups`), and `retention` (`status`, canonical `series_index=0` rows). The gate rejects stale snapshots, missing updates, non-conserving buckets, and non-exclusive tags used as if they were exhaustive. Only a `READY` result may be used to upgrade the registry entry; otherwise retain `EVIDENCE_REQUIRED` and show the exact missing boundary.

## Missing Information Rules

Before producing a final query or request, verify:

1. Analysis target and metric口径.
2. Time range and timezone.
3. User identity grain, using `#account_id` unless the request explicitly requires a different grain.
4. Required event names and property names from bundled resources.
5. Filters, cohorts, grouping dimensions, and output format.
6. For each tag/cohort: `specifiedClusterDate`, last successful update time, and whether its snapshot is current for the requested behavior date.
7. Whether the user wants only SQL/JSON or an actual API call/export.

If the user asks for a simple draft and the identity grain is unclear, use a clearly marked assumption and keep the query easy to adjust. This draft-only exception never authorizes a live query or a formal business conclusion.

## Query Definition and Validation

Before a live MH2 query, first resolve the request from this Skill's verified
facts. `projectId = 41`, `#account_id` as the generic identity, the documented
event/field meanings, approved mappings, frozen retention definitions, and
Dungeon rules are already confirmed definitions. Reuse them without asking the
user again.

### Definition Readiness Gate

Classify a new request after reading this Skill, its project resources, the
existing DataSkill, and the MH2 verified Baseline. Check only definitions that
can change the result: statistical subject, date/timezone, metric/event,
people versus occurrences, numerator/denominator for a rate, first/last/all
behavior, state meaning, and any population/Tag/cohort restriction.

| State | Decision |
| --- | --- |
| `KNOWN` | A formal MH2 business definition already resolves the relevant items. Reuse it and continue to the existing formal query route without asking again. |
| `DERIVABLE` | The user omitted a detail, but an applicable verified MH2 rule determines it unambiguously. State the adopted rule and continue; do not ask a redundant question. |
| `UNKNOWN_BUSINESS_DEFINITION` | More than one plausible business interpretation would change the result. Return `ASK_USER` / `EVIDENCE_REQUIRED`, ask only the blocking Chinese question(s), and do not call ThinkingData until answered. |
| `UNKNOWN_DATA_CAPABILITY` | The business definition is complete, but the Skill/resources do not establish a required event, field, mapping, or formal executor. Report that capability boundary separately; never disguise it as a business-definition question or substitute a similarly named field. |

Do not infer business semantics from a field name. A user answer remains
conversation-local until a real request and validation establish it as a
reusable MH2 definition; only then may it be added to the project Skill.

Ask in Chinese only when this Skill cannot resolve a definition and the
alternative interpretations would change the number. Ask every genuinely
blocking question together, then retain the user's answers in the current
conversation and continue the original request. Do not call a ThinkingData
Tool while that key ambiguity remains unresolved. For example, an unknown
named item/set must be resolved to the applicable object mapping and behavior
(such as obtained, owned, or equipped) before it can be counted.

For every live result, apply the generic Data Skill validation gate together
with this Skill's project-specific rules. At minimum, preserve the user
question, adopted MH2 definition, selected Tool/request, raw response,
normalized result, and validation outcome in the existing conversation/Tool
evidence. When a tag/cohort is used, additionally preserve `behavior_date` /
`cohort_date`, `return_window` when applicable, actual
`specifiedClusterDate`, `tag_last_updated_at`, and the freshness verdict from
`mh2-segment-registry.md`. A nonzero `return_code`, an immature lifecycle day, an unresolved
mapping, `NO_SOURCE_RELATION`, or a failed applicable assertion is not a
formal business conclusion. State the corresponding Chinese boundary instead
of treating it as zero, guessing a mapping, or falling back to another Tool.
Keep existing MH2 anomaly boundaries intact: `scene_num=0` remains
`DATA_ABNORMAL`, must be reported separately, cannot support scene-level
inference, and does not block an otherwise valid aggregate Dungeon result.

## Gate 3 Reversible Query-Experience Policy

<!-- GATE3_REVERSIBLE_POLICY:START -->

This section is a query-experience policy only. It controls whether to ask an
operator a business clarification; it does not define MH2 facts, event
semantics, mappings, SQL, adapters, Tools, filters, `groupBy`, Tags, cohorts,
or validation. If this policy regresses an established query, revert only this
marked block. Do not revert any other Skill resource, DataSkill contract, or
MH2 implementation.

Classify a request as `DIRECT` when its business meaning is `KNOWN` or
`DERIVABLE`. Continue into the existing formal query route without an operator
question. Ask in Chinese (`ASK_USER`) only when the business meaning itself
has two or more reasonable interpretations and choosing among them would
materially change what the operator sees.

The following are never Gate 3 clarification reasons: an immature return day;
a missing or stale Tag/cohort snapshot; a field, mapping, Tool, SQL, or
executor choice; request construction; validation; data availability; or the
choice between SQL/JSON, a live query, and an export. Resolve these in the
existing downstream route and report the resulting evidence or data boundary
there. In particular, a clear request such as `看 8 月新玩家留存` proceeds with
the formal new-player and retention defaults; unavailable return days are
shown as not mature rather than turned into an operator question.

Use an existing formal definition when one exists, and obey a definition the
operator supplies in the current request. An `UNKNOWN_DATA_CAPABILITY` is a
downstream evidence boundary, not `ASK_USER`. A valid `ASK_USER` question must
use only business language and name the competing business outcomes, never
SQL, fields, joins, Tags, cohorts, Tools, or other implementation details.

<!-- GATE3_REVERSIBLE_POLICY:END -->

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

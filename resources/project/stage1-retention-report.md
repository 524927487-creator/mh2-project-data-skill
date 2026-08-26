# MH2 Stage1 Retention Report Contract

This resource applies only to the existing Stage1 report at `F:\Projects\data-analysis-agent\scripts\run_mh2_retention_report.py` and the OpenClaw `analysis_mh2_retention` Tool. It is not a replacement for generic MH2 defaults.

## Current Deployed Contract

- Project: `41`.
- Formal implementation identity: `#account_id`.
- Cohort event: `role_create_success`.
- D1 return event: `login`.
- Verified `groupBy` rules: read `verified-reporting-contracts.md`; this report does not maintain separate grouping definitions.

The generic Skill default and the current Stage1 implementation both use `#account_id`. The business subject is `角色`.

## D0 And Realtime Rules

- Small dungeon: `dungeon`, `dungeon_result=1`, `dungeon_type=1`, user-day maximum final progress. Chapter D1 uses dynamic tag `tag_20260824_1` after the next-day 01:00 snapshot.
- Large dungeon: use the accepted Stage1 `dungeon_type=2` distribution result and render its numeric progress in 10-level buckets.
- Realtime D1 points: 08:00, 09:00, 10:00, 12:00, 14:00, 16:00, 18:00, 20:00, 22:00, and 24:00. Compare only the preceding cohort at the same time point.

## Stage1 2.4 小秘境全量关卡分布

This is the required per-level progression table in the Stage1 Markdown. It is separate from the final-success chapter and Top10 distributions.

- Fixed report entry: `F:\Projects\data-analysis-agent\scripts\run_mh2_retention_report.py --stage1-cohort <cohort_date>`.
- Fixed source contract: `F:\Projects\data-analysis-agent\contracts\mh2_retention_report\small_dungeon_full_distribution.sql`. Only replace `${cohort_date}`; do not create an ad hoc SQL/API request or a parallel query implementation.
- Scope: the same D0 `role_create_success` cohort, `#account_id`, and `dungeon_type=1` throughout. The source must return the cohort-wide `small_dungeon_participant_count` with every level row.
- Report location: `### 2.4 小秘境全量关卡分布`, after `2.3` and before `## 3. 小秘境 Top10 下钻`.
- Required source/derived fields: formal level name, unlock-content summary, entry roles, challenge count, successful roles, success count, next actual output level entry roles, residency roles, residency share of small-dungeon participants, success rate, success-to-next-entry rate, defeat count, normal-exit count, abnormal-exit count. The Stage1 Markdown presentation is `关卡中文名`, `关联解锁内容`, `累计到达比例`, and `本关驻留率`; do not omit the unlock-content column.
- Order and adjacency: use the formal `dungeon_type=1` `层级/难度` order in `mapping_exports/dungeon_id.csv`; never infer the next level with `dungeon_id + 1`.
- Residency roles: current level entry roles minus next actual output level entry roles. The denominator for its share is the cohort-wide `small_dungeon_participant_count`, not a sum of per-level users. For the final actual output level, use its own entry roles and cumulative share as the endpoint residency so the displayed per-level residency shares sum to 100%.
- The final actual output level leaves next entry and success-to-next-entry rate blank; do not substitute zero.
- Displayed level names first match `mapping_exports/small_dungeon_unlock.csv` by `dungeon_type + dungeon_id` and use its `关卡` value. Only an unmatched level falls back to the formal name in `mapping_exports/dungeon_id.csv`; do not infer from an ID sequence. Unlock-content summary preserves one-to-many entries, and a missing in-scope relation is `NO_SOURCE_RELATION`.

## Execution Boundary

Use the existing shared ThinkingData Tool. Native retention and distribution requests do not fall back to `querySql`; the realtime D1 SQL is the existing approved exception. A private test send requires caller authorization.

## Stage1 B-Line Production Chain

- Status: `READY + REQUIRED`. B-line is not an optional capability for a complete Stage1 run.
- Every `--stage1-cohort <cohort_date>` run must execute the existing small-dungeon B-line after the full per-level distribution source returns and before the Markdown is generated. Do not add a Tool, a second runner, or a parallel analysis architecture.
- Fixed sequence: `Stage1 -> 小秘境全关卡分布 -> 选出重点/TopN关卡 -> 现有B线 -> 保存下钻证据 -> 生成报告`.
- The B-line execution must use the same creator cohort/date and retain its complete detailed result under `<run_dir>/small_dungeon_b_line/`, including the per-dungeon and per-scene settlement evidence. The user-facing Markdown may summarize only the selected/TopN key levels.
- A B-line status of `PASS` or `PASS_WITH_DATA_ANOMALIES` is usable. Any other B-line status blocks the complete Stage1 run; do not emit a partial report that merely says B-line is available.

## Production Contract

When the user asks for a complete MH2 Stage1 / first-day report, run the single fixed entry rather than assembling a second report chain:

```text
F:\Projects\data-analysis-agent\scripts\run_mh2_retention_report.py --stage1-cohort <cohort_date> --as-of <Shanghai local time>
```

- The report is for that one `role_create_success` creator cohort only. It must include the fixed Stage1 sections: creation, channel x Hero1 old/new creation split, creator-cohort online overview, D1 overall/profession/old-new/channel, G2.2 activation funnel, G2.3 same-day creator gameplay, small-dungeon chapter residency, and the full per-level small-dungeon distribution.
- The complete Stage1 report also requires the B-line production chain above: execute it for every run and preserve the full B-line evidence. The user-facing Markdown may render only a selected/TopN small-dungeon drilldown summary after the all-level table; it must not be treated as the complete B-line evidence.
- Channel x Hero1 old/new creation uses `role_create_success`, `#account_id`, `groupBy=[channel, cohort_20260824_202104]`, and renders TapTap/r2_cn x 新用户/勇1老用户. Each share uses that channel's fixed creation count as its denominator, not all creations or a reconstructed group sum.
- Creator-cohort online overview uses the same-day creator filter. Render `login.PER_CAPITA_TIMES` as per-capita login times and `logout.online_time.PER_CAPITA_NUM` as per-capita online seconds converted to minutes. Their denominators are respectively login roles and logout roles; do not silently replace unavailable values with zero.
- Keep G2.2 explicitly marked as the non-creator `#user_id` installation-day funnel. Keep G2.3 and all retention/small-dungeon sections explicitly marked as the creator cohort; do not combine their denominators.
- A successful run writes `run_summary.json`, request parameters, raw responses, normalized results, derived report data, and the Markdown report under one new `runtime\mh2_retention_report\<run_id>\` evidence directory. The report is usable only when every fixed source has `HTTP 200` and `return_code=0`, and the B-line status is `PASS` or `PASS_WITH_DATA_ANOMALIES`.
- The native retention API can return `retained_users = "-"` before D1 matures. This is a successful but unavailable D1 observation: retain the cohort denominator, render returned users and rate as `-`, and label it `D1 尚未成熟/暂无观察值`. Never coerce it to `0` or `0%`, and do not block Stage1 report generation for this case.
- The approved realtime D1 SQL can return exactly the fixed headers `cohort_date`, `return_date`, `hour_point`, `cohort_users`, `retained_users` with no data rows. With `HTTP 200` and `return_code=0`, this is a successful empty observation: normalize it to an empty point set and render `暂无实时观察值`, not `0%`. This exception is only for that exact realtime contract; a nonzero return code, mismatched headers, unexpected response shape, or missing source evidence remains blocking.
- Do not fall back to browser, cached reports, a different SQL query, or a partial report when a required source is blocking. Report the first failing source and preserve its raw evidence instead.

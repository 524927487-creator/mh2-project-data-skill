# MH2 Stage1 Retention Analysis Contract

This is a portable analytical contract for an MH2 Stage1 / first-day report. It defines the required queries, calculations, and evidence rules. This Skill does not bundle a private production runner, a local project directory, an OpenClaw Tool, or ThinkingData credentials.

## Scope And Identity

- Project: `41`.
- Default reporting identity: `#account_id`.
- Cohort event: `role_create_success`.
- D1 return event: `login`.
- Read `verified-reporting-contracts.md` for verified `groupBy` definitions. This document does not introduce competing grouping definitions.

## Required Report Coverage

A complete Stage1 result has all of the following. A final-success distribution, chapter distribution, or Top-N final stages alone is not a complete Stage1 result.

1. Creation and creator-cohort online overview.
2. D1 retention overall and by profession, Hero1 old/new cohort, and channel.
3. G2.2 installation-day funnel and G2.3 same-day creator gameplay, only when their source contracts are available to the installing user.
4. Small-dungeon chapter residency and the full per-`dungeon_id` small-dungeon progression chain.
5. A clearly separated large-dungeon `dungeon_type=2` final-progress distribution rendered in 10-level buckets.

When source contracts for an optional panel are unavailable, return its required request shape and label the report plan incomplete. Do not fabricate the missing panel from historical data.

## Small-Dungeon Progression Chain

For the same D0 `role_create_success` cohort, `#account_id`, and `dungeon_type=1`, prepare or execute these read-only inputs:

- `dungeon_enter`: distinct `#account_id` as `entry_roles` and event count as `challenge_count`, grouped by `dungeon_id`.
- Server `dungeon`: distinct successful `#account_id` as `success_role_count`, successful event count as `success_count`, and each `dungeon_result` event count, grouped by `dungeon_id`.
- `dungeon_key` reconciliation for `enter_only`, `settlement_only`, and duplicated server settlements.

Derive the following for every actual output level:

- `success_rate = success_count / challenge_count`.
- `next_enter_roles`: distinct entrants of the next actual output level, not `dungeon_id + 1`.
- `success_to_next_rate = next_enter_roles / success_role_count`.
- Residency roles: current `entry_roles` minus next actual output level's `entry_roles`.

For the final actual output level, leave `next_enter_roles` and `success_to_next_rate` blank. Its residency is its own entry count and cumulative share so displayed residency shares sum to 100%. Never replace unavailable values with zero.

Map level identity with `dungeon_type:dungeon_id`. For displayed Stage1 names, use `small_dungeon_unlock.csv` `关卡` first and then `dungeon_id.csv`; preserve conditional unlock relations rather than claiming that a level always unlocks content. `scene_num=0` is `DATA_ABNORMAL`, not room 0. Continue to scene-level data only when it is needed for an explanation or quality issue.

## Execution And Evidence

Use native ThinkingData retention and distribution endpoints for their matching models. Use a compatible configured read-only connector for the progression-chain requests. When the installing user has not provided an endpoint and token, generate the complete SQL or model-request bodies from the bundled templates and label the output `EVIDENCE_REQUIRED`; this is a runnable analysis plan, not a completed report.

For a live request, business success requires `return_code=0`; HTTP 200 alone is insufficient. Preserve the request body, raw response, normalized result, and derived report data in the caller's chosen output location. Do not fall back to browser results, cached reports, an unrelated SQL query, or a partial report when a required live source fails.

An organization may install a separate local production extension that orchestrates these same requests and evidence artifacts. Invoke it only when it is explicitly installed and configured in the current environment; its absence must not prevent this Skill from producing the portable request plan.

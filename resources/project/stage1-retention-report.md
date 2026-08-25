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

## Execution Boundary

Use the existing shared ThinkingData Tool. Native retention and distribution requests do not fall back to `querySql`; the realtime D1 SQL is the existing approved exception. A private test send requires caller authorization.

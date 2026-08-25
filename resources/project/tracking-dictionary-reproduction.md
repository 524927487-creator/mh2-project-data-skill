# Core Tracking Dictionary Reproduction

`scripts/build_tracking_dictionary.py` regenerates a readable MH2 event and event-property dictionary from `raw/代号：MH2_埋点方案_20260821.xlsx`.

```text
py scripts/build_tracking_dictionary.py resources/project/raw/代号：MH2_埋点方案_20260821.xlsx <output.md>
```

For the upstream workbook currently bundled here, the generated dictionary has 34 events and 156 event-property pairs, matching `mh2-event-index.csv` and `mh2-event-properties.csv` by technical name.

This is not a full Skill rebuild: it does not produce the CSV display/index layout, server-side mapping tables and exports, templates, model API reference, or local verified reporting and Stage1 rules. Those artifacts keep their own sources.

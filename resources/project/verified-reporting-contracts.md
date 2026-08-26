# MH2 已验证报表口径

此文件是本机 MH2 Skill 对数据组基础资源的已验证补充。默认口径与本文件冲突时，以本文件为准；用户明确指定且有业务依据的口径除外。

## 默认指标和身份

- MH2 ThinkingData 项目为 `projectId=41`。
- 默认报表和留存身份为 `{"columnDesc":"账户ID","columnName":"#account_id","tableType":"event"}`。
- 默认 DAU：事件 `login`，按 `#account_id` 去重。
- 默认新增：事件 `role_create_success`，按 `#account_id` 去重。

## 已验证 `groupBy` 规则

| 业务分组 | `groupBy` key / 字段 | `tableType` / 来源 | 已验证请求证据 |
| --- | --- | --- | --- |
| 职业 | `career` | `user` | Stage1 Golden `20260825_130113`, `requests/final_profession.json` (`/open/retention-analyze`, HTTP 200, `return_code=0`) |
| 渠道 | `channel` | `user` | Stage1 Golden `20260825_130113`, `requests/final_channel.json` (`/open/retention-analyze`, HTTP 200, `return_code=0`) |
| 勇1老用户 | `cohort_20260824_202104` | `user_cluster` / `cluster_by_import` | Stage1 Golden `20260825_130113`, `requests/final_old_new.json` (`/open/retention-analyze`, HTTP 200, `return_code=0`) |
| D1 小秘境章节 | `tag_20260824_1` | `user_cluster` / `tag_by_dynamic_condition` | Stage1 Golden `20260825_130113`, `requests/final_small_chapter_retention.json` (`/open/retention-analyze`, HTTP 200, `return_code=0`) |
| D0 小秘境 | `tag_20260824_1` | `user_cluster` / `tag_by_dynamic_condition` | Stage1 Golden `20260825_130113`, `requests/final_small_chapter_retention.json` (`/open/retention-analyze`, HTTP 200, `return_code=0`) |

- D0 小秘境分层只使用上述动态标签。对于 cohort 日期 `D`，在 `D+1` 的 01:00 标签更新后，使用该标签的 `specifiedClusterDate=D+1`。
- 历史 D0 进度分布查询：`dungeon`、`dungeon_type=1`、`dungeon_result=1`、`MAX(dungeon_id)`。它不是 D0 小秘境用户分层，不替代动态标签，也不得作为此业务名称的 `groupBy` 来源。

## 执行与返回

- 职业使用 `career`，勇1老用户使用 `cohort_20260824_202104`，D0 小秘境使用 `tag_20260824_1`；这些是新分层留存的正式分层定义。
- 新分层留存直接复用当前环境中已有的分组 builder、`retention_request` 与共享 `retention_analyze` 逐项执行；不得依赖某个本机代码路径。Stage1 仅是 Golden/历史案例，不是新查询的默认入口；只发用户要求的请求，不得调用完整 Stage1 Runner，也不得附带 D0 进度分布或实时查询。
- 用户要求实际“查”分层留存时，必须在当前会话调用真实只读 API。旧 runtime、Golden 和报告只能作历史证据，不得替代当前查询。保留本次请求、原始返回与标准化结果；本次只请求职业、勇1老用户、D0 小秘境时，必须恰好发出三份 `retention_analyze` 请求。
- 实时 ThinkingData 查询只有 `return_code=0` 才算业务成功；HTTP 200 只表示请求到达服务，不能单独作为成功依据。
- 成功答复只给日期、指标、数值和业务定义，不展示 HTTP 状态等诊断信息。
- 失败答复必须给出 `return_code` 和 `return_message`，并说明未取得业务结果。
- 对应原生能力优先使用事件、留存、漏斗或分布模型。原生模型业务失败时，不改用 SQL 兜底；应返回失败边界。

## 副本字段与映射

- 查副本配置 ID 对应的玩法、名称或配置属性时，用 `dungeon_type:dungeon_id` 这个组合键，在 `mh2-mapping-tables.md` 和 `mapping_exports/` 中查找；不可只凭裸 `dungeon_id` 推断。
- 查玩家实际大秘境进度或各层战斗结果时，筛选 `dungeon_type=2`，按事件字段 `dungeon_level` 统计。`dungeon_level` 是关卡/层数/难度等级，不需要用 `dungeon_id` 映射成层数。

## Dungeon 通用分析边界

- Dungeon 通用分析默认以 `dungeon_id` 下钻。职业、新老用户、渠道和标签等只能作为附加分组维度，不能替代 `dungeon_id`。
- 只有 `dungeon_type=1`（小秘境）支持继续按 `scene_num` 下探。`scene_num` 只表示本局到达场景数，不得自行解释为房间、Boss 或具体玩法内容。
- 小秘境行为结果只能使用服务器 `dungeon` 事件的 `dungeon_result`。不得混用 `dungeon_end` 的同名字段统计成功、战败、正常点击退出 / 主动换场景或异常退出。
- 小秘境挑战次数为 `dungeon_enter` 事件次数；参与角色为对应事件的 `#account_id` 去重；人均挑战次数为挑战次数除以参与角色数。四类状态率均为对应服务器 `dungeon` 结算次数除以同一 `dungeon_id` 的挑战次数。服务器结算少于挑战次数时，状态率之和可以不足 100%，不得改用结算次数作为分母。
- 小秘境进入到结算的链路质量按 `dungeon_id + dungeon_key` 对账，分别保留 `enter_only`、`settlement_only` 和重复服务器结算。链缺漏率为 `(enter_only_keys + settlement_only_keys) / dungeon_key 去重并集`；它是数据质量指标，不是玩家行为指标，不阻断业务结果。
- `scene_num=0` 是 `DATA_ABNORMAL`：保留原始记录并单列异常率，不映射为具体房间，不参与房间级解释，也不阻断分析。
- Agent 根据用户问题自行决定是否查询 overall、按 `dungeon_id` 下钻或小秘境的 `scene_num` 下探；不得因附加分组而退化为没有 `dungeon_id` 的 Dungeon 明细查询。
- Dungeon 静态身份使用 `dungeon_type:dungeon_id`；小秘境显示名已验收为 `小秘境 章节-章节内序号`，例如 `10101 -> 小秘境 1-1`。小秘境局内场景使用 `small_dungeon_scene.csv` 的 `dungeon_type=1 + 关卡ID + 数数通过场景数`；它只对来源表覆盖的正常值 `1..4` 有效，其他组合必须为 `unmapped`。小秘境局外绑定/解锁关系使用独立的 `small_dungeon_unlock.csv`，其键为 `dungeon_type=1 + dungeon_id` 且一关多行。三类事实必须分开：静态关卡身份、局内房间/内容、局外绑定/解锁关系；它们互相不能推断。
- `small_dungeon_unlock.csv` 是受 `small_dungeon_unlock.manifest.json` 约束的正式资源。它的关系不是“通关即解锁”结论；引用时必须保留源表的 `实际条件`、`状态`、`说明` 和 `配置证据`，尤其不得忽略等级、技能等级、孔位或条件冲突。`dungeon_type=1 + dungeon_id` 未命中返回 `NO_SOURCE_RELATION` 与空关系列表；不得按相邻关卡、章节顺序或其他映射推测补齐。
- `scene_num=0` 仍是 `DATA_ABNORMAL`，没有 `small_dungeon_scene` 正常映射；缺失组合必须停在场景进度，不得推测业务内容。

## Stage1 小秘境关卡推进指标

This section applies to the verified, per-`dungeon_id` small-dungeon progression view. It supplements the Stage1 final-success-progress distribution; it does not redefine Stage1 "驻留" as an unsuccessful/stuck-player population.

- For a user-only MH2 "首日分析", this chain is required alongside the existing Stage1 retention and final-success-progress results. Do not treat a `MAX(dungeon_id)` final-progress distribution, chapter distribution, or Top-N final stages as a substitute for the per-level chain.
- Required output at the `dungeon_id` level: entry roles, `challenge_count`, `success_role_count`, `success_count`, `success_rate`, `next_enter_roles`, `success_to_next_rate`, result-state counts, mapped level name, applicable unlock relations, and the `dungeon_key` reconciliation/data-quality summary. Preserve evidence for every real request, raw response, normalized result, and derived report.
- Use the existing shared ThinkingData query capability for this chain. A missing live result is `EVIDENCE_REQUIRED`, not permission to omit the chain or replace it with historical runtime output.
- This is an operational data query. Do not modify query code, tests, runtime, mappings, or Skill resources to make a user report contain these fields. If an existing B-line response lacks `success_role_count`, `next_enter_roles`, or `success_to_next_rate`, send the necessary direct read-only request through the existing shared capability and calculate the documented fields in the current session.
- The minimum live inputs for that direct request are: per-`dungeon_id` `dungeon_enter` distinct `#account_id` and event count; per-`dungeon_id` server `dungeon` distinct successful `#account_id`, successful event count, and result-state event counts; plus the existing `dungeon_key` reconciliation. Use `dungeon_type=1` and the same D0 cohort throughout.
- Do not default to a full `scene_num` table. Continue to `scene_num` only when a small-dungeon question needs scene-level explanation or a quality anomaly needs reporting; never use it for other `dungeon_type` values.
- Scope: D0 `role_create_success` cohort, `#account_id` as the role grain, and `dungeon_type=1`. Use `dungeon_enter` for entries/challenges and server `dungeon` for settlement results.
- `success_role_count` (成功角色数): for one `dungeon_id`, count distinct `#account_id` in server `dungeon` records where `dungeon_result=1`.
- `success_count` (成功次数): for the same `dungeon_id`, count server `dungeon` settlement events where `dungeon_result=1`. One role can succeed multiple times, so `success_role_count` and `success_count` must never be mixed.
- `challenge_count` (挑战次数): count `dungeon_enter` events for the same `dungeon_id`.
- `success_rate` (当前关成功率) = `success_count / challenge_count`.
- `next_enter_roles` (下一实际输出关卡进入角色数): count distinct `#account_id` in `dungeon_enter` for the next actual output level. It is not a strict immediate-next-level relation from game configuration.
- `success_to_next_rate` (成功到下一关进入率) = `next_enter_roles / success_role_count`. Its meaning is the share of successful roles observed entering the next actual output level; do not label it as a strict configured-next-level conversion rate.
- For the final actual output level, `next_enter_roles` and `success_to_next_rate` are blank. Its derived residency uses its own entry-role count and cumulative share as the endpoint so the displayed per-level residency shares sum to 100%; do not substitute zero.
- Map displayed levels with `dungeon_type:dungeon_id`. For Stage1's `关卡中文名`, first use the matched `small_dungeon_unlock.csv` `关卡` value and only then fall back to the static formal `dungeon_id.csv` display name. When an unlock relation exists, preserve its conditional, non-causal meaning.

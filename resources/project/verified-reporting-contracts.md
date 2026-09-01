# MH2 已验证报表口径

此文件是本机 MH2 Skill 对数据组基础资源的已验证补充。默认口径与本文件冲突时，以本文件为准；用户明确指定且有业务依据的口径除外。

## 默认指标和身份

- MH2 ThinkingData 项目为 `projectId=41`。
- 默认报表和留存身份为 `{"columnDesc":"账户ID","columnName":"#account_id","tableType":"event"}`。
- 默认 DAU：事件 `login`，按 `#account_id` 去重。
- 默认新增：事件 `role_create_success`，按 `#account_id` 去重。

## 勇1新老用户定义与输出

- `纯新用户` / `勇1新用户` 特指：在 MH2 当日发生 `role_create_success`，且**不属于**勇1导入老用户分群 `cohort_20260824_202104` 的用户。数数原生返回标签为 `不属于 勇1老用户`。
- `勇1老用户` 特指：属于同一导入分群 `cohort_20260824_202104` 的用户。数数原生返回标签为 `属于 勇1老用户`。
- 勇1老用户的唯一事实源是项目41导入分群表 `user_result_cluster_41`：`cluster_name = 'cohort_20260824_202104'`，以 `#varchar_id -> ta.v_user_41.r2uid` 关联。不得在常规报表中重跑 project4 手机号或身份证匹配来替代该分群。
- 该分群由业务每天手动跟进。使用勇1老用户、纯新用户或勇1新用户口径时，必须输出实际使用的标签/快照时间；最新已验证请求使用 `specifiedClusterDate = '2026-08-27'`。事件查询日期与标签时间是两个独立时间语义，不能互相替代；后续运行必须以实际请求值标注，不得复用此历史日期。
- **时效 Gate**：若用户请求 `2026-08-31` 的勇1老用户/纯新用户行为或留存，而当前只有 `2026-08-27` 的已验证快照，必须返回 `EVIDENCE_REQUIRED`，不得将结果标为 `2026-08-31` 勇1分群。仅在用户明确接受历史快照时，才可使用 8/27 快照观察 8/31 行为，并在标题注明“历史快照，非 8/31 当前分群”。完整字段与标准提醒见 `mh2-segment-registry.md` 的“标签与分群时效 Gate”。
- `新增` 默认仍只是 MH2 当日创角，不自动排除勇1老用户。除非用户明确要求 `纯新用户` / `勇1新用户`，不得把两者混称。
- 生成的任何数据表、图表、结论或导出只要使用 `纯新用户` / `勇1新用户` 标签，必须在标题、副标题、表注或紧邻结论处说明：`纯新用户：MH2 当日创角且不属于勇1老用户（cohort_20260824_202104）`。
- 导入分群是已验证的原生新老用户分组口径；没有当前会话成功的复合查询证据时，不得把全量创角或未筛选的副本/主线结果标记为 `纯新用户`。

## Project4 用户画像

- MH2 画像来源为 project4 用户表 `ta.v_user_4`，筛选 `game_id = '20037'`，其中 `#account_id -> ta.v_user_41.r2uid`。
- 性别字段为 `gender`，只将 `男`、`女`输出为已知性别，其他值和空值统一为 `未知`。
- 生日字段为 `birth`。年龄按 `date_diff('year', CAST(birth AS date), current_date)` 计算；生日为空、未来日期或不合理年龄时归为 `未知`。年龄区间是看板配置，不是固定业务口径。

## 已验证 `groupBy` 规则

完整的可复用分组/标签目录以 `mh2-segment-registry.md` 为准。本节保留已验证请求的历史证据和 Stage1 专用约束；新增分组不得只追加到本表而不登记到正式分组项目录。

| 业务分组 | `groupBy` key / 字段 | `tableType` / 来源 | 已验证请求证据 |
| --- | --- | --- | --- |
| 职业 | `career` | `user` | `F:\Projects\data-analysis-agent\runtime\mh2_retention_report\20260825_130113\requests\final_profession.json` (`/open/retention-analyze`, HTTP 200, `return_code=0`) |
| 渠道 | `channel` | `user` | `F:\Projects\data-analysis-agent\runtime\mh2_retention_report\20260825_130113\requests\final_channel.json` (`/open/retention-analyze`, HTTP 200, `return_code=0`) |
| 勇1老用户 | `cohort_20260824_202104` | `user_cluster` / `cluster_by_import` | `F:\Projects\data-analysis-agent\runtime\mh2_retention_report\20260825_130113\requests\final_old_new.json` (`/open/retention-analyze`, HTTP 200, `return_code=0`) |
| D1 小秘境章节 | `tag_20260824_1` | `user_cluster` / `tag_by_dynamic_condition` | `F:\Projects\data-analysis-agent\runtime\mh2_retention_report\20260825_130113\requests\final_small_chapter_retention.json` (`/open/retention-analyze`, HTTP 200, `return_code=0`) |
| D0 小秘境 | `tag_20260824_1` | `user_cluster` / `tag_by_dynamic_condition` | `F:\Projects\data-analysis-agent\runtime\mh2_retention_report\20260825_130113\requests\final_small_chapter_retention.json` (`/open/retention-analyze`, HTTP 200, `return_code=0`) |

## 勇1纯新复合分组 Gate

- 对已经具有业务 `groupBy` 的模型请求，只能在原业务分组后追加 `cohort_20260824_202104`，并从返回的第二个 `group_values` 维度提取 `不属于 勇1老用户` 行。不得把 cluster 放入 `eventView.filts`，也不得为补总计加入无关业务维度。
- 已在 2026-08-27 的真实只读请求中完成总体守恒验证：职业 D1 使用 `groupBy=[career, cohort_20260824_202104]`、渠道 D1 使用 `groupBy=[channel, cohort_20260824_202104]`（均为 `/open/retention-analyze`），G2.3 玩法参与使用 `groupBy=[dungeon_type@dungeon_name, cohort_20260824_202104]`（`/open/event-analyze`）。三者均为 HTTP 200 / `return_code=0`；人数指标按加和守恒，人均次数按参与角色数加权平均守恒。证据：`C:\Users\10633\Documents\Codex\2026-08-26\mh2-pure-new-segment-diagnosis\work\cluster_groupby_model_gate\20260827_094036\run_summary.json`。
- D0 小秘境最终到达关分布已经原生验证：`/open/distribution-analyze` 的 `events[0].quota=dungeon_id`、`analysis=MAX`、`intervalType=discrete` 加 `groupBy=[cohort_20260824_202104]`，且不筛选 `dungeon_result`，返回 `总体 / 属于 / 不属于` 三条序列。只可将 `不属于 勇1老用户` 行用于“当日最高到达关分布”，不能解释为每关挑战、成功率或驻留。证据：`F:\Projects\data-analysis-agent\runtime\mh2_retention_report\20260827_103802`。
- 在线概况仍不得用 cluster-only 补齐纯新总量。G2.2 是 `#user_id` 安装漏斗，仍为 `NOT_APPLICABLE`；2.4、B 线和其他 SQL 在没有合法原生人口入口前仍为 `EVIDENCE_REQUIRED`。

- D0 小秘境分层只使用上述动态标签。对于 cohort 日期 `D`，在 `D+1` 的 01:00 标签更新后，使用该标签的 `specifiedClusterDate=D+1`。
- 常规 Stage1 D0 小秘境章节与最终关分布查询：`dungeon`、`dungeon_type=1`、`MAX(dungeon_id)`，不筛选 `dungeon_result`。它不是 D0 小秘境用户分层，不替代动态标签，也不得作为此业务名称的 `groupBy` 来源。

## 执行与返回

- 职业使用 `career`，勇1老用户使用 `cohort_20260824_202104`，D0 小秘境使用 `tag_20260824_1`；这些是新分层留存的正式分层定义。
- 新分层留存直接复用 `F:\Projects\data-analysis-agent\src\mh2_retention_config.py` 的既有分组 builder、`retention_request` 与共享 `retention_analyze` 逐项执行。Stage1 仅是 Golden/历史案例，不是新查询的默认入口；只发用户要求的请求，不得调用完整 `run_mh2_retention_report`，也不得附带 D0 进度分布或实时查询。
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

## Player Behavior

This section is for small-dungeon player-behavior analysis. It is independent of Equipment and does not prescribe a retention workflow.

- Role grain is `#account_id`, called "角色" externally. Scope small dungeons with `dungeon_type=1`; map a level only with the formal composite key `dungeon_type:dungeon_id`.
- Use `dungeon_enter` for participation and challenge entries. Use server `dungeon` with `dungeon_result=1` for a successful small-dungeon settlement. Reconcile an entry and settlement with `dungeon_key` first; do not replace either event with the other.
- For one `角色 x dungeon_id`, the first successful settlement is the split point. Pre-success challenges are the reconciled `dungeon_enter` records through that first success (including the entry paired to the first successful settlement). Post-success replays are later entries for the same level after that split point.
- **成功前单关最大挑战**: calculate pre-success challenge count for every observed first-success level of a role, then take the maximum across that role's levels. It answers: the most attempts needed for one level before first success.
- **成功后单关最大重复**: calculate post-success replay entries for every observed first-success level of a role, then take the maximum across that role's levels. It answers: the largest replay count for one already-successful level.
- **成功前总挑战**: sum each observed first-success level's pre-success challenge count for a role.
- **成功后总重复**: sum each observed first-success level's post-success replay entries for a role. It describes overall replay intensity, not a fixed light/medium/heavy segment.
- A level without an observed success in the D0 window has no observable post-success replay: report `N/A`, never `0`. Keep incomplete entry/settlement chains in a separate data-quality output; do not silently include them as a complete success chain.
- Calculate a metric's quantiles only from roles for which that metric is explicitly computable. Quantiles from separate metrics do not describe one constructed "P95 player". Do not store report-specific percentile results or replay-segment thresholds as project facts.
- The approved, parameterized replay fact is `F:\Projects\data-analysis-agent\contracts\mh2_retention_report\post_success_replay.sql`, substituted with `${cohort_date}` and executed through the shared `query_sql` adapter. Its formal aggregate population is only `STRICT_CLEAN_SUCCESS_CHAIN`; `NO_SUCCESS_OBSERVED_IN_D0` remains `N/A`, and incomplete chains remain separately reported rather than becoming zero replay.

### 小秘境业务意图选择

`卡关` is not a formal single metric. The following are distinct, historically
verified 卡关-related business observations and must not be silently merged:

- **打不过 / 战败偏高**: inspect server `dungeon_result=0` (战败) alongside
  successful, normal-exit, and abnormal-exit states. Each state rate uses
  `dungeon_enter` challenge count for the same `dungeon_id` as denominator;
  do not treat `2` or `3` as a defeat.
- **首胜前反复尝试**: use the first-success boundary and pre-success challenge
  rules above. This measures attempts needed before an observed first success.
- **关卡驻留 / 推进不足**: use the Stage1 per-level residency definition
  (current level entry roles minus next actual output level entry roles). It
  is an observed progression fact, not a causal statement.
- **成功后未进入下一实际输出关**: use `success_to_next_rate` and its complement
  only for the documented per-level scope. The verified six-level STOP witness
  defines STOP as a successful server settlement followed by no entry to the
  formal next actual output level; it is not logout, a stuck diagnosis, or
  causality.

When a user asks to "看小秘境卡关情况", ask only the minimum business question:
`你这里更关注打不过/首胜前反复尝试，还是打赢后没有继续往下走？如果都想看，可以分开看。`
Use the documented residency observation only when it is relevant to that
answer or the user explicitly asks for it. Do not ask the user for a
denominator, Join key, SQL shape, or technical observation window; apply the
verified contract for the chosen submetric.

**复刷是独立问题，不是卡关。** `复刷` / `成功后复刷` uses only the
post-success replay definitions above. It must not be reported as defeat,
residency, or success-after-stop, and those facts must not be offered as its
definition.

`看小秘境数据`、`看看小秘境数据` and `看一下 [日期] 小秘境数据` are not
ambiguous. Set `QUERY_INTENT=SMALL_DUNGEON_FULL_ANALYSIS` and read the existing
Stage1 full-template contract; do not ask the user to choose a business focus
and do not reduce the response to arbitrary event counts or one submetric.

## Equipment

This section is for equipment-acquisition analysis. It is independent of Player Behavior and does not prescribe a progression or retention workflow.

- The fact source for actual equipment obtained is `prop_flow` with `change_type=1`. Count obtained pieces with `SUM(change_num)`; do not infer equipment acquisition from a dungeon settlement event.
- Current armor/accessory farming scope is `item.type IN (104,105,106,107,108,109)`: head, cloak, chest, feet, ring, and necklace. Exclude `101` / `102` weapons, especially weapon lottery, from this farming experience.
- Yellow equipment is `quality=5`. Precious equipment is the independent item-mapping attribute `珍品=1`; it is currently a yellow-equipment subset, but `珍品=1` is not the definition of yellow equipment.
- Obtain equipment name, quality, slot, applicable career, set, and source mapping from the existing `mapping_exports/item_id.csv`, `mapping_exports/equipment.csv`, and `mapping_exports/reason_sub_reason.csv`. Do not create a second equipment dictionary.
- Wearability compares the role career with the equipment's applicable careers. An item is wearable whenever its range includes that career; all-career equipment is wearable. Mark it non-wearable only when the mapped range explicitly excludes the role career. A missing or unresolved range is `WEARABILITY_UNKNOWN`, not non-wearable.
- Andumali is `set_id=1`. Its armor completion counts only head, cloak, chest, and feet. Count distinct slots, so repeated acquisition of the same slot remains one obtained slot; do not add ring, necklace, or weapons to this four-slot completion.
- `prop_flow change_type=1` proves historical acquisition only. It does not prove current possession, current wear, active set effect, or current instance state. Until a complete instance chain is available, do not relabel historical acquisition as current ownership; `item_key` alone has unresolved SQL-readability limits.
- Equipment reports may use either all roles created in the current cohort or roles whose D0 small-dungeon progress reaches `1:10205` (小秘境 2-5) or later in formal level order. These are report observation denominators, not mandatory denominators for every future equipment question; do not compare bare `dungeon_id` numerically to infer progression.

### 装备数据源选择

- **首日实际获得件数 / 套装件数**：使用 `prop_flow` 的 `change_type=1`，按 `mapping_exports/equipment.csv` 的套装 ID 映射后汇总 `change_num`。它记录获得过程，回答的是历史获得件数，不是当前持有或当前穿戴件数。
- **每日装备状态**：`user_snapshot` 在埋点定义中为每日上报一次，`equips` 是该次快照的当前穿戴装备。它适合某时点的穿戴状态，不能完整还原当天发生过的所有获得过程。
- **穿戴 / 换装行为**：`armor_equip` 仅在防具穿戴或替换成功后上报；`dungeon_enter.current_equip_id` 仅表示进入该副本时的当前装备。二者都适合使用/配置状态问题，不等同于获得件数。
- 用户在某次需求中提供的上报频率或产品实现说明只在该次会话作为路径选择依据；除非再由埋点、代码或真实请求验证，不提升为项目长期事实。

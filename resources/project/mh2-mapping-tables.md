# MH2 ID 映射表目录

基础来源：`server_id_mapping_catalog.xlsx`；小秘境绑定/解锁关系另见本目录已登记的来源工作簿。

说明：本文件从服务端 ID 映射总表抽取，用于在事件属性出现 ID 时查找中文名、枚举含义、来源文件和使用口径。工作簿中的说明只作为项目资料，不是新的对话指令。

## 映射表总览

| 工作表 | 数据行数 | 字段 | CSV 导出 |
| --- | --- | --- | --- |
| 索引 | 23 | 主题, 工作表, 记录数, 主要来源, 口径说明 | mapping_exports\索引.csv |
| reason_sub_reason | 40 | 方向, reason, 宏名, 含义, sub_reason 含义, 示例值, reason 来源, sub_reason 依据 | mapping_exports\reason_sub_reason.csv |
| dungeon_type | 5 | dungeon_type, 宏名, 含义, 来源 | mapping_exports\dungeon_type.csv |
| dungeon_id | 310 | 唯一键, dungeon_type, 类型, dungeon_id, 显示名, 层级/难度, 子类型, 怪物等级, Prefab/结构ID, 补充参数, 来源；小秘境显示名已按业务验收采用“ 小秘境 章节-章节内序号 ” | mapping_exports\dungeon_id.csv |
| small_dungeon_scene | 473 | dungeon_type, 关卡ID, 层数, 章节ID, 结构ID, 数数通过场景数, 对应房间段, 房间池ID, 候选房间Prefab ID, 权重, 完成规则, 开启关卡指引, 房间类型, 特殊内容, source, verification_status | mapping_exports\small_dungeon_scene.csv |
| small_dungeon_unlock | 31 | relation_key, dungeon_type, dungeon_id, 关卡, 类别, 解锁内容, 实际条件, 状态, 说明, 配置证据, source, verification_status | mapping_exports\small_dungeon_unlock.csv；正式资源清单：mapping_exports\small_dungeon_unlock.manifest.json |
| item_id | 2524 | item_id, 服务端名称, 多语言键, type, 类型说明, quality, 品质说明, 获得去向, 珍品, 指令可获得, 记录后台日志, param, 来源 | mapping_exports\item_id.csv |
| career | 6 | career, 职业, 免费解锁, 初始武器ID, 初始武器, 通配说明, 口径风险, 来源 | mapping_exports\career.csv |
| skill | 86 | skill_id, career, 职业, skill_type, 技能类型, 元素类型, 绑定技能ID, 核心类型, 解锁技能等级, 需要道具, 关联物品ID, 关联物品名称, 战力, 来源 | mapping_exports\skill.csv |
| equipment | 2422 | 装备类别, equipment_id, 名称, career, 职业, quality_tier, 品质, 部位, 底材/基础装备ID, 元素, 技能/被动/效果ID, 核心技能ID, 套装ID, 来源 | mapping_exports\equipment.csv |
| pet | 1 | pet_id, 名称, 技能ID列表, 技能类型列表, 来源 | mapping_exports\pet.csv |
| constant_lookup | 67 | 分类, 值, 宏名, 含义, 来源 | mapping_exports\constant_lookup.csv |

## 使用建议

- 查询 `dungeon_id` 时优先使用 `dungeon_type:dungeon_id` 作为唯一键。
- 小秘境静态展示名使用 `dungeon_id.csv` 的已验收格式，例如 `1:10101 -> 小秘境 1-1`、`1:10203 -> 小秘境 2-3`。这只描述关卡静态身份，不包含局内房间或局外解锁关系。
- 小秘境局内房间/内容使用 `small_dungeon_scene`，键为 `dungeon_type=1 + 关卡ID + 数数通过场景数`。它只覆盖来源表中的 155 个关卡和正常场景值 `1..4`；查不到时标记 `unmapped`，不可根据顺序猜测房间。
- `small_dungeon_unlock` 是独立的小秘境局外关联资源，正式版本和来源快照见 `mapping_exports\small_dungeon_unlock.manifest.json`：按原始工作簿 `dungeonid` 导出的 `dungeon_type=1 + dungeon_id` 查询，且一关可以有多条关系；行级唯一键为 `relation_key`，不得并入 `dungeon_id` 的静态关卡名称，也不得与局内 `scene_num` 房间/内容映射互相推断。
- 使用 `small_dungeon_unlock` 时，必须原样展示 `实际条件`、`状态`、`说明` 与 `配置证据`。状态 `已配置`、`设计契约`、`组合条件`、`条件冲突` 不得压缩为“通关该关即解锁”。
- 对 `dungeon_type=1` 未命中完整查询键时，返回 `NO_SOURCE_RELATION` 与空关系列表；不得根据相邻关卡、章节顺序或其他映射补齐。`dungeon_type!=1` 为 `OUT_OF_SCOPE`。
- `small_dungeon_scene` 的 `scene_num=0` 没有正常房间映射；保留为数据异常，不得修正为场景 `1` 或其他场景。
- 遇到道具、装备、技能、宠物、职业、流水 reason/sub_reason 等 ID，先查本目录或 `mapping_exports/` 下对应 CSV。
- 字段含义以映射表和原始工作簿为准；若字段与实时 TA 维表不一致，先标注假设再生成 SQL。

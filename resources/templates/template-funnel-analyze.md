# 漏斗分析配置模板

适用场景：生成 ThinkingData / 数数「漏斗分析」模型查询请求体，或作为 `使用本 skill 执行` 的 Markdown 任务配置。

## 请求 JSON

请把 漏斗分析 请求体直接写在下面的 JSON 代码块中。

日期范围通常在 `eventView` 中设置：相对日期用 `recentDay` / `recent_day`，固定日期范围用 `startTime` + `endTime` 或 `from_date` + `to_date`。不要在同一个请求中混用相互冲突的时间范围。

<!-- request -->
```json
{
  "eventView": {
    "endTime": "2021-10-31 23:59:59",
    "filts": [
      {
        "columnDesc": "城市",
        "columnName": "city",
        "comparator": "equal",
        "filterType": "SIMPLE",
        "ftv": [
          "上海市",
          "北京市",
          "广州市",
          "深圳市"
        ],
        "specifiedClusterDate": "2022-01-24",
        "tableType": "user",
        "timeUnit": ""
      }
    ],
    "groupBy": [
      {
        "columnDesc": "城市",
        "columnName": "city",
        "propertyRange": "",
        "specifiedClusterDate": "2022-01-24",
        "tableType": "user"
      }
    ],
    "recentDay": "",
    "relation": "and",
    "startTime": "2021-10-01 00:00:00",
    "taIdMeasureVo": {
      "columnDesc": "账户ID",
      "columnName": "#account_id",
      "tableType": "event"
    },
    "windows_gap": 1,
    "windows_gap_tu": "hour"
  },
  "events": [
    {
      "eventName": "register",
      "eventNameDisplay": "",
      "filts": [
        {
          "columnDesc": "app版本",
          "columnName": "app_version",
          "comparator": "equal",
          "filterType": "SIMPLE",
          "ftv": [
            "V1.0"
          ],
          "specifiedClusterDate": "2022-01-26",
          "tableType": "event",
          "timeUnit": ""
        }
      ],
      "relation": "and"
    },
    {
      "eventName": "login",
      "eventNameDisplay": "",
      "filts": [],
      "relation": "and"
    },
    {
      "eventName": "activity_attend",
      "eventNameDisplay": "",
      "filts": [],
      "relation": "and"
    },
    {
      "eventName": "logout",
      "eventNameDisplay": "",
      "filts": [],
      "relation": "and"
    }
  ],
  "projectId": 377,
  "limit": 2,
  "timeoutSeconds": 10,
  "useCache": true,
  "zoneOffset": 10
}
```

## Skill Markdown 配置示例

```json
[
  {
    "name": "funnel-analyze",
    "enabled": true,
    "action": "funnel-analyze",
    "mode": "once",
    "request": "__markdown_json__"
  }
]
```

安装本 skill 后，可让 ChatGPT / Agent 读取本 Markdown 任务并生成或执行对应请求。

## 参数说明（来自数数模型查询 API 模板）

| 参数名 | 示例值 | 参数类型 | 是否必填 | 参数描述 |
| --- | --- | --- | --- | --- |
| eventView | - | Object | 是 | 分组属性 |
| ∟col_limit | 10 | Integer | 是 | 列限制，0到20之间 |
| ∟from_date | 44470 | String | 否 | 开始时间 yyyy-MM-dd HH:mm:ss |
| ∟recent_day |  | String | 否 | 相对时间 |
| ∟session_interval | 30 | Integer | 是 | 会话间隔时长 |
| ∟session_type | minute | String | 是 | 会话间隔时长单位: second, minute, hour |
| ∟to_date | 44471.9999884259 | String | 否 | 结束时间 yyyy-MM-dd HH:mm:ss |
| events | - | List | 是 | 事件指标列表 |
| ∟by_fields | - | List | List | 按事件属性拆分 |
| ∟event_name | login | String | 是 | 事件名 |
| ∟field | browser | String | 是 | 拆分字段 |
| ∟range_type | - | String | 否 | 区间间隔类型 |
|  |  |  |  | •discrete：离散数字 |
|  |  |  |  | •def：默认区间 |
|  |  |  |  | •user_defined：用户自定义 |
| ∟range | - | String | 否 | 区间 |
| ∟table_type | event | String | 是 | 表类型，event：事件表，user：用户表 |
| ∟event_names | ["logout","login"] | String | 是 | 事件名称，特别的，可以使用 anyEvent 表示任意事件 |
| ∟source_event | - | Object | 是 | 源事件 |
| ∟event_name | login | String | 是 | 事件名 |
| ∟filter | - | Object | 否 | 源事件过滤 |
| ∟filterType | COMPOUND | String | 否 | 过滤模式，SIMPLE：简单，COMPOUND：复合 |
| ∟source_type | initial_event | String | 是 | 事件类型，initial_event，termination_event |
| ∟user_filter | - | Object | 否 | 用户表过滤 |
| ∟filterType | COMPOUND | String | 否 | 过滤模式，SIMPLE：简单，COMPOUND：复合 |
| ∟filts | - | List | 否 | 条件列表列表 |
| ∟columnDesc | app_version | String | 否 | 字段显示名 |
| ∟columnName | app_version | String | 是 | 字段名称 |
| ∟comparator | equal | String | 是 | 参考： 模型查询API的筛选表达式 |
| ∟filterType | SIMPLE | String | 否 | 过滤模式，SIMPLE：简单，COMPOUND：复合 |
| ∟ftv | ["V1.0"] | List | 否 | 用于属性比较边界的字面常量 |
| ∟specifiedClusterDate | 44587 | String | 否 | 指定对应日期的标签历史版本 |
| ∟tableType | event | String | 是 | 表类型，event：事件表，user：用户表 |
| ∟relation | and | String | 否 | 逻辑关系，and：逻辑与，or：逻辑或 |
| projectId | 377 | Integer | 是 | 项目ID |
| timeoutSeconds | 10 | Integer | 否 | 请求超时参数，超时则取消查询任务 |
| useCache | 1 | Boolean | 是 | true为使用缓存 |
| zoneOffset | 10 | Integer | 否 | 时区 |

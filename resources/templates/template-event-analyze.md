# 事件分析配置模板

适用场景：按事件、指标、分组、筛选条件进行事件分析，或下载事件分析结果。

## 请求 JSON

请把事件分析请求体直接写在下面的 JSON 代码块中。

日期范围写法和事件分析/用户列表下钻一致：

~~~text
近 7 天：  "recentDay": "1-7"
近 30 天： "recentDay": "1-30"
固定范围： "startTime": "2026-05-01 00:00:00", "endTime": "2026-05-07 23:59:59"
~~~

相对日期优先使用 `recentDay`；固定日期范围使用 `startTime` + `endTime`。两种写法不要同时放在同一个 `eventView` 中。

<!-- request -->
~~~json
{
  "projectId": 377,
  "eventView": {
    "recentDay": "1-7",
    "timeParticleSize": "day",
    "relation": "and",
    "groupBy": [{
      "columnName": "brand",
      "tableType": "event"
    }],
    "filts": [{
      "columnName": "brand",
      "comparator": "equal",
      "ftv": ["Apple", "Xiaomi"],
      "tableType": "event"
    }]
  },
  "events": [{
    "analysis": "TOTAL_TIMES",
    "analysisParams": "",
    "eventName": "login",
    "filts": [],
    "relation": "and",
    "type": "normal"
  }],
  "useSameResultKey": false,
  "useCache": true,
  "limit": 1000,
  "timeoutSeconds": 10,
  "zoneOffset": 10
}
~~~

## Skill Markdown 配置示例

~~~json
[
  {
    "name": "event-analyze-download",
    "enabled": true,
    "action": "event-analyze-download",
    "mode": "once",
    "request": "__markdown_json__"
  }
]
~~~

安装本 skill 后，可让 ChatGPT / Agent 读取本 Markdown 任务并生成或执行对应请求。

## 接口说明

~~~text
Read event analysis parameters from a JSON file, or from stdin with "-".

Endpoint: POST /open/event-analyze?token=$TA_USER_TOKEN

Example:
```json
{
  "projectId": 377,
  "eventView": {
    "recentDay": "1-7",
    "timeParticleSize": "day",
    "relation": "and",
    "groupBy": [{
      "columnName": "brand",
      "tableType": "event"
    }],
    "filts": [{
      "columnName": "brand",
      "comparator": "equal",
      "ftv": ["Apple", "Xiaomi"],
      "tableType": "event"
    }]
  },
  "events": [{
    "analysis": "TOTAL_TIMES",
    "analysisParams": "",
    "eventName": "login",
    "filts": [],
    "relation": "and",
    "type": "normal"
  }],
  "useSameResultKey": false,
  "useCache": true,
  "limit": 1000,
  "timeoutSeconds": 10,
  "zoneOffset": 10
}
```

Required body fields:
  - projectId: TE project ID.
  - eventView: common query settings.
  - eventView.timeParticleSize: minute|minute5|minute10|hour|day|week|month|total.
  - eventView time range: provide recentDay (for example "1-7") or startTime + endTime.
  - events: one or more metrics. Use type "normal" with analysis, or type "customized" with customEvent.
  - useCache: true or false.

Common eventView fields:
  - relation: and|or for global filters.
  - comparedByTime plus comparedRecentDay, or comparedStartTime + comparedEndTime, enables time comparison.
  - eventSplit: split one event by a property; metrics can reference it with eventSplitIndexes.
  - groupBy: list of grouping properties, each with columnName and tableType.
  - filts: global filters, each with columnName, comparator, ftv, and tableType.
  - queryFeature.approximateOn: enable approximate calculation.

Normal metric fields:
  - eventName: event name; "anyEvent" means any event.
  - analysis: aggregation enum listed below.
  - analysisParams: required for PERCENTILE, value 1-100.
  - quota: property name required by property-based aggregations such as SUM or AVG.
  - filts: metric-level filters.
  - quotaEntities: ID system config, for example taIdMeasure.columnName "#account_id".

analysis enum:
  - TOTAL_TIMES: total event count; quota not required.
  - TRIG_USER_NUM: trigger user count; quota not required.
  - PER_CAPITA_TIMES: average event count per user; quota not required.
  - SUM: numeric property sum; quota required.
  - AVG: numeric property average; quota required.
  - PER_CAPITA_NUM: numeric property average per user; quota required.
  - MAX: numeric property maximum; quota required.
  - MIN: numeric property minimum; quota required.
  - DISTINCT: distinct property count; quota required.
  - TRUE: boolean true count; quota required.
  - FALSE: boolean false count; quota required.
  - IS_NOT_EMPTY: non-empty property count; quota required.
  - IS_EMPTY: empty property count; quota required.
  - ARRAY_DISTINCT: distinct whole-list count; quota required.
  - ARRAY_SET_DISTINCT: distinct element-set count; quota required.
  - ARRAY_ITEM_DISTINCT: distinct list-item count; quota required.
  - MEDIAN: numeric property median; quota required.
  - PERCENTILE: numeric property percentile; quota and analysisParams required.

Customized metric fields:
  - type: "customized".
  - customEvent: formula, for example "logout.PER_CAPITA_TIMES" or "$metric.metricName/event.analysis".
  - eventName: display metric name.
  - customFilters, quotaEntities, quotaTimeRanges, format are optional.

Other optional top-level fields:
  - useSameResultKey: keep same event result key when event names repeat.
  - limit: group count limit per metric, default 1000, max 10000.
  - timeoutSeconds: cancel query after timeout.
  - zoneOffset: timezone offset.
~~~

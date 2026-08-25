# 用户列表下钻配置模板

适用场景：从事件分析结果下钻用户列表，或下载用户列表文件。

## 请求 JSON

请把用户列表下钻请求体直接写在下面的 JSON 代码块中。

日期范围写法和事件分析/用户列表下钻一致：

~~~text
近 7 天：  "recentDay": "1-7"
近 30 天： "recentDay": "1-30"
固定范围： "startTime": "2026-05-01 00:00:00", "endTime": "2026-05-07 23:59:59"
~~~

相对日期优先使用 `recentDay`；固定日期范围使用 `startTime` + `endTime`。两种写法不要同时放在同一个 `eventView` 中。

用户列表按天结果下钻时，额外填写 `sliceDate`，例如 `"sliceDate": "2026-05-07"`；总计下钻可以不填 `sliceDate`。

<!-- request -->
~~~json
{
  "projectId": 377,
  "eventView": {
    "recentDay": "1-7",
    "timeParticleSize": "day",
    "groupBy": [{
      "columnName": "#city",
      "tableType": "event"
    }]
  },
  "events": [{
    "analysis": "TRIG_USER_NUM",
    "eventName": "consume_item",
    "quota": "",
    "relation": "and",
    "type": "normal"
  }],
  "sliceDate": "2021-12-27",
  "sliceGroupVal": ["Beijing"],
  "eventIndex": 0,
  "timeoutSeconds": 10,
  "zoneOffset": 10
}
~~~

## Skill Markdown 配置示例

~~~json
[
  {
    "name": "event-user-list-download",
    "enabled": true,
    "action": "event-user-list-download",
    "mode": "once",
    "request": "__markdown_json__"
  }
]
~~~

安装本 skill 后，可让 ChatGPT / Agent 读取本 Markdown 任务并生成或执行对应请求。

## 接口说明

~~~text
Drill down from an event analysis result to the user list.

Endpoint: POST /open/event-user-list?token=$TA_USER_TOKEN

Use the same projectId, eventView, and events structure as event-analyze, then add drill-down fields:
```json
{
  "projectId": 377,
  "eventView": {
    "recentDay": "1-7",
    "timeParticleSize": "day",
    "groupBy": [{
      "columnName": "#city",
      "tableType": "event"
    }]
  },
  "events": [{
    "analysis": "TRIG_USER_NUM",
    "eventName": "consume_item",
    "quota": "",
    "relation": "and",
    "type": "normal"
  }],
  "sliceDate": "2021-12-27",
  "sliceGroupVal": ["Beijing"],
  "eventIndex": 0,
  "timeoutSeconds": 10,
  "zoneOffset": 10
}
```

Additional fields:
  - sliceDate: date to drill down, for example "2021-12-27". Optional for total queries.
  - sliceGroupVal: selected group values, required. Use the exact group labels returned by event analysis.
  - eventIndex: metric index to drill down, starting from 0.
  - timeoutSeconds: optional query timeout.
  - zoneOffset: optional timezone offset.
~~~

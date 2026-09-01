# Skill 任务模板

这个模板用于让安装了本 skill 的 ChatGPT / Agent 识别一份查询任务。可以直接复制一段改成自己的任务，也可以把 SQL 或请求 JSON 拆到单独文件里再引用。

支持的 action：`sql`、`event-analyze`、`event-analyze-download`、`event-user-list`、`event-user-list-download`、`retention-analyze`、`funnel-analyze`、`distribution-analyze`、`path-analyze`、`interval-analyze`、`attribute-analyze`。

常见放置方式：

- `tasks/*.md`：Markdown 任务模板，里面直接放 SQL 或请求 JSON 代码块。
- `sql/*.sql`：SQL 独立文件。
- `json/*.json`：事件分析或模型分析请求文件。

## 1. SQL 任务模板

```sql
select
  "#account_id" as "账号ID",
  count(*) as "次数"
from ta.v_event_10
where "$part_event" = 'login'
  and ${PartDate:yesterday}
group by "#account_id"
order by "次数" desc
limit 100
```

```json
[
  {
    "name": "daily-sql-analysis",
    "enabled": true,
    "action": "sql",
    "mode": "once",
    "sql": "__markdown_sql__",
    "output": {"format": "xlsx", "dir": "output"}
  }
]
```

如果 SQL 单独放文件里，就保存成 `sql/daily-sql-analysis.sql`，然后把任务里的 `sql` 改成 `input`：

```json
{"input": "sql/daily-sql-analysis.sql"}
```

## 2. 事件分析任务模板

```json
{
  "projectId": 10,
  "eventView": {
    "recentDay": "1-7",
    "timeParticleSize": "day",
    "relation": "and",
    "groupBy": []
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
  "zoneOffset": 8
}
```

```json
[
  {
    "name": "daily-event-analyze",
    "enabled": true,
    "action": "event-analyze",
    "mode": "once",
    "request": "__markdown_json__",
    "output": {"format": "xlsx", "dir": "output"}
  }
]
```

## 3. 模型分析任务模板

留存、漏斗、分布、路径、间隔、属性分析都使用同样结构：上方是请求 JSON，下方是任务配置 JSON。

```json
[
  {
    "name": "funnel-analyze",
    "enabled": true,
    "action": "funnel-analyze",
    "mode": "once",
    "request": "__markdown_json__",
    "output": {"format": "xlsx", "dir": "output"}
  }
]
```

## 查询时间参数

SQL 中可以使用：

```text
${PartDate:today}                    今天
${PartDate:yesterday}                昨天
${PartDate:last7days}                近 7 天，不含今天
${PartDate:last30days}               近 30 天，不含今天
${PartDate:2026-05-01}               指定某一天
${PartDate:2026-05-01..2026-05-07}   自定义日期范围，包含开始和结束日期
```

事件分析/模型分析的日期通常写在 JSON 的 `recentDay`、`startTime`、`endTime`、`from_date`、`to_date` 等字段里，按对应模板使用。

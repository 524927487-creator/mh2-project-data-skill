# SQL 查询配置模板

适用场景：自定义 SQL 分析、查询结果下载为 Excel。

## SQL 内容

请把要执行的 SQL 直接写在下面的 `sql` 代码块中。SQL 中需要保留 ThinkingData 字段的双引号，例如 `"$part_date"`、`"$part_event"`、`"#account_id"`。

如果用户说“今天、昨天、近 7 天、近 30 天、自定义日期范围”等时间范围，请把条件写成工具动态日期占位符：

~~~text
${PartDate:today}                    今天
${PartDate:yesterday}                昨天
${PartDate:last7days}                近 7 天，不含今天
${PartDate:last30days}               近 30 天，不含今天
${PartDate:2026-05-01}               指定某一天
${PartDate:2026-05-01..2026-05-07}   自定义日期范围，包含开始和结束日期
~~~

Skill 执行/生成时会替换成数数 SQL 可用的 `"$part_date"` 条件。生成 SQL 时，用户说“近 7 天”“近 30 天”“最近一个月”等时间范围，默认都写成 `"$part_date"` 分区过滤。

硬性要求：每一个查询 `ta.v_event_10` 的子查询 / CTE 都必须带 `"$part_date"` 限制。如果同一条 SQL 里有多个事件表别名，需要分别加条件，例如 `e."$part_date"`、`login."$part_date"`、`pay."$part_date"`。`"#event_time"` 只用于事件实际发生时间、创角同日校验、精确时间排序等业务限制，不能替代 `"$part_date"`。

生成留存 / LTV SQL 时，未到完整观察期的指标不要输出 0，应输出 `null` / `未到观察期`。生成 `order_finish` 流水或 LTV SQL 时，默认加 `coalesce(is_test, false) = false` 排除测试订单。

~~~sql
select
  "#account_id" as "账号ID"
from ta.v_user_10
limit 100
~~~

## Skill Markdown 配置示例

~~~json
[
  {
    "name": "sql-analysis",
    "enabled": true,
    "action": "sql",
    "mode": "once",
    "sql": "__markdown_sql__"
  }
]
~~~

安装本 skill 后，可让 ChatGPT / Agent 读取本 Markdown 任务并生成或执行对应请求。


## 接口说明

~~~text
Execute SQL query from a UTF-8 SQL file.

Endpoint: POST /querySql?token=$TA_USER_TOKEN


Notes:
  - SQL query requires TA_USER_TOKEN.
  - Reading SQL from a file preserves ThinkingData quoted fields such as "$part_date" and "#account_id".
  - When -o is omitted, output is saved to output/<sql file name>_<date>.xlsx.
~~~

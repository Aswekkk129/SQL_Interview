### 案例：Users By Avg Session Time

#### 表/Column：
- `facebook_web_log`
  - `user_id` (int): 用户ID
  - `timestamp` (datetime): 事件发生的时间戳
  - `action` (varchar): 用户执行的操作（如 `page_load`, `page_exit`）

#### 问题：
计算每个用户的平均会话时间。会话时间定义为 `page_load` 和 `page_exit` 之间的时间差。假设每个用户每天只有一个会话，并且如果某天有多个相同的事件，只考虑最新的 `page_load` 和最早的 `page_exit`。输出 `user_id` 和他们的平均会话时间。

#### 流程，Step by Step：

1. **提取最新的 `page_load` 时间**：
   - 使用 `ROW_NUMBER()` 窗口函数，按 `user_id` 和日期分区，并按 `timestamp` 降序排列，找到每个用户每天最新的 `page_load` 时间。
   - 过滤出每个用户每天最新的 `page_load` 事件。

2. **提取最早的 `page_exit` 时间**：
   - 使用 `ROW_NUMBER()` 窗口函数，按 `user_id` 和日期分区，并按 `timestamp` 升序排列，找到每个用户每天最早的 `page_exit` 时间。
   - 过滤出每个用户每天最早的 `page_exit` 事件。

3. **计算会话时间**：
   - 将最新的 `page_load` 和最早的 `page_exit` 时间进行匹配，计算它们之间的时间差，得到每次会话的持续时间。

4. **计算平均会话时间**：
   - 按 `user_id` 分组，计算每个用户的平均会话时间。

#### SQL 代码：

```sql
WITH latest_page_load AS (
    SELECT 
        user_id, 
        DATE(timestamp) AS date, 
        timestamp AS page_load_time,
        ROW_NUMBER() OVER (PARTITION BY user_id, DATE(timestamp) ORDER BY timestamp DESC) AS rn
    FROM facebook_web_log
    WHERE action = 'page_load'
),
earliest_page_exit AS (
    SELECT 
        user_id, 
        DATE(timestamp) AS date, 
        timestamp AS page_exit_time,
        ROW_NUMBER() OVER (PARTITION BY user_id, DATE(timestamp) ORDER BY timestamp ASC) AS rn
    FROM facebook_web_log
    WHERE action = 'page_exit'
),
session_time AS (
    SELECT 
        lp.user_id, 
        lp.date, 
        ep.page_exit_time - lp.page_load_time AS session_duration
    FROM latest_page_load lp
    JOIN earliest_page_exit ep
        ON lp.user_id = ep.user_id
        AND lp.date = ep.date
        AND lp.rn = 1
        AND ep.rn = 1
)
SELECT 
    user_id, 
    AVG(session_duration) AS avg_session_time
FROM session_time
GROUP BY user_id;
```

#### 考点回顾：
1. **窗口函数**：`ROW_NUMBER()` 用于按用户和日期分区，找到最新或最早的事件。
2. **日期处理**：`DATE(timestamp)` 用于提取日期部分，以便按天分组。
3. **时间差计算**：直接使用 `timestamp` 相减来计算会话时间。
4. **聚合函数**：`AVG()` 用于计算平均会话时间。

#### 重点函数复习：
- `ROW_NUMBER()`: 为每一行分配一个唯一的序号，通常用于分区和排序。
- `DATE()`: 提取日期部分，忽略时间部分。
- `AVG()`: 计算平均值。

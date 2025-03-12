### 案例：7-Day Moving Average Calculation with Window Functions_SQL

#### Example Table:
**customer** 表结构：
- `visited_on` (DATE) - 顾客访问日期
- `amount` (INT) - 消费金额

#### 问题：
计算以 7 天（某日期 + 该日期前的 6 天）为一个时间段的顾客消费总和及移动平均值，且仅当时间段内包含完整的 7 天数据时才输出结果。

---

### 流程步骤说明：

1. **按天聚合消费金额**  
   计算每天的消费总额，作为后续窗口函数的基础：
   ```sql
   SELECT 
       visited_on,
       SUM(amount) AS daily_total
   FROM customer
   GROUP BY visited_on
   ```
   - **目的**：确保每个日期对应一条汇总记录，便于后续按天计算。

2. **使用窗口函数计算移动总和、平均值及窗口内天数**  
   通过窗口函数，定义当前行及其前6行作为窗口范围：
   ```sql
   SUM(daily_total) OVER (
       ORDER BY visited_on 
       ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
   ) AS amount,
   ROUND(AVG(daily_total) OVER (...), 2) AS average_amount,
   COUNT(*) OVER (...) AS day_count
   ```
   - **ROWS BETWEEN 6 PRECEDING AND CURRENT ROW**：窗口包含当前行及前6行，共7条记录。
   - **COUNT(*)**：统计窗口内的记录数，确保正好7天。

3. **过滤出完整7天的窗口**  
   仅保留窗口内恰好有7条记录的日期：
   ```sql
   SELECT 
       visited_on,
       amount,
       average_amount
   FROM moving_avg
   WHERE day_count = 7;
   ```
   - **目的**：排除前6天数据不足的情况，确保结果有效性。

---

### 完整 SQL 代码：
```sql
WITH daily_sales AS (
    -- Step 1: 按天聚合消费金额
    SELECT 
        visited_on,
        SUM(amount) AS daily_total
    FROM customer
    GROUP BY visited_on
), 
moving_avg AS (
    -- Step 2: 计算移动总和、平均值及窗口内天数
    SELECT 
        visited_on,
        SUM(daily_total) OVER (
            ORDER BY visited_on 
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) AS amount,
        ROUND(AVG(daily_total) OVER (
            ORDER BY visited_on 
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ), 2) AS average_amount,
        COUNT(*) OVER (
            ORDER BY visited_on 
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) AS day_count
    FROM daily_sales
)
-- Step 3: 筛选出完整7天的结果
SELECT 
    visited_on,
    amount,
    average_amount
FROM moving_avg
WHERE day_count = 7;
```

---

### 考点回顾：
- **窗口函数（Window Functions）**：掌握 `SUM() OVER` 和 `AVG() OVER` 结合 `ROWS BETWEEN` 的用法，实现滑动窗口计算。
- **数据过滤逻辑**：通过 `COUNT(*) OVER` 统计窗口内行数，确保时间段完整性。
- **执行顺序理解**：明确 CTE（WITH 子句）和窗口函数的执行顺序。

---

### 重点函数复习：
1. **SUM() OVER()**  
   计算指定窗口范围内的总和，支持动态窗口定义（如 `ROWS BETWEEN 6 PRECEDING`）。

2. **AVG() OVER()**  
   类似 `SUM`，计算窗口内平均值，注意保留小数位的处理（`ROUND`）。

3. **COUNT(*) OVER()**  
   统计窗口内记录数，用于过滤非完整时间段。

4. **ROWS BETWEEN**  
   定义窗口的物理行范围，确保精确控制窗口大小（如固定7行）。

---

通过以上步骤和代码，未来复习时可快速理解如何利用窗口函数高效计算移动平均值，避免自连接带来的性能问题。

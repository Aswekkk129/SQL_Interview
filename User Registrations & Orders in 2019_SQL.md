# User Registrations & Orders in 2019_SQL

### 问题描述
我们需要查询每个用户的注册日期和在 2019 年作为买家的订单总数。对于那些在 2019 年没有订单的用户，结果应该显示为 0。

---

### 使用窗口函数的解法

#### 示例表结构
假设表名为 `Orders` 和 `Users`，表结构如下：

**Users 表**
| user_id | join_date  |
|---------|------------|
| 1       | 2018-01-01 |
| 2       | 2019-05-10 |
| 3       | 2020-03-15 |

**Orders 表**
| order_id | buyer_id | order_date  |
|----------|----------|-------------|
| 101      | 1        | 2019-06-01  |
| 102      | 1        | 2019-08-15  |
| 103      | 2        | 2019-09-20  |
| 104      | 4        | 2019-10-10  |

---

#### 通用化的 SQL 查询语句
```sql
SELECT 
    u.user_id AS buyer_id,
    u.join_date,
    COALESCE(SUM(CASE WHEN YEAR(o.order_date) = 2019 THEN 1 ELSE 0 END), 0) AS orders_in_2019
FROM Users u
LEFT JOIN Orders o
ON u.user_id = o.buyer_id
GROUP BY u.user_id, u.join_date;
```

---

### 流程解析

#### Step-by-Step
1. **从用户表开始**：
   • 我们从 `Users` 表中选择所有用户，因为我们需要返回所有用户的注册日期，即使他们在 2019 年没有订单。
   • 使用 `LEFT JOIN` 将 `Orders` 表连接到 `Users` 表，连接条件是 `u.user_id = o.buyer_id`，这样即使某些用户没有订单，他们仍然会出现在结果中。

2. **判断订单是否属于 2019 年**：
   • 使用 `CASE WHEN YEAR(o.order_date) = 2019 THEN 1 ELSE 0 END` 来标记每个订单是否属于 2019 年。
   • 如果订单日期在 2019 年，则返回 1；否则返回 0。

3. **按用户分组并计算订单总数**：
   • 使用 `GROUP BY u.user_id, u.join_date` 按用户分组。
   • 使用 `SUM` 聚合函数计算每个用户在 2019 年的订单总数。

4. **处理没有订单的用户**：
   • 使用 `COALESCE` 函数将 `NULL` 值替换为 0，确保没有订单的用户显示为 0。

---

### 考点回顾
1. **LEFT JOIN 的作用**：
   • 确保左表（`Users`）中的所有记录都被保留，即使右表（`Orders`）中没有匹配的记录。

2. **CASE WHEN 的用法**：
   • 用于条件判断，可以根据不同的条件返回不同的值。

3. **聚合函数的使用**：
   • `SUM` 用于计算总和，`COUNT` 用于计算数量。

4. **COALESCE 的作用**：
   • 将 `NULL` 值替换为指定的默认值（如 0）。

5. **GROUP BY 的作用**：
   • 按指定的列分组，通常与聚合函数一起使用。

---

### 重点函数复习
1. **CASE WHEN**：
   • 语法：`CASE WHEN condition THEN result ELSE other_result END`
   • 用于根据条件返回不同的值。

2. **COALESCE**：
   • 语法：`COALESCE(value1, value2, ..., valueN)`
   • 返回第一个非 `NULL` 的值。

3. **YEAR**：
   • 语法：`YEAR(date_column)`
   • 提取日期中的年份部分。

4. **LEFT JOIN**：
   • 返回左表中的所有记录，以及右表中匹配的记录。如果右表中没有匹配的记录，则结果为 `NULL`。

5. **GROUP BY**：
   • 按指定的列分组，通常与聚合函数（如 `SUM`、`COUNT`）一起使用。

---

### 总结
通过使用 `LEFT JOIN` 和 `CASE WHEN`，我们可以确保查询结果中包含所有用户，并正确计算每个用户在 2019 年的订单总数。对于没有订单的用户，使用 `COALESCE` 将结果替换为 0。这种方法比直接使用 `WHERE` 过滤更全面，适合处理类似的统计问题。
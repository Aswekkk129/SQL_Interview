你的SQL查询整体上是正确的，但在一些细节上可以进一步优化。以下是针对每个问题的优化建议：

### Q1: CID, total spend in USD

你的查询已经基本正确，但可以做一些小的改进，比如使用`SUM`函数时，确保所有字段名和表名都清晰且一致。此外，可以使用`COALESCE`来处理可能的`NULL`值。

```sql
-- Q1: CID, total spend in USD
SELECT 
    S.c_id AS CID, 
    SUM(S.s_amount * E.e_rate) AS total_spend_usd
FROM 
    Spend S
JOIN 
    ExchangeRate E 
ON 
    S.s_currency = E.e_currency
GROUP BY 
    S.c_id;
```

### Q2: AID, number of days from first spend date to highest spend date

你的查询逻辑是正确的，但可以通过简化子查询和使用`DATEDIFF`函数来优化。此外，确保字段名和表名的一致性。

```sql
-- Q2: AID, number of days from first spend date to highest spend date
WITH SummaryTable AS (
  SELECT 
      C.a_id, 
      S.s_date, 
      SUM(S.s_amount * E.e_rate) AS t_amount
  FROM 
      Campaigns C 
  JOIN 
      Spend S 
  ON 
      C.c_id = S.c_id
  JOIN 
      ExchangeRate E 
  ON 
      S.s_currency = E.e_currency
  GROUP BY 
      C.a_id, S.s_date
), 

FirstSpend AS (
  SELECT 
      a_id, 
      MIN(s_date) AS first_spend_date
  FROM 
      SummaryTable
  GROUP BY 
      a_id
), 

HighestSpend AS (
  SELECT 
      a_id, 
      s_date AS highest_spend_date
  FROM (
    SELECT 
        a_id, 
        s_date,
        ROW_NUMBER() OVER(PARTITION BY a_id ORDER BY t_amount DESC) AS rk
    FROM 
        SummaryTable
  ) T
  WHERE 
      rk = 1
)

SELECT 
    F.a_id, 
    DATEDIFF(H.highest_spend_date, F.first_spend_date) AS days_diff
FROM 
    FirstSpend F 
JOIN 
    HighestSpend H 
ON 
    F.a_id = H.a_id;
```

### 创建表和插入数据

你的表创建和数据插入部分是正确的，但可以稍微优化一下，比如使用`DECIMAL`时指定精度和小数位数。

```sql
CREATE TABLE Campaigns (
    a_id INT,
    c_id INT
);

CREATE TABLE Spend (
    c_id INT,
    s_date DATE,
    s_amount DECIMAL(10, 2),
    s_currency TEXT
);

CREATE TABLE ExchangeRate (
    e_currency TEXT,
    e_rate DECIMAL(10, 2)
);

INSERT INTO Campaigns
VALUES 
    (1, 123), 
    (1, 234), 
    (2, 235);

INSERT INTO Spend
VALUES 
    (123, '2017-08-01', 200, 'USD'),
    (123, '2017-08-02', 150, 'USD'),
    (234, '2017-09-01', 500, 'USD'),
    (235, '2017-07-01', 100, 'CAD');

INSERT INTO ExchangeRate
VALUES 
    ('CAD', 0.79), 
    ('USD', 1.00);
```

### 总结

- **Q1**：你的查询已经基本正确，只需确保字段名和表名的一致性。
- **Q2**：逻辑正确，但可以通过简化子查询和使用`DATEDIFF`函数来优化。
- **表创建和数据插入**：可以指定`DECIMAL`的精度和小数位数。

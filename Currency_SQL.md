To solve this problem using window functions, you can follow these steps:

1. **Filter the data for the relevant dates**: We need to get the exchange rates for January 1, 2020, and July 1, 2020.
2. **Use a window function to calculate the difference**: We can use the `LAG` or `LEAD` function to get the exchange rate on January 1, 2020, and then calculate the difference with the exchange rate on July 1, 2020.

Here’s how you can write the SQL query:

```sql
WITH exchange_rates AS (
    SELECT
        source_currency,
        exchange_rate,
        date,
        -- Use LAG to get the exchange rate on January 1, 2020
        LAG(exchange_rate) OVER (PARTITION BY source_currency ORDER BY date) AS jan_rate
    FROM
        sf_exchange_rate
    WHERE
        date IN ('2020-01-01', '2020-07-01')
)
SELECT
    source_currency AS currency_code,
    exchange_rate - jan_rate AS rate_difference
FROM
    exchange_rates
WHERE
    date = '2020-07-01';
```

### Explanation:

1. **Filtering the Data**:
   - The `WHERE` clause filters the data to only include exchange rates on January 1, 2020, and July 1, 2020.

2. **Window Function**:
   - The `LAG` function is used to get the exchange rate on January 1, 2020, for each currency. The `PARTITION BY source_currency` ensures that the calculation is done separately for each currency, and `ORDER BY date` ensures that the previous value (January 1, 2020) is correctly aligned with the current value (July 1, 2020).

3. **Calculating the Difference**:
   - In the final `SELECT` statement, we calculate the difference between the exchange rate on July 1, 2020, and the exchange rate on January 1, 2020.

4. **Final Filter**:
   - The final `WHERE` clause ensures that we only output the results for July 1, 2020, which contains the difference we are interested in.

### Output:
The output will be a list of currency codes and the difference in their exchange rates between July 1, 2020, and January 1, 2020.

### Example Output:
| currency_code | rate_difference |
|---------------|-----------------|
| EUR           | 0.05            |
| GBP           | -0.02           |
| JPY           | 0.10            |

This output shows how the exchange rate of each currency changed between January 1, 2020, and July 1, 2020.

# SQL_Interview

This repository is primarily my collection of challenging SQL interview questions. I hope that each case not only helps you learn various SQL functions and CTEs, but also guides you through a practical problem-solving process using actual SQL queries, ultimately serving as a valuable resource for enhancing your SQL knowledge and interview preparation.
---

# SQL Query Analysis and Rewrite

This document contains the analysis, rewrite, and explanation of two SQL queries. The goal is to improve readability and maintainability by using Common Table Expressions (CTEs). Each query is broken down step-by-step for clarity.

---

## **Query 1: Check if the Second Item Sold by a User is Their Favorite Brand**

### **Problem Statement**
We want to determine whether the second item sold by each user (as a seller) matches their favorite brand (`fav_brand`). This involves:
1. Identifying the second item sold by each user.
2. Comparing the brand of that item with the user's favorite brand.

### **Rewritten SQL Code**
```sql
WITH RankedOrders AS (
    -- Step 1: Get all items sold by each user and rank them by order time
    SELECT 
        u.user_id AS seller_id,
        u.fav_brand,
        o.item_id,
        o.order_at,
        RANK() OVER (PARTITION BY u.user_id ORDER BY o.order_at) AS rank_order
    FROM tb_users u
    LEFT JOIN tb_orders o
    ON u.user_id = o.seller_id
),
SecondItemDetails AS (
    -- Step 2: Filter for the second item sold by each user
    SELECT 
        ro.seller_id,
        ro.fav_brand,
        ro.item_id
    FROM RankedOrders ro
    WHERE ro.rank_order = 2
),
ItemBrandInfo AS (
    -- Step 3: Get the brand information for the second item
    SELECT 
        sid.seller_id,
        sid.fav_brand,
        i.item_brand
    FROM SecondItemDetails sid
    LEFT JOIN tb_items i
    ON sid.item_id = i.item_id
)
-- Step 4: Compare the second item's brand with the user's favorite brand
SELECT 
    ibi.seller_id AS user_id,
    CASE 
        WHEN ibi.fav_brand = ibi.item_brand THEN 'yes'
        ELSE 'no'
    END AS is_2nd_item_fav_brand
FROM ItemBrandInfo ibi;
```

### **Step-by-Step Explanation**
1. **RankedOrders CTE**:
   - Joins `tb_users` and `tb_orders` to get all items sold by each user.
   - Uses the `RANK()` function to assign a ranking to each order based on the `order_at` timestamp.

2. **SecondItemDetails CTE**:
   - Filters the results to include only the second item sold by each user (`rank_order = 2`).

3. **ItemBrandInfo CTE**:
   - Joins the second item details with `tb_items` to retrieve the brand of the second item.

4. **Final Query**:
   - Compares the user's `fav_brand` with the brand of the second item.
   - Outputs `'yes'` if they match, otherwise `'no'`.

---

## **Query 2: List All Users with Their 2018 Purchase and Sales Amounts**

### **Problem Statement**
As a marketing manager, you need a list of all users with the following information:
1. User ID (`user_id`).
2. Join date (`joined_at`).
3. Total purchase amount in 2018.
4. Total sales amount in 2018.

### **Rewritten SQL Code**
```sql
WITH UserPurchases AS (
    -- Step 1: Calculate the total purchase amount for each user in 2018
    SELECT 
        u.user_id,
        DATE(u.joined_at) AS joined_date,
        SUM(CASE WHEN YEAR(o.order_at) = 2018 THEN o.order_amount ELSE 0 END) AS purchase_amount
    FROM tb_users u
    LEFT JOIN tb_orders o
    ON u.user_id = o.buyer_id
    GROUP BY u.user_id, DATE(u.joined_at)
),
UserSales AS (
    -- Step 2: Calculate the total sales amount for each user in 2018
    SELECT 
        u.user_id,
        SUM(CASE WHEN YEAR(o.order_at) = 2018 THEN o.order_amount ELSE 0 END) AS sales_amount
    FROM tb_users u
    LEFT JOIN tb_orders o
    ON u.user_id = o.seller_id
    GROUP BY u.user_id
)
-- Step 3: Combine purchase and sales amounts
SELECT 
    up.user_id,
    up.joined_date,
    up.purchase_amount,
    us.sales_amount
FROM UserPurchases up
LEFT JOIN UserSales us
ON up.user_id = us.user_id;
```

### **Step-by-Step Explanation**
1. **UserPurchases CTE**:
   - Joins `tb_users` and `tb_orders` to get all orders where the user is the buyer.
   - Uses a `CASE` statement to calculate the total purchase amount for 2018.
   - Groups the results by `user_id` and `joined_at`.

2. **UserSales CTE**:
   - Similar to `UserPurchases`, but focuses on orders where the user is the seller.
   - Calculates the total sales amount for 2018.
   - Groups the results by `user_id`.

3. **Final Query**:
   - Combines the purchase and sales amounts by performing a `LEFT JOIN` between `UserPurchases` and `UserSales`.
   - Outputs the final result with all required fields.

---

## **Conclusion**
- Both queries have been rewritten using CTEs to improve readability and maintainability.
- The step-by-step breakdown ensures that the logic is easy to follow and debug.
- Using CTEs makes the queries modular, allowing for easier modifications or extensions in the future.

---

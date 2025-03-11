# Platform_SQL

以下是对两个查询的重写和逐步解释。我们将使用CTE（Common Table Expressions）来提高可读性和逻辑清晰度，并对每一步进行详细说明。

---

### **查询1：判断用户卖出的第二个商品是否是他们的喜爱品牌**

#### **需求分析**
我们需要检查每个用户（作为卖家）卖出的第二个商品是否是他们最喜欢的品牌。为此，我们需要：
1. 从 `tb_users` 和 `tb_orders` 中提取用户的 `user_id`、`fav_brand` 和订单信息。
2. 使用窗口函数 `RANK()` 对每个用户的订单按时间排序，找到每个用户卖出的第二个商品。
3. 将订单的商品信息与 `tb_items` 表连接，获取商品的品牌信息。
4. 判断第二个商品的品牌是否与用户的 `fav_brand` 匹配。

#### **重写后的SQL代码**
```sql
WITH RankedOrders AS (
    -- Step 1: 获取每个用户卖出的所有商品，并按时间排序
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
    -- Step 2: 筛选出每个用户卖出的第二个商品
    SELECT 
        ro.seller_id,
        ro.fav_brand,
        ro.item_id
    FROM RankedOrders ro
    WHERE ro.rank_order = 2
),
ItemBrandInfo AS (
    -- Step 3: 获取第二个商品的品牌信息
    SELECT 
        sid.seller_id,
        sid.fav_brand,
        i.item_brand
    FROM SecondItemDetails sid
    LEFT JOIN tb_items i
    ON sid.item_id = i.item_id
)
-- Step 4: 判断第二个商品的品牌是否是用户的喜爱品牌
SELECT 
    ibi.seller_id AS user_id,
    CASE 
        WHEN ibi.fav_brand = ibi.item_brand THEN 'yes'
        ELSE 'no'
    END AS is_2nd_item_fav_brand
FROM ItemBrandInfo ibi;
```

#### **逐步解释**
1. **RankedOrders CTE**:
   - 通过 `LEFT JOIN` 将 `tb_users` 和 `tb_orders` 连接，获取每个用户卖出的所有商品。
   - 使用 `RANK()` 函数按 `order_at` 时间排序，为每个用户的订单分配排名。

2. **SecondItemDetails CTE**:
   - 筛选出排名为 2 的订单（即每个用户卖出的第二个商品）。
   - 只保留 `seller_id`、`fav_brand` 和 `item_id`。

3. **ItemBrandInfo CTE**:
   - 将 `SecondItemDetails` 与 `tb_items` 连接，获取第二个商品的品牌信息。

4. **最终查询**:
   - 比较用户的 `fav_brand` 和第二个商品的品牌，输出结果为 `'yes'` 或 `'no'`。

---

### **查询2：列出所有用户及其2018年的购买金额和销售金额**

#### **需求分析**
我们需要生成一个列表，包含以下信息：
1. 用户ID (`user_id`)。
2. 用户加入日期 (`joined_at`)。
3. 用户在2018年的购买金额总和。
4. 用户在2018年的销售金额总和。

#### **重写后的SQL代码**
```sql
WITH UserPurchases AS (
    -- Step 1: 计算每个用户在2018年的购买金额
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
    -- Step 2: 计算每个用户在2018年的销售金额
    SELECT 
        u.user_id,
        SUM(CASE WHEN YEAR(o.order_at) = 2018 THEN o.order_amount ELSE 0 END) AS sales_amount
    FROM tb_users u
    LEFT JOIN tb_orders o
    ON u.user_id = o.seller_id
    GROUP BY u.user_id
)
-- Step 3: 合并购买金额和销售金额
SELECT 
    up.user_id,
    up.joined_date,
    up.purchase_amount,
    us.sales_amount
FROM UserPurchases up
LEFT JOIN UserSales us
ON up.user_id = us.user_id;
```

#### **逐步解释**
1. **UserPurchases CTE**:
   - 通过 `LEFT JOIN` 将 `tb_users` 和 `tb_orders` 连接，筛选出用户作为买家的订单。
   - 使用 `CASE` 语句计算2018年的购买金额总和。
   - 按 `user_id` 和 `joined_at` 分组。

2. **UserSales CTE**:
   - 类似于 `UserPurchases`，但筛选的是用户作为卖家的订单。
   - 计算2018年的销售金额总和。
   - 按 `user_id` 分组。

3. **最终查询**:
   - 将 `UserPurchases` 和 `UserSales` 通过 `LEFT JOIN` 合并，生成最终结果。

---

### **总结**
- 查询1通过CTE分步实现了对用户卖出的第二个商品的品牌判断，逻辑清晰且易于维护。
- 查询2通过两个独立的CTE分别计算购买金额和销售金额，最后合并结果，避免了重复计算和复杂嵌套。
- 使用CTE不仅提高了代码的可读性，还便于调试和扩展。
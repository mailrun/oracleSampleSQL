## 产品分析
### 查询销售量最高的前10个产品
```sql
SELECT p.prod_name, SUM(s.quantity_sold) AS total_quantity
FROM sh.sales s
JOIN sh.products p ON s.prod_id = p.prod_id
GROUP BY p.prod_name
ORDER BY total_quantity DESC
FETCH FIRST 10 ROWS ONLY;
```

## 客户分析
### 查询每个客户的总消费金额
```sql
SELECT c.cust_last_name, c.cust_first_name, SUM(s.amount_sold) AS total_spent
FROM sh.customers c
JOIN sh.sales s ON c.cust_id = s.cust_id
GROUP BY c.cust_last_name, c.cust_first_name
ORDER BY total_spent DESC;
```
### 查询客户的地理分布
```sql
SELECT c.country_id, COUNT(*) AS num_customers
FROM sh.customers c
GROUP BY c.country_id
ORDER BY num_customers DESC;
```

## 渠道分析
### 查询每个渠道的平均订单金额
```sql
SELECT ch.channel_desc, AVG(s.amount_sold) AS avg_order_amount
FROM sh.sales s
JOIN sh.channels ch ON s.channel_id = ch.channel_id
GROUP BY ch.channel_desc
ORDER BY avg_order_amount DESC;
```
### 查询每个渠道的订单数量
```sql
SELECT ch.channel_desc, COUNT(s.amount_sold) AS num_orders
FROM sh.sales s
JOIN sh.channels ch ON s.channel_id = ch.channel_id
GROUP BY ch.channel_desc
ORDER BY num_orders DESC;
```

## 促销分析
### 查询每个促销活动的总销售额
```sql
SELECT p.promo_name, SUM(s.amount_sold) AS total_sales
FROM sh.sales s
JOIN sh.promotions p ON s.promo_id = p.promo_id
GROUP BY p.promo_name
ORDER BY total_sales DESC;
```
### 查询参与特定促销活动(TV program sponsorship)的产品销售情况
```sql
SELECT p.prod_name, SUM(s.amount_sold) AS total_sales
FROM sh.sales s
JOIN sh.products p ON s.prod_id = p.prod_id
WHERE s.promo_id IN (SELECT promo_id FROM sh.promotions WHERE promo_subcategory = 'TV program sponsorship')
GROUP BY p.prod_name
ORDER BY total_sales DESC;
```

## 时间分析
### 查询每个月的销售额
```sql
SELECT t.calendar_month_desc, SUM(s.amount_sold) AS total_sales
FROM sh.sales s
JOIN sh.times t ON s.time_id = t.time_id
GROUP BY t.calendar_month_desc
ORDER BY t.calendar_month_desc;
```

## 同比（年同比增长）查询
### 查询每年的销售额及其同比增长率
```sql
SELECT t.calendar_year,
       SUM(s.amount_sold) AS total_sales,
       LAG(SUM(s.amount_sold)) OVER (ORDER BY t.calendar_year) AS prev_year_sales,
       ROUND((SUM(s.amount_sold) - LAG(SUM(s.amount_sold)) OVER (ORDER BY t.calendar_year)) / LAG(SUM(s.amount_sold)) OVER (ORDER BY t.calendar_year) * 100, 2) AS yoy_growth_rate
FROM sh.sales s
JOIN sh.times t ON s.time_id = t.time_id
GROUP BY t.calendar_year
ORDER BY t.calendar_year;
```

## 环比（季度环比增长）查询
### 查询每季度的销售额及其环比增长率
```sql
SELECT t.calendar_quarter_desc,
       SUM(s.amount_sold) AS total_sales,
       LAG(SUM(s.amount_sold)) OVER (ORDER BY t.calendar_quarter_desc) AS prev_quarter_sales,
       ROUND((SUM(s.amount_sold) - LAG(SUM(s.amount_sold)) OVER (ORDER BY t.calendar_quarter_desc)) / LAG(SUM(s.amount_sold)) OVER (ORDER BY t.calendar_quarter_desc) * 100, 2) AS qoq_growth_rate
FROM sh.sales s
JOIN sh.times t ON s.time_id = t.time_id
GROUP BY t.calendar_quarter_desc
ORDER BY t.calendar_quarter_desc;
```

### 查询每个月的销售额及其环比增长率
```sql
SELECT t.calendar_month_desc,
       SUM(s.amount_sold) AS total_sales,
       LAG(SUM(s.amount_sold)) OVER (ORDER BY t.calendar_month_desc) AS prev_month_sales,
       ROUND((SUM(s.amount_sold) - LAG(SUM(s.amount_sold)) OVER (ORDER BY t.calendar_month_desc)) / LAG(SUM(s.amount_sold)) OVER (ORDER BY t.calendar_month_desc) * 100, 2) AS mom_growth_rate
FROM sh.sales s
JOIN sh.times t ON s.time_id = t.time_id
GROUP BY t.calendar_month_desc
ORDER BY t.calendar_month_desc;
```

## 综合示例
### 查询价格、促销、渠道对销售额的影响
```sql
SELECT p.prod_name, ch.channel_desc, pr.promo_name, SUM(s.amount_sold) AS total_sales
FROM sh.sales s
JOIN sh.products p ON s.prod_id = p.prod_id
JOIN sh.channels ch ON s.channel_id = ch.channel_id
JOIN sh.promotions pr ON s.promo_id = pr.promo_id
GROUP BY p.prod_name, ch.channel_desc, pr.promo_name
ORDER BY total_sales DESC;
```

### 查询销售额最高的前10个产品及其同比增长
```sql
WITH product_sales AS (
  SELECT p.prod_name, t.calendar_year, SUM(s.amount_sold) AS total_sales
  FROM sh.sales s
  JOIN sh.products p ON s.prod_id = p.prod_id
  JOIN sh.times t ON s.time_id = t.time_id
  GROUP BY p.prod_name, t.calendar_year
)
SELECT prod_name, calendar_year, total_sales,
       LAG(total_sales) OVER (PARTITION BY prod_name ORDER BY calendar_year) AS prev_year_sales,
       ROUND((total_sales - LAG(total_sales) OVER (PARTITION BY prod_name ORDER BY calendar_year)) / LAG(total_sales) OVER (PARTITION BY prod_name ORDER BY calendar_year) * 100, 2) AS yoy_growth_rate
FROM product_sales
WHERE prod_name IN (
  SELECT prod_name
  FROM product_sales
  WHERE calendar_year = (SELECT MAX(calendar_year) FROM product_sales)
  ORDER BY total_sales DESC
  FETCH FIRST 10 ROWS ONLY
)
ORDER BY prod_name, calendar_year;
```

### 查询每个渠道的季度销售额及其环比增长
```sql
SELECT ch.channel_desc, t.calendar_quarter_desc,
       SUM(s.amount_sold) AS total_sales,
       LAG(SUM(s.amount_sold)) OVER (PARTITION BY ch.channel_desc ORDER BY t.calendar_quarter_desc) AS prev_quarter_sales,
       ROUND((SUM(s.amount_sold) - LAG(SUM(s.amount_sold)) OVER (PARTITION BY ch.channel_desc ORDER BY t.calendar_quarter_desc)) / LAG(SUM(s.amount_sold)) OVER (PARTITION BY ch.channel_desc ORDER BY t.calendar_quarter_desc) * 100, 2) AS qoq_growth_rate
FROM sh.sales s
JOIN sh.channels ch ON s.channel_id = ch.channel_id
JOIN sh.times t ON s.time_id = t.time_id
GROUP BY ch.channel_desc, t.calendar_quarter_desc
ORDER BY ch.channel_desc, t.calendar_quarter_desc;
```
```

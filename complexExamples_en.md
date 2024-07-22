## product analysis
### Query the top 10 products with the highest sales volume
```sql
SELECT p.prod_name, SUM(s.quantity_sold) AS total_quantity
FROM sh.sales s
JOIN sh.products p ON s.prod_id = p.prod_id
GROUP BY p.prod_name
ORDER BY total_quantity DESC
FETCH FIRST 10 ROWS ONLY;
```

## Customer Analysis
### Query the total consumption amount of each customer
```sql
SELECT c.cust_last_name, c.cust_first_name, SUM(s.amount_sold) AS total_spent
FROM sh.customers c
JOIN sh.sales s ON c.cust_id = s.cust_id
GROUP BY c.cust_last_name, c.cust_first_name
ORDER BY total_spent DESC;
```
### Query the geographical distribution of customers
```sql
SELECT c.country_id, COUNT(*) AS num_customers
FROM sh.customers c
GROUP BY c.country_id
ORDER BY num_customers DESC;
```

## Channel Analysis
### Query the average order amount for each channel
```sql
SELECT ch.channel_desc, AVG(s.amount_sold) AS avg_order_amount
FROM sh.sales s
JOIN sh.channels ch ON s.channel_id = ch.channel_id
GROUP BY ch.channel_desc
ORDER BY avg_order_amount DESC;
```
### Query the order quantity of each channel
```sql
SELECT ch.channel_desc, COUNT(s.amount_sold) AS num_orders
FROM sh.sales s
JOIN sh.channels ch ON s.channel_id = ch.channel_id
GROUP BY ch.channel_desc
ORDER BY num_orders DESC;
```

## Promotion Analysis
### Query the total sales for each promotion
```sql
SELECT p.promo_name, SUM(s.amount_sold) AS total_sales
FROM sh.sales s
JOIN sh.promotions p ON s.promo_id = p.promo_id
GROUP BY p.promo_name
ORDER BY total_sales DESC;
```
### Check the sales of products participating in a specific promotion (TV program sponsorship)
```sql
SELECT p.prod_name, SUM(s.amount_sold) AS total_sales
FROM sh.sales s
JOIN sh.products p ON s.prod_id = p.prod_id
WHERE s.promo_id IN (SELECT promo_id FROM sh.promotions WHERE promo_subcategory = 'TV program sponsorship')
GROUP BY p.prod_name
ORDER BY total_sales DESC;
```

## Time analysis
### Query the sales volume for each month
```sql
SELECT t.calendar_month_desc, SUM(s.amount_sold) AS total_sales
FROM sh.sales s
JOIN sh.times t ON s.time_id = t.time_id
GROUP BY t.calendar_month_desc
ORDER BY t.calendar_month_desc;
```

## Year-over-year (YoY) growth query
### Query annual sales and its year-on-year growth rate
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

## Month-on-month (quarter-on-quarter growth) query
### Query the sales volume and month-on-month growth rate for each quarter
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

### Query the sales volume and month-on-month growth rate of each month
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

## Comprehensive Example
### Check the impact of price, promotion and channel on sales
```sql
SELECT p.prod_name, ch.channel_desc, pr.promo_name, SUM(s.amount_sold) AS total_sales
FROM sh.sales s
JOIN sh.products p ON s.prod_id = p.prod_id
JOIN sh.channels ch ON s.channel_id = ch.channel_id
JOIN sh.promotions pr ON s.promo_id = pr.promo_id
GROUP BY p.prod_name, ch.channel_desc, pr.promo_name
ORDER BY total_sales DESC;
```

### Query the top 10 products with the highest sales and their year-on-year growth
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

### Query the quarterly sales and month-on-month growth of each channel
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

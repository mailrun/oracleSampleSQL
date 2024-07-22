# Simple SQL Example

## SUM

### 查询所有销售的总金额
```sql
SELECT SUM(amount_sold) AS total_sales FROM sh.sales;
```

### Query the total amount of all sales
```sql
SELECT prod_id, SUM(amount_sold) AS total_sales FROM sh.sales GROUP BY prod_id;
```

## AVG

### Query the average amount of all sales
```sql
SELECT AVG(amount_sold) AS average_sales FROM sh.sales;
```

### Query the average sales amount of each product
```sql
SELECT prod_id, AVG(amount_sold) AS average_sales FROM sh.sales GROUP BY prod_id;
```

## MEDIAN

### Query the median amount of all sales
```sql
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY amount_sold) AS median_sales FROM sh.sales;
```

### Query the median sales amount of each product
```sql
SELECT prod_id, PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY amount_sold) AS median_sales FROM sh.sales GROUP BY prod_id;
```

## MAX

### Query the maximum amount of all sales
```sql
SELECT MAX(amount_sold) AS max_sales FROM sh.sales;
```

### Query the maximum amount of all product
```sql
SELECT prod_id, MAX(amount_sold) AS max_sales FROM sh.sales GROUP BY prod_id;
```

## MIN

### Query the minimum amount of all sales
```sql
SELECT MIN(amount_sold) AS min_sales FROM sh.sales;
```

### Query the minimum sales amount for each product
```sql
SELECT prod_id, MIN(amount_sold) AS min_sales FROM sh.sales GROUP BY prod_id;
```

## COUNT

### Query the total number of sales records
```sql
SELECT COUNT(*) AS total_sales_count FROM sh.sales;
```

### Query the sales record quantity of each product
```sql
SELECT prod_id, COUNT(*) AS sales_count FROM sh.sales GROUP BY prod_id;
```

## Comprehensive Example

### Query the total sales amount, average sales amount, maximum sales amount, minimum sales amount and number of sales records for each product
```sql
SELECT prod_id, 
       SUM(amount_sold) AS total_sales, 
       AVG(amount_sold) AS average_sales, 
       MAX(amount_sold) AS max_sales, 
       MIN(amount_sold) AS min_sales, 
       COUNT(*) AS sales_count 
FROM sh.sales 
GROUP BY prod_id;
```
```

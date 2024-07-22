# 简单SQL示例

## 求和 (SUM)

### 查询所有销售的总金额
```sql
SELECT SUM(amount_sold) AS total_sales FROM sh.sales;
```

### 查询每个产品的总销售金额
```sql
SELECT prod_id, SUM(amount_sold) AS total_sales FROM sh.sales GROUP BY prod_id;
```

## 求平均值 (AVG)

### 查询所有销售的平均金额
```sql
SELECT AVG(amount_sold) AS average_sales FROM sh.sales;
```

### 查询每个产品的平均销售金额
```sql
SELECT prod_id, AVG(amount_sold) AS average_sales FROM sh.sales GROUP BY prod_id;
```

## 求中位数 (MEDIAN)

### 查询所有销售的中位数金额
```sql
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY amount_sold) AS median_sales FROM sh.sales;
```

### 查询每个产品的中位数销售金额
```sql
SELECT prod_id, PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY amount_sold) AS median_sales FROM sh.sales GROUP BY prod_id;
```

## 求最大值 (MAX)

### 查询所有销售的最大金额
```sql
SELECT MAX(amount_sold) AS max_sales FROM sh.sales;
```

### 查询每个产品的最大销售金额
```sql
SELECT prod_id, MAX(amount_sold) AS max_sales FROM sh.sales GROUP BY prod_id;
```

## 求最小值 (MIN)

### 查询所有销售的最小金额
```sql
SELECT MIN(amount_sold) AS min_sales FROM sh.sales;
```

### 查询每个产品的最小销售金额
```sql
SELECT prod_id, MIN(amount_sold) AS min_sales FROM sh.sales GROUP BY prod_id;
```

## 求数据总量 (COUNT)

### 查询销售记录的总数量
```sql
SELECT COUNT(*) AS total_sales_count FROM sh.sales;
```

### 查询每个产品的销售记录数量
```sql
SELECT prod_id, COUNT(*) AS sales_count FROM sh.sales GROUP BY prod_id;
```

## 综合示例

### 查询每个产品的总销售金额、平均销售金额、最大销售金额、最小销售金额和销售记录数量
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

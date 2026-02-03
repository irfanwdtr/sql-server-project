## Description
This project analyzes BikeStores sales data using SQL Server (T-SQL).
The output queries are designed to be directly integrated into Power BI.

# BikeStores SQL Project

## Table of Contents
- [1. check_schema](#1-check_schema)
- [2. sales_base_query](#2-sales_base_query)
- [3. sales_summary](#3-sales_summary)
- [4. product_performance](#4-product_performance)
- [5. dashboard](#5-dashboard)


## 1. check_schema
Tujuan: menampilkan daftar schema + nama tabel yang ada di BikeStores.
```sql
USE BikeStores;
GO

SELECT 
    s.name AS schema_name,
    t.name AS table_name
FROM sys.tables t
JOIN sys.schemas s ON t.schema_id = s.schema_id
ORDER BY s.name, t.name;
```
**output :**
![Output](images/image1.png)


## 2. sales_base_query
tujuan :
Membuat dataset utama (single source of truth) untuk analisis

Menggabungkan:
1. order
2. customer
3. store
4. product
5. category
6. brand
Menghitung nilai penjualan bersih

```sql
USE BikeStores;
GO

SELECT
    o.order_id,
    o.order_date,
    YEAR(o.order_date) AS order_year,
    MONTH(o.order_date) AS order_month,

    c.customer_id,
    c.first_name + ' ' + c.last_name AS customer_name,

    st.store_name,

    p.product_id,
    p.product_name,
    cgy.category_name,
    b.brand_name,

    oi.quantity,
    oi.list_price,
    oi.discount,

    (oi.quantity * oi.list_price) * (1 - oi.discount) AS total_sales
FROM sales.orders o
JOIN sales.order_items oi 
    ON o.order_id = oi.order_id
JOIN sales.customers c 
    ON o.customer_id = c.customer_id
JOIN sales.stores st 
    ON o.store_id = st.store_id
JOIN production.products p 
    ON oi.product_id = p.product_id
JOIN production.categories cgy 
    ON p.category_id = cgy.category_id
JOIN production.brands b 
    ON p.brand_id = b.brand_id;

```
**output:**
![Output](images/image2.png)

## 3. sales_summary
Mengubah data detail menjadi ringkasan waktu

Menjawab pertanyaan bisnis:
1. tren penjualan per bulan
2. performa tiap store

```sql
USE BikeStores;
GO

SELECT
    YEAR(o.order_date) AS order_year,
    MONTH(o.order_date) AS order_month,
    st.store_name,
    SUM((oi.quantity * oi.list_price) * (1 - oi.discount)) AS total_sales
FROM sales.orders o
JOIN sales.order_items oi ON o.order_id = oi.order_id
JOIN sales.stores st ON o.store_id = st.store_id
GROUP BY
    YEAR(o.order_date),
    MONTH(o.order_date),
    st.store_name
ORDER BY order_year, order_month;
```
**output**
![Output](images/image3.png)

## 4. product_performance
Menilai produk mana yang paling berkontribusi

```sql
USE BikeStores;
GO

SELECT
    p.product_name,
    cgy.category_name,
    SUM(oi.quantity) AS total_qty_sold,
    SUM((oi.quantity * oi.list_price) * (1 - oi.discount)) AS total_revenue
FROM sales.order_items oi
JOIN production.products p ON oi.product_id = p.product_id
JOIN production.categories cgy ON p.category_id = cgy.category_id
GROUP BY
    p.product_name,
    cgy.category_name
ORDER BY total_revenue DESC;
```
**output :**
![Output](images/image4.png)

## 5. Dashboard
![Output](images/dashboard.png)


## Snowflake Schema

```sql
CREATE DATABASE sales_db_snowflake;
USE sales_db_snowflake;

-- Time Dimension (Snowflake normalized)
CREATE TABLE Dim_Date (
    date_id INT PRIMARY KEY,
    date DATE,
    year INT,
    month INT,
    day INT,
    day_name VARCHAR(20)
);

-- CREATE TABLE Dim_Quarter (
--     quarter_id INT PRIMARY KEY,
--     quarter INT
-- );

INSERT INTO Dim_Date (date_id, date, year, month, day, day_name)
VALUES
  (1, '2023-01-01', 2023, 1, 1, 'Sunday'),
  (2, '2023-02-15', 2023, 2, 15, 'Wednesday');

-- INSERT INTO Dim_Quarter (quarter_id, quarter)
-- VALUES
--   (1, 1),
--   (2, 2);

-- Product Dimension (Snowflake normalized)
CREATE TABLE Dim_Product (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category_id INT
);

CREATE TABLE Dim_Category (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(50)
);

INSERT INTO Dim_Category (category_id, category_name)
VALUES
  (1, 'Electronics'),
  (2, 'Clothing');

INSERT INTO Dim_Product (product_id, product_name, category_id)
VALUES
  (1, 'Laptop', 1),
  (2, 'Smartphone', 1),
  (3, 'T-Shirt', 2);

-- Customer Dimension (Snowflake normalized)
CREATE TABLE Dim_Customer (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    geography_id INT
);

CREATE TABLE Dim_Geography (
    geography_id INT PRIMARY KEY,
    city VARCHAR(50),
    country VARCHAR(50)
);

INSERT INTO Dim_Geography (geography_id, city, country)
VALUES
  (1, 'New York', 'USA'),
  (2, 'London', 'UK');

INSERT INTO Dim_Customer (customer_id, customer_name, geography_id)
VALUES
  (1, 'Alice Johnson', 1),
  (2, 'Bob Smith', 2);

-- Fact Table
CREATE TABLE Fact_Sales (
    sale_id INT PRIMARY KEY,
    date_id INT,
    product_id INT,
    customer_id INT,
    quantity INT,
    amount DECIMAL(10,2),
    FOREIGN KEY (date_id) REFERENCES Dim_Date(date_id),
    FOREIGN KEY (product_id) REFERENCES Dim_Product(product_id),
    FOREIGN KEY (customer_id) REFERENCES Dim_Customer(customer_id)
);

INSERT INTO Fact_Sales (sale_id, date_id, product_id, customer_id, quantity, amount)
VALUES
  (1, 1, 1, 1, 2, 1799.98),
  (2, 2, 2, 2, 3, 1799.97);
```

Queries for Analysis
Here are some analytical queries based on the snowflake schema:

1. Total Sales by Product Category

```sql
SELECT
    dc.category_name,
    SUM(fs.amount) AS total_sales
FROM
    Fact_Sales fs
INNER JOIN Dim_Product dp ON fs.product_id = dp.product_id
INNER JOIN Dim_Category dc ON dp.category_id = dc.category_id
GROUP BY
    dc.category_name;
```

2. Top Products by Sales Quantity

```sql
SELECT
    dp.product_name,
    SUM(fs.quantity) AS total_quantity
FROM
    Fact_Sales fs
INNER JOIN Dim_Product dp ON fs.product_id = dp.product_id
GROUP BY
    dp.product_name
ORDER BY
    total_quantity DESC
LIMIT 10;
```

- Advantages of Snowflake Schema
-- Normalization: Reduces data redundancy.
-- Scalability: Easier to manage and add new dimensions or attributes.
-- Better Query Performance: For specific queries with normalized dimensions.




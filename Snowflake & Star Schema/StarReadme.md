## Star Schema

```sql
CREATE DATABASE sales_db;
USE sales_db;

CREATE TABLE Dim_Time (
    time_id INT PRIMARY KEY,
    date DATE,
    year INT,
    quarter INT,
    month INT,
    day INT,
    day_name VARCHAR(20)
);

INSERT INTO Dim_Time (time_id, date, year, quarter, month, day, day_name)
VALUES
  (1, '2023-01-01', 2023, 1, 1, 1, 'Sunday'),
  (2, '2023-02-15', 2023, 1, 2, 15, 'Wednesday'),
  (3, '2023-07-04', 2023, 3, 7, 4, 'Tuesday'),
  (4, '2023-12-25', 2023, 4, 12, 25, 'Monday');

CREATE TABLE Dim_Product (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    product_category VARCHAR(50),
    product_price DECIMAL(10,2)
);

INSERT INTO Dim_Product (product_id, product_name, product_category, product_price)
VALUES
  (1, 'Laptop', 'Electronics', 899.99),
  (2, 'Smartphone', 'Electronics', 599.99),
  (3, 'T-Shirt', 'Clothing', 19.99),
  (4, 'Jeans', 'Clothing', 49.99);

CREATE TABLE Dim_Customer (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    customer_city VARCHAR(50),
    customer_country VARCHAR(50)
);

INSERT INTO Dim_Customer (customer_id, customer_name, customer_city, customer_country)
VALUES
  (1, 'Alice Johnson', 'New York', 'USA'),
  (2, 'Bob Smith', 'London', 'UK'),
  (3, 'Charlie Brown', 'Paris', 'France'),
  (4, 'David Lee', 'Tokyo', 'Japan');

CREATE TABLE Fact_Sales (
    sale_id INT PRIMARY KEY,
    time_id INT,
    product_id INT,
    customer_id INT,
    quantity INT,
    amount DECIMAL(10,2),
    FOREIGN KEY (time_id) REFERENCES Dim_Time(time_id),
    FOREIGN KEY (product_id) REFERENCES Dim_Product(product_id),
    FOREIGN KEY (customer_id) REFERENCES Dim_Customer(customer_id)
);

INSERT INTO Fact_Sales (sale_id, time_id, product_id, customer_id, quantity, amount)
VALUES
  (1, 1, 1, 1, 2, 1799.98),
  (2, 2, 2, 2, 3, 1799.97),
  (3, 3, 3, 3, 5, 99.95),
  (4, 4, 4, 4, 2, 99.98);

```

## Performing Queries

- Once the tables are populated, you can perform various analytical queries. Here are a few examples:

- Total Sales by Product Category:

```sql
SELECT
    dp.product_category,
    SUM(fs.amount) AS total_sales
FROM
    Fact_Sales fs
INNER JOIN Dim_Product dp ON fs.product_id = dp.product_id
GROUP BY
    dp.product_category;
```

- Sales by Customer Country and Year:

````sql
SELECT
    dc.customer_country,
    dt.year,
    SUM(fs.amount) AS total_sales
FROM
    Fact_Sales fs
INNER JOIN Dim_Customer dc ON fs.customer_id = dc.customer_id
INNER JOIN Dim_Time dt ON fs.time_id = dt.time_id
GROUP BY
    dc.customer_country, dt.year;
    ```
````

- Top 10 Products by Sales:

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

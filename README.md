# 📝 Assignment: Database Design and Normalization

## 🎯 **Learning Objectives**
* 🛠️ **Understand the principles of good database design** and **normalization**.
* 💡 **Apply normalization techniques** to improve database structure and efficiency.
* 🔍 **Learn First, Second, and Third Normal Forms** (1NF, 2NF, 3NF) to eliminate redundancy and optimize data storage.

---

## 📋 **What You'll Need**
* 💻 A computer with internet access.
* ✍️ A code editor (e.g., Visual Studio Code).
* 🖥️ MySQL Workbench or another SQL database environment.

---


## 📝 Submission Instructions  
📂 Write all your SQL queries in the **answers.sql** file.  
✍️ Answer each question concisely and make sure your queries are clear and correct.  
🗣️ Structure your responses clearly, and use comments if necessary to explain your approach.

--- 

## 📚 Assignment Questions

---

### Question 1 Achieving 1NF (First Normal Form) 🛠️
Task:
- You are given the following table **ProductDetail**:

| OrderID | CustomerName  | Products                        |
|---------|---------------|---------------------------------|
| 101     | John Doe      | Laptop, Mouse                   |
| 102     | Jane Smith    | Tablet, Keyboard, Mouse         |
| 103     | Emily Clark   | Phone                           |


- In the table above, the **Products column** contains multiple values, which violates **1NF**.
- **Write an SQL query** to transform this table into **1NF**, ensuring that each row represents a single product for an order

--- 

### Question 2 Achieving 2NF (Second Normal Form) 🧩

- You are given the following table **OrderDetails**, which is already in **1NF** but still contains partial dependencies:

| OrderID | CustomerName  | Product      | Quantity |
|---------|---------------|--------------|----------|
| 101     | John Doe      | Laptop       | 2        |
| 101     | John Doe      | Mouse        | 1        |
| 102     | Jane Smith    | Tablet       | 3        |
| 102     | Jane Smith    | Keyboard     | 1        |
| 102     | Jane Smith    | Mouse        | 2        |
| 103     | Emily Clark   | Phone        | 1        |

- In the table above, the **CustomerName** column depends on **OrderID** (a partial dependency), which violates **2NF**. 

- Write an SQL query to transform this table into **2NF** by removing partial dependencies. Ensure that each non-key column fully depends on the entire primary key.

---
Good luck 🚀


-- answers.sql
-- SQL solutions for database normalization tasks

/* 
QUESTION 1: Converting to First Normal Form (1NF)
The original ProductDetail table violates 1NF with multi-valued Products column.
We'll create a new table with atomic values in each cell.
*/

-- Solution using STRING_SPLIT (for SQL Server)
CREATE TABLE ProductDetail_1NF AS
SELECT 
    OrderID,
    CustomerName,
    TRIM(value) AS Product
FROM ProductDetail
CROSS APPLY STRING_SPLIT(Products, ',');

-- Alternative solution for databases without STRING_SPLIT
/*
CREATE TABLE ProductDetail_1NF AS
SELECT OrderID, CustomerName, 'Laptop' AS Product 
FROM ProductDetail WHERE Products LIKE '%Laptop%'
UNION ALL
SELECT OrderID, CustomerName, 'Mouse' 
FROM ProductDetail WHERE Products LIKE '%Mouse%'
UNION ALL
SELECT OrderID, CustomerName, 'Tablet' 
FROM ProductDetail WHERE Products LIKE '%Tablet%'
UNION ALL
SELECT OrderID, CustomerName, 'Keyboard' 
FROM ProductDetail WHERE Products LIKE '%Keyboard%'
UNION ALL
SELECT OrderID, CustomerName, 'Phone' 
FROM ProductDetail WHERE Products LIKE '%Phone%';
*/

/*
QUESTION 2: Converting to Second Normal Form (2NF)
The OrderDetails table has partial dependency (CustomerName → OrderID).
We'll eliminate partial dependencies by splitting into two tables.
*/

-- Create Orders table (removes partial dependency)
CREATE TABLE Orders_2NF (
    OrderID INT PRIMARY KEY,
    CustomerName VARCHAR(100) NOT NULL
);

-- Populate Orders table
INSERT INTO Orders_2NF
SELECT DISTINCT OrderID, CustomerName
FROM OrderDetails;

-- Create OrderProducts table (full dependency on composite key)
CREATE TABLE OrderProducts_2NF (
    OrderID INT,
    Product VARCHAR(50),
    Quantity INT,
    PRIMARY KEY (OrderID, Product),
    FOREIGN KEY (OrderID) REFERENCES Orders_2NF(OrderID)
);

-- Populate OrderProducts table
INSERT INTO OrderProducts_2NF
SELECT OrderID, Product, Quantity
FROM OrderDetails;

-- Final schema meets 2NF requirements:
-- Orders_2NF(OrderID PK, CustomerName)
-- OrderProducts_2NF(OrderID PK/FK, Product PK, Quantity)

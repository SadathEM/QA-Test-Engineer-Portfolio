# SQL for QA Testing - Essential Guide
*By Sadath EM - QA Test Lead*

## Introduction

SQL is an indispensable skill for QA engineers. Throughout my career testing banking systems at Deutsche Bank, healthcare applications at Tieto EVRY, and telecom platforms at British Telecom, SQL has been my go-to tool for data validation, test data creation, and backend verification.

This guide covers essential SQL queries every QA engineer should master.

## 1. Why SQL Matters in QA

### Use Cases:
- ✅ **Data Validation**: Verify data accuracy in databases
- ✅ **Test Data Creation**: Insert, update test records
- ✅ **Backend Verification**: Validate UI actions reflected in DB
- ✅ **Defect Analysis**: Root cause analysis through data investigation
- ✅ **ETL Testing**: Validate data transformations
- ✅ **Performance Testing**: Check data volume, query performance
- ✅ **Integration Testing**: Verify data flow between systems

## 2. Basic SQL Queries for QA

### SELECT - Reading Data

```sql
-- Basic SELECT
SELECT * FROM users;

-- Select specific columns
SELECT user_id, username, email, status FROM users;

-- With WHERE clause
SELECT * FROM users WHERE status = 'active';

-- Multiple conditions
SELECT * FROM users 
WHERE status = 'active' AND created_date >= '2024-01-01';

-- Using IN operator
SELECT * FROM orders 
WHERE status IN ('pending', 'processing', 'shipped');

-- Using LIKE for pattern matching
SELECT * FROM users WHERE email LIKE '%@gmail.com';

-- Using NOT
SELECT * FROM users WHERE status != 'deleted';
```

### COUNT - Verify Record Counts

```sql
-- Total records
SELECT COUNT(*) FROM orders;

-- Count with condition
SELECT COUNT(*) FROM orders WHERE status = 'completed';

-- Count distinct values
SELECT COUNT(DISTINCT customer_id) FROM orders;

-- Group by count (very useful for testing)
SELECT status, COUNT(*) as count 
FROM orders 
GROUP BY status;
```

### INSERT - Creating Test Data

```sql
-- Single record insert
INSERT INTO users (username, email, status, created_date)
VALUES ('test_user_001', 'test001@example.com', 'active', NOW());

-- Multiple records insert
INSERT INTO users (username, email, status)
VALUES 
    ('test_user_002', 'test002@example.com', 'active'),
    ('test_user_003', 'test003@example.com', 'inactive'),
    ('test_user_004', 'test004@example.com', 'active');

-- Insert from SELECT (copy production data structure)
INSERT INTO test_orders (order_id, customer_id, amount, status)
SELECT order_id, customer_id, amount, status 
FROM orders 
WHERE created_date = '2024-01-01';
```

### UPDATE - Modifying Test Data

```sql
-- Update single record
UPDATE users 
SET status = 'inactive' 
WHERE user_id = 101;

-- Update multiple columns
UPDATE users 
SET status = 'active', last_login = NOW() 
WHERE user_id = 101;

-- Update with condition
UPDATE orders 
SET status = 'cancelled', cancelled_date = NOW() 
WHERE order_id IN (1001, 1002, 1003);

-- Update based on another table
UPDATE users u
SET u.total_orders = (SELECT COUNT(*) FROM orders o WHERE o.customer_id = u.user_id);
```

### DELETE - Cleaning Test Data

```sql
-- Delete specific record
DELETE FROM users WHERE user_id = 101;

-- Delete with condition
DELETE FROM users WHERE username LIKE 'test_user_%';

-- Delete all test data (be careful!)
DELETE FROM test_orders WHERE created_date < '2024-01-01';

-- Safe practice: Always SELECT before DELETE
SELECT * FROM users WHERE username LIKE 'test_user_%';
-- If results look correct, then:
DELETE FROM users WHERE username LIKE 'test_user_%';
```

## 3. Intermediate SQL for Testing

### JOIN Operations

```sql
-- INNER JOIN - Get orders with customer details
SELECT o.order_id, o.amount, u.username, u.email
FROM orders o
INNER JOIN users u ON o.customer_id = u.user_id;

-- LEFT JOIN - Get all customers with or without orders
SELECT u.user_id, u.username, COUNT(o.order_id) as total_orders
FROM users u
LEFT JOIN orders o ON u.user_id = o.customer_id
GROUP BY u.user_id, u.username;

-- Multiple JOINs
SELECT 
    o.order_id,
    u.username,
    p.product_name,
    o.amount
FROM orders o
INNER JOIN users u ON o.customer_id = u.user_id
INNER JOIN products p ON o.product_id = p.product_id;
```

### Aggregate Functions

```sql
-- SUM - Verify total amounts
SELECT SUM(amount) as total_revenue 
FROM orders 
WHERE status = 'completed';

-- AVG - Average calculations
SELECT AVG(amount) as average_order_value 
FROM orders;

-- MIN and MAX
SELECT 
    MIN(amount) as min_order,
    MAX(amount) as max_order,
    AVG(amount) as avg_order
FROM orders;

-- GROUP BY with aggregates
SELECT 
    status,
    COUNT(*) as order_count,
    SUM(amount) as total_amount,
    AVG(amount) as avg_amount
FROM orders
GROUP BY status;

-- HAVING clause (filter after grouping)
SELECT customer_id, COUNT(*) as order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 5;
```

### Subqueries

```sql
-- Subquery in WHERE clause
SELECT * FROM users
WHERE user_id IN (
    SELECT DISTINCT customer_id FROM orders WHERE amount > 1000
);

-- Subquery in SELECT clause
SELECT 
    user_id,
    username,
    (SELECT COUNT(*) FROM orders WHERE customer_id = users.user_id) as order_count
FROM users;

-- Subquery in FROM clause
SELECT avg_orders.customer_id, avg_orders.avg_amount
FROM (
    SELECT customer_id, AVG(amount) as avg_amount
    FROM orders
    GROUP BY customer_id
) as avg_orders
WHERE avg_orders.avg_amount > 500;
```

## 4. Advanced SQL for QA

### CASE Statements

```sql
-- Simple CASE
SELECT 
    order_id,
    amount,
    CASE 
        WHEN amount < 100 THEN 'Small'
        WHEN amount BETWEEN 100 AND 500 THEN 'Medium'
        WHEN amount > 500 THEN 'Large'
    END as order_size
FROM orders;

-- CASE with aggregation
SELECT 
    status,
    COUNT(CASE WHEN amount > 500 THEN 1 END) as high_value_orders,
    COUNT(CASE WHEN amount <= 500 THEN 1 END) as low_value_orders
FROM orders
GROUP BY status;
```

### Date Functions (Critical for Testing)

```sql
-- Current date/time
SELECT NOW(), CURDATE(), CURTIME();

-- Date filtering
SELECT * FROM orders 
WHERE created_date >= DATE_SUB(NOW(), INTERVAL 7 DAY);

-- Extract date parts
SELECT 
    DATE(created_date) as order_date,
    YEAR(created_date) as year,
    MONTH(created_date) as month,
    DAY(created_date) as day
FROM orders;

-- Date difference
SELECT 
    order_id,
    DATEDIFF(delivered_date, order_date) as delivery_days
FROM orders
WHERE delivered_date IS NOT NULL;

-- Format dates
SELECT DATE_FORMAT(created_date, '%Y-%m-%d %H:%i:%s') FROM orders;
```

### String Functions

```sql
-- Concatenation
SELECT CONCAT(first_name, ' ', last_name) as full_name FROM users;

-- Upper/Lower case
SELECT UPPER(email), LOWER(username) FROM users;

-- Substring
SELECT SUBSTRING(phone_number, 1, 3) as area_code FROM users;

-- Trim whitespace
SELECT TRIM(username) FROM users;

-- Length
SELECT username, LENGTH(username) FROM users;

-- Replace
SELECT REPLACE(email, '@gmail.com', '@testmail.com') FROM users;
```

## 5. Domain-Specific SQL Queries

### Banking Domain (Deutsche Bank Experience)

```sql
-- Validate payment transactions
SELECT 
    t.transaction_id,
    t.account_number,
    t.amount,
    t.transaction_type,
    t.status,
    a.balance
FROM transactions t
INNER JOIN accounts a ON t.account_id = a.account_id
WHERE t.transaction_date = CURDATE()
AND t.status = 'completed';

-- Verify SWIFT message validation
SELECT 
    swift_message_id,
    sender_bic,
    receiver_bic,
    amount,
    currency,
    validation_status
FROM swift_messages
WHERE created_date >= DATE_SUB(NOW(), INTERVAL 1 DAY)
AND validation_status = 'failed';

-- Check batch transaction processing
SELECT 
    batch_id,
    COUNT(*) as total_transactions,
    SUM(amount) as total_amount,
    SUM(CASE WHEN status = 'success' THEN 1 ELSE 0 END) as successful,
    SUM(CASE WHEN status = 'failed' THEN 1 ELSE 0 END) as failed
FROM batch_transactions
WHERE batch_date = CURDATE()
GROUP BY batch_id;

-- Validate loan application workflow
SELECT 
    l.loan_id,
    l.applicant_id,
    l.loan_amount,
    l.status,
    l.credit_score,
    a.repayment_schedule
FROM loans l
LEFT JOIN amortization_schedule a ON l.loan_id = a.loan_id
WHERE l.application_date >= '2024-01-01'
ORDER BY l.application_date DESC;
```

### Healthcare Domain (Tieto EVRY Experience)

```sql
-- Validate order management workflow
SELECT 
    o.order_id,
    o.patient_id,
    o.order_type,
    o.status,
    o.ordered_date,
    p.patient_name
FROM medical_orders o
INNER JOIN patients p ON o.patient_id = p.patient_id
WHERE o.ordered_date = CURDATE()
AND o.status IN ('pending', 'in_progress');

-- Check data accuracy in patient records
SELECT 
    patient_id,
    COUNT(*) as duplicate_count
FROM patients
GROUP BY patient_id
HAVING COUNT(*) > 1;

-- Verify regulatory compliance (data retention)
SELECT 
    COUNT(*) as old_records
FROM patient_records
WHERE last_updated_date < DATE_SUB(NOW(), INTERVAL 7 YEAR);
```

### Telecom Domain (British Telecom Experience)

```sql
-- Service activation validation
SELECT 
    s.service_id,
    s.customer_id,
    s.service_type,
    s.status,
    s.activation_date,
    c.customer_name
FROM services s
INNER JOIN customers c ON s.customer_id = c.customer_id
WHERE s.activation_date = CURDATE()
AND s.status = 'active';

-- Mobile Number Portability (MNP) testing
SELECT 
    mnp_request_id,
    mobile_number,
    current_operator,
    new_operator,
    request_status,
    porting_date
FROM mnp_requests
WHERE request_status = 'in_progress'
AND porting_date <= CURDATE();

-- Billing workflow validation
SELECT 
    b.bill_id,
    b.customer_id,
    b.bill_amount,
    b.bill_status,
    p.payment_status,
    p.payment_date
FROM bills b
LEFT JOIN payments p ON b.bill_id = p.bill_id
WHERE b.bill_month = '2024-02'
AND b.bill_status = 'generated';
```

## 6. ETL Testing SQL Queries

```sql
-- Row count validation (source vs target)
SELECT COUNT(*) FROM source_table;
SELECT COUNT(*) FROM target_table;

-- Data type validation
SELECT 
    COLUMN_NAME,
    DATA_TYPE,
    CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'target_table';

-- Null value check
SELECT 
    COUNT(*) as total_rows,
    SUM(CASE WHEN column_name IS NULL THEN 1 ELSE 0 END) as null_count
FROM target_table;

-- Duplicate check
SELECT 
    column1, column2,
    COUNT(*) as duplicate_count
FROM target_table
GROUP BY column1, column2
HAVING COUNT(*) > 1;

-- Data reconciliation
SELECT 
    s.id,
    s.amount as source_amount,
    t.amount as target_amount,
    s.amount - t.amount as difference
FROM source_table s
INNER JOIN target_table t ON s.id = t.id
WHERE s.amount != t.amount;
```

## 7. Performance Testing Queries

```sql
-- Check table size
SELECT 
    table_name,
    ROUND(((data_length + index_length) / 1024 / 1024), 2) as size_mb
FROM information_schema.TABLES
WHERE table_schema = 'your_database'
ORDER BY (data_length + index_length) DESC;

-- Find slow queries
SHOW PROCESSLIST;

-- Analyze query execution plan
EXPLAIN SELECT * FROM orders WHERE customer_id = 101;

-- Index usage check
SHOW INDEX FROM orders;

-- Check for missing indexes
SELECT * FROM orders WHERE email = 'test@example.com';
-- If slow, create index:
CREATE INDEX idx_email ON orders(email);
```

## 8. Common QA SQL Scenarios

### Scenario 1: Verify User Registration

```sql
-- Test: User registration creates record correctly
INSERT INTO users (username, email, password_hash, status)
VALUES ('new_user', 'new@test.com', 'hashed_pwd', 'active');

-- Verify insertion
SELECT * FROM users WHERE username = 'new_user';

-- Cleanup
DELETE FROM users WHERE username = 'new_user';
```

### Scenario 2: Test Order Workflow

```sql
-- Test: Complete order workflow
-- 1. Create order
INSERT INTO orders (customer_id, product_id, quantity, amount, status)
VALUES (101, 501, 2, 99.99, 'pending');

-- 2. Verify order created
SELECT * FROM orders WHERE customer_id = 101 ORDER BY created_date DESC LIMIT 1;

-- 3. Update order status
UPDATE orders SET status = 'processing' WHERE order_id = LAST_INSERT_ID();

-- 4. Complete order
UPDATE orders 
SET status = 'completed', completed_date = NOW() 
WHERE order_id = LAST_INSERT_ID();

-- 5. Verify final status
SELECT * FROM orders WHERE order_id = LAST_INSERT_ID();
```

### Scenario 3: Data Integrity Check

```sql
-- Test: Orphan records check
SELECT o.*
FROM orders o
LEFT JOIN users u ON o.customer_id = u.user_id
WHERE u.user_id IS NULL;

-- Test: Referential integrity
SELECT COUNT(*) as orphan_records
FROM order_items oi
LEFT JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_id IS NULL;
```

## 9. SQL Best Practices for QA

### DO's:
1. ✅ **Always use WHERE clause** when deleting/updating
2. ✅ **SELECT before DELETE/UPDATE** to verify
3. ✅ **Use transactions** for test data management
4. ✅ **Create indexes** on frequently queried columns
5. ✅ **Comment your queries** for documentation
6. ✅ **Use meaningful aliases** for table names
7. ✅ **Limit results** when testing large tables

### DON'Ts:
1. ❌ **Never run DELETE/UPDATE without WHERE** clause
2. ❌ **Don't use SELECT *** in production queries
3. ❌ **Avoid testing on production database**
4. ❌ **Don't hardcode values** - use variables
5. ❌ **Never commit without reviewing** changes

### Safety Practice - Use Transactions

```sql
-- Start transaction
START TRANSACTION;

-- Make changes
UPDATE users SET status = 'inactive' WHERE user_id = 101;

-- Verify changes
SELECT * FROM users WHERE user_id = 101;

-- If correct, commit
COMMIT;

-- If wrong, rollback
ROLLBACK;
```

## 10. SQL Cheat Sheet for Quick Reference

```sql
-- SELECT variations
SELECT * FROM table;
SELECT col1, col2 FROM table WHERE condition;
SELECT DISTINCT col FROM table;
SELECT * FROM table ORDER BY col DESC;
SELECT * FROM table LIMIT 10;

-- Filtering
WHERE col = value
WHERE col IN (val1, val2)
WHERE col BETWEEN val1 AND val2
WHERE col LIKE '%pattern%'
WHERE col IS NULL
WHERE col IS NOT NULL

-- Aggregations
COUNT(*), SUM(col), AVG(col), MIN(col), MAX(col)
GROUP BY col
HAVING condition

-- Joins
INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL OUTER JOIN

-- Date functions
NOW(), CURDATE(), DATE_ADD(), DATE_SUB(), DATEDIFF()

-- String functions
CONCAT(), UPPER(), LOWER(), TRIM(), SUBSTRING(), LENGTH()
```

## 11. Tools for SQL Testing

1. **MySQL Workbench** - For MySQL databases
2. **SQL Server Management Studio (SSMS)** - For SQL Server
3. **DBeaver** - Universal database tool
4. **pgAdmin** - For PostgreSQL
5. **Oracle SQL Developer** - For Oracle databases
6. **DataGrip** - JetBrains SQL IDE

## 12. Learning Path

### Beginner:
- Master SELECT, INSERT, UPDATE, DELETE
- Learn WHERE, ORDER BY, LIMIT
- Understand data types

### Intermediate:
- Master JOINs
- Learn aggregate functions
- Use subqueries
- Work with date/string functions

### Advanced:
- Complex queries with multiple JOINs
- Query optimization
- Stored procedures
- Triggers and functions

## Conclusion

SQL is a powerful tool in the QA arsenal. Whether you're validating data transformations in ETL processes, verifying backend transactions in banking applications, or ensuring data accuracy in healthcare systems, SQL helps you dig deeper and find issues that UI testing alone cannot catch.

Practice these queries regularly, and adapt them to your specific testing needs.

---

## Practice Exercises

I've created a set of 50+ practice SQL queries based on real-world testing scenarios. Download them from the Resources section.

---

*Have SQL questions or want to discuss database testing strategies? [Contact me](../contact.html).*

**Tags**: #SQL #DatabaseTesting #ETL #QA #DataValidation

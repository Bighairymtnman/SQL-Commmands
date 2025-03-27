# Advanced SQL Commands Reference

## Complex Querying Techniques

1. Window Functions
   ```sql
   SELECT 
       employee_name,
       salary,
       department,
       AVG(salary) OVER (PARTITION BY department) as dept_avg,
       RANK() OVER (ORDER BY salary DESC) as salary_rank,
       LAG(salary) OVER (ORDER BY hire_date) as previous_salary
   FROM employees;
   ```

2. Common Table Expressions (CTE)
   ```sql
   WITH recursive_cte AS (
       SELECT id, parent_id, name, 1 as level
       FROM organization
       WHERE parent_id IS NULL
       UNION ALL
       SELECT o.id, o.parent_id, o.name, rc.level + 1
       FROM organization o
       JOIN recursive_cte rc ON o.parent_id = rc.id
   )
   SELECT * FROM recursive_cte;
   ```





## Advanced Joins and Subqueries

1. Multiple Join Types
   ```sql
   SELECT 
       c.customer_name,
       o.order_date,
       p.product_name,
       i.quantity
   FROM customers c
   LEFT JOIN orders o ON c.customer_id = o.customer_id
   FULL OUTER JOIN order_items i ON o.order_id = i.order_id
   RIGHT JOIN products p ON i.product_id = p.product_id
   WHERE o.order_date >= (SELECT MAX(order_date) - INTERVAL '30 days' FROM orders);
   ```

2. Correlated Subqueries
   ```sql
   SELECT 
       department_name,
       employee_name,
       salary
   FROM employees e1
   WHERE salary = (
       SELECT MAX(salary)
       FROM employees e2
       WHERE e2.department_id = e1.department_id
   );
   ```

3. Lateral Joins
   ```sql
   SELECT 
       c.customer_name,
       recent_orders.*
   FROM customers c
   CROSS APPLY (
       SELECT order_id, order_date
       FROM orders o
       WHERE o.customer_id = c.customer_id
       ORDER BY order_date DESC
       LIMIT 3
   ) recent_orders;
   ```








## Advanced Data Manipulation

1. Merge Operations
   ```sql
   MERGE INTO target_table t
   USING source_table s
   ON (t.id = s.id)
   WHEN MATCHED THEN
       UPDATE SET t.column1 = s.column1
   WHEN NOT MATCHED THEN
       INSERT (id, column1)
       VALUES (s.id, s.column1);
   ```

2. Bulk Operations
   ```sql
   INSERT ALL
       INTO table1 (column1, column2) VALUES (value1, value2)
       INTO table2 (column1, column2) VALUES (value3, value4)
       INTO table3 (column1, column2) VALUES (value5, value6)
   SELECT * FROM dual;
   ```

3. Conditional Updates
   ```sql
   UPDATE employees
   SET salary = CASE
       WHEN department = 'IT' THEN salary * 1.15
       WHEN department = 'Sales' THEN salary * 1.10
       ELSE salary * 1.05
   END
   WHERE hire_date < CURRENT_DATE - INTERVAL '1 year';
   ```








## Performance Optimization

1. Index Optimization
   ```sql
   CREATE INDEX idx_composite 
   ON orders (customer_id, order_date)
   INCLUDE (total_amount)
   WHERE status = 'active';
   
   CREATE BITMAP INDEX idx_status
   ON orders(status);
   ```

2. Partitioning
   ```sql
   CREATE TABLE sales (
       sale_id NUMBER,
       sale_date DATE,
       amount NUMBER
   )
   PARTITION BY RANGE (sale_date) (
       PARTITION sales_2023 VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD')),
       PARTITION sales_2024 VALUES LESS THAN (TO_DATE('2025-01-01', 'YYYY-MM-DD'))
   );
   ```

3. Materialized Views
   ```sql
   CREATE MATERIALIZED VIEW mv_sales_summary
   REFRESH ON DEMAND
   AS
   SELECT 
       product_id,
       SUM(quantity) total_quantity,
       SUM(amount) total_amount
   FROM sales
   GROUP BY product_id;
   ```






## Advanced Stored Procedures and Functions

1. Complex Stored Procedure
   ```sql
   CREATE PROCEDURE process_sales_data(
       in_date DATE,
       in_region VARCHAR(50)
   )
   LANGUAGE plpgsql
   AS $$
   DECLARE
       v_total_amount DECIMAL(10,2);
   BEGIN
       -- Transaction handling
       BEGIN
           -- Process sales
           INSERT INTO sales_summary
           SELECT product_id, SUM(amount)
           FROM daily_sales
           WHERE sale_date = in_date
           AND region = in_region
           GROUP BY product_id;

           -- Update statistics
           UPDATE region_stats
           SET last_processed = CURRENT_TIMESTAMP
           WHERE region = in_region;

           COMMIT;
       EXCEPTION
           WHEN OTHERS THEN
               ROLLBACK;
               RAISE NOTICE 'Error: %', SQLERRM;
       END;
   END;
   $$;
   ```

2. Table-Valued Functions
   ```sql
   CREATE FUNCTION get_top_customers(
       min_purchase DECIMAL,
       from_date DATE
   )
   RETURNS TABLE (
       customer_id INT,
       total_spent DECIMAL,
       purchase_count INT
   )
   AS $$
   BEGIN
       RETURN QUERY
       SELECT 
           c.customer_id,
           SUM(o.amount),
           COUNT(*)
       FROM customers c
       JOIN orders o ON c.customer_id = o.customer_id
       WHERE o.order_date >= from_date
       GROUP BY c.customer_id
       HAVING SUM(o.amount) >= min_purchase;
   END;
   $$ LANGUAGE plpgsql;
   ```








## Advanced Security and Auditing

1. Row-Level Security
   ```sql
   CREATE POLICY data_access_policy ON employee_data
   FOR SELECT
   USING (
       department_id IN (
           SELECT dept_id 
           FROM user_departments 
           WHERE user_id = CURRENT_USER
       )
   );
   ```

2. Advanced Auditing
   ```sql
   CREATE TRIGGER audit_employee_changes
   AFTER INSERT OR UPDATE OR DELETE ON employees
   FOR EACH ROW
   BEGIN
       INSERT INTO audit_log (
           action_type,
           table_name,
           record_id,
           changed_by,
           change_timestamp,
           old_value,
           new_value
       )
       VALUES (
           TG_OP,
           'employees',
           COALESCE(NEW.id, OLD.id),
           CURRENT_USER,
           CURRENT_TIMESTAMP,
           row_to_json(OLD),
           row_to_json(NEW)
       );
   END;
   ```





## Advanced Database Maintenance

1. Statistics Management
   ```sql
   ANALYZE table_name;
   
   UPDATE pg_statistic
   SET statime = CURRENT_TIMESTAMP
   WHERE starelid = 'table_name'::regclass;
   ```

2. Vacuum Operations
   ```sql
   VACUUM (VERBOSE, ANALYZE) table_name;
   
   VACUUM FULL table_name;
   
   CLUSTER table_name USING index_name;
   ```

3. Table Space Management
   ```sql
   CREATE TABLESPACE fast_storage
   OWNER postgres
   LOCATION '/custom/location';
   
   ALTER TABLE large_table 
   SET TABLESPACE fast_storage;
   ```

4. Database Monitoring
   ```sql
   SELECT 
       datname,
       numbackends,
       xact_commit,
       xact_rollback,
       blks_read,
       blks_hit
   FROM pg_stat_database
   WHERE datname = current_database();
   ```
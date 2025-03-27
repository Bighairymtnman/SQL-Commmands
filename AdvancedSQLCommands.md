# Advanced SQL Commands Reference

## Complex Querying Techniques

/* Window functions enable calculations across row sets related to the current row.
   They're essential for analytical queries and running totals without using GROUP BY. */
1. Window Functions
   ```sql
   -- Calculates department averages, salary rankings, and previous salary based on hire date
   -- Useful for comparing individual values to group aggregates
   SELECT 
       employee_name,
       salary,
       department,
       AVG(salary) OVER (PARTITION BY department) as dept_avg, -- Department average without losing detail
       RANK() OVER (ORDER BY salary DESC) as salary_rank,      -- Ranks employees by salary
       LAG(salary) OVER (ORDER BY hire_date) as previous_salary -- Previous employee's salary
   FROM employees;
   ```

/* CTEs provide a way to write auxiliary statements for use in a larger query.
   Recursive CTEs are particularly useful for hierarchical or tree-structured data. */
2. Common Table Expressions (CTE)
   ```sql
   -- Traverses an organizational hierarchy to show reporting relationships
   -- Useful for org charts, category trees, or any hierarchical data
   WITH recursive_cte AS (
       -- Base case: start with top-level (parent) records
       SELECT id, parent_id, name, 1 as level
       FROM organization
       WHERE parent_id IS NULL
       UNION ALL
       -- Recursive case: join children with parents
       SELECT o.id, o.parent_id, o.name, rc.level + 1
       FROM organization o
       JOIN recursive_cte rc ON o.parent_id = rc.id
   )
   SELECT * FROM recursive_cte;
   ```

## Advanced Joins and Subqueries

/* Complex joins combine data from multiple tables while handling missing data
   and complex relationships between tables. */
1. Multiple Join Types
   ```sql
   -- Combines customer, order, and product data with different join types
   -- Useful for comprehensive reporting while preserving records from all tables
   SELECT 
       c.customer_name,
       o.order_date,
       p.product_name,
       i.quantity
   FROM customers c
   LEFT JOIN orders o ON c.customer_id = o.customer_id      -- Keeps all customers
   FULL OUTER JOIN order_items i ON o.order_id = i.order_id -- Keeps orphaned items
   RIGHT JOIN products p ON i.product_id = p.product_id     -- Keeps all products
   WHERE o.order_date >= (SELECT MAX(order_date) - INTERVAL '30 days' FROM orders);
   ```





/* Correlated subqueries reference the outer query and execute for each outer row.
   They're powerful for row-by-row comparisons and finding related data. */
2. Correlated Subqueries
   ```sql
   -- Finds highest paid employee in each department
   -- Useful for finding maximum values within groups
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

/* LATERAL joins allow subqueries in the FROM clause to reference columns from preceding tables.
   Perfect for row-specific calculations or limiting related data. */
3. Lateral Joins
   ```sql
   -- Gets the three most recent orders for each customer
   -- Ideal for showing limited sets of related records
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

/* MERGE operations combine INSERT, UPDATE, and DELETE actions in a single statement.
   Useful for synchronizing tables or handling upsert scenarios. */
1. Merge Operations
   ```sql
   -- Updates existing records and inserts new ones in a single operation
   -- Commonly used for data warehouse loading and synchronization
   MERGE INTO target_table t
   USING source_table s
   ON (t.id = s.id)
   WHEN MATCHED THEN
       UPDATE SET t.column1 = s.column1
   WHEN NOT MATCHED THEN
       INSERT (id, column1)
       VALUES (s.id, s.column1);
   ```




/* Bulk operations allow multiple data modifications in a single statement.
   They're more efficient than individual statements and maintain data consistency. */
2. Bulk Operations
   ```sql
   -- Inserts data into multiple tables in one statement
   -- Excellent for distributing related data across tables
   INSERT ALL
       INTO table1 (column1, column2) VALUES (value1, value2)
       INTO table2 (column1, column2) VALUES (value3, value4)
       INTO table3 (column1, column2) VALUES (value5, value6)
   SELECT * FROM dual;
   ```

/* Conditional updates modify data based on specific criteria.
   They're essential for complex business rules and data transformations. */
3. Conditional Updates
   ```sql
   -- Updates salaries with different percentages based on department
   -- Perfect for applying different rules to different data segments
   UPDATE employees
   SET salary = CASE
       WHEN department = 'IT' THEN salary * 1.15      -- 15% increase for IT
       WHEN department = 'Sales' THEN salary * 1.10   -- 10% increase for Sales
       ELSE salary * 1.05                             -- 5% increase for others
   END
   WHERE hire_date < CURRENT_DATE - INTERVAL '1 year';
   ```

## Performance Optimization

/* Indexes are crucial for query performance optimization.
   Different index types serve different query patterns. */
1. Index Optimization
   ```sql
   -- Creates a composite index with included columns
   -- Speeds up queries that filter by customer and date while selecting amount
   CREATE INDEX idx_composite 
   ON orders (customer_id, order_date)
   INCLUDE (total_amount)
   WHERE status = 'active';
   
   -- Bitmap indexes are efficient for low-cardinality columns
   CREATE BITMAP INDEX idx_status
   ON orders(status);
   ```






/* Partitioning divides large tables into smaller, manageable chunks.
   Improves query performance and data management for large datasets. */
2. Partitioning
   ```sql
   -- Creates a table partitioned by date range
   -- Enables efficient querying of time-based data
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

/* Materialized Views store pre-computed query results.
   Perfect for complex queries that don't need real-time data. */
3. Materialized Views
   ```sql
   -- Creates a materialized view for sales summaries
   -- Dramatically improves performance for frequent aggregate queries
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

/* Complex stored procedures handle multiple operations in a single transaction.
   They ensure data consistency and encapsulate business logic. */
1. Complex Stored Procedure
   ```sql
   -- Processes sales data with error handling and transaction management
   -- Ensures atomic operations and data integrity
   CREATE PROCEDURE process_sales_data(
       in_date DATE,
       in_region VARCHAR(50)
   )
   LANGUAGE plpgsql
   AS $$
   DECLARE
       v_total_amount DECIMAL(10,2);
   BEGIN
       -- Transaction handling ensures all operations succeed or none do
       BEGIN
           -- Aggregates and inserts sales data
           INSERT INTO sales_summary
           SELECT product_id, SUM(amount)
           FROM daily_sales
           WHERE sale_date = in_date
           AND region = in_region
           GROUP BY product_id;

           -- Updates processing timestamp
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




/* Table-valued functions return result sets rather than single values.
   They're powerful for reusable data transformations and complex calculations. */
2. Table-Valued Functions
   ```sql
   -- Returns a table of top customers based on purchase criteria
   -- Excellent for flexible reporting and data analysis
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

/* Row-Level Security (RLS) implements fine-grained access control.
   Essential for multi-tenant databases and data privacy. */
1. Row-Level Security
   ```sql
   -- Restricts data access based on user department
   -- Automatically filters queries based on user context
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

/* Audit triggers automatically track all data changes.
   Critical for compliance and data change tracking. */
2. Advanced Auditing
   ```sql
   -- Creates comprehensive audit trail for employee data changes
   -- Captures who changed what and when
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
           TG_OP,                    -- Operation type (INSERT/UPDATE/DELETE)
           'employees',              -- Table being audited
           COALESCE(NEW.id, OLD.id), -- Record identifier
           CURRENT_USER,             -- User making the change
           CURRENT_TIMESTAMP,        -- When the change occurred
           row_to_json(OLD),         -- Previous record state
           row_to_json(NEW)          -- New record state
       );
   END;
   ```

## Advanced Database Maintenance

/* Regular statistics management ensures optimal query planning.
   Critical for maintaining database performance. */
1. Statistics Management
   ```sql
   -- Updates table statistics for query optimizer
   -- Improves query plan generation
   ANALYZE table_name;
   
   -- Updates statistics timestamp
   UPDATE pg_statistic
   SET statime = CURRENT_TIMESTAMP
   WHERE starelid = 'table_name'::regclass;
   ```

/* Vacuum operations reclaim storage and update statistics.
   Essential for maintaining database health. */
2. Vacuum Operations
   ```sql
   -- Analyzes table and updates statistics
   VACUUM (VERBOSE, ANALYZE) table_name;
   
   -- Reclaims space and defragments the table
   VACUUM FULL table_name;
   
   -- Physically reorders table based on index
   CLUSTER table_name USING index_name;
   ```

/* Tablespace management optimizes storage distribution.
   Important for I/O optimization and storage management. */
3. Table Space Management
   ```sql
   -- Creates new tablespace on fast storage
   -- Useful for performance-critical tables
   CREATE TABLESPACE fast_storage
   OWNER postgres
   LOCATION '/custom/location';
   
   -- Moves table to new tablespace
   ALTER TABLE large_table 
   SET TABLESPACE fast_storage;
   ```

/* Database monitoring helps track performance and usage.
   Essential for proactive database management. */
4. Database Monitoring
   ```sql
   -- Retrieves key database performance metrics
   -- Helps identify performance issues and usage patterns
   SELECT 
       datname,           -- Database name
       numbackends,       -- Number of active connections
       xact_commit,       -- Successful transactions
       xact_rollback,     -- Failed transactions
       blks_read,         -- Disk blocks read
       blks_hit          -- Buffer hits
   FROM pg_stat_database
   WHERE datname = current_database();
   ```

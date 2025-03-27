# SQL Commands Reference

## Data Definition Language (DDL) Commands

### CREATE Commands

1. Create Table

   ```sql
   CREATE TABLE table_name (
       column1 data_type [NULL | NOT NULL],
       column2 data_type [NULL | NOT NULL],
       CONSTRAINT constraint_name PRIMARY KEY (column1)
   );
   ```

2. Create Table From Existing Table

   ```sql
   CREATE TABLE new_table AS
   SELECT column1, column2
   FROM existing_table
   [WHERE conditions];
   ```

3. Create Index

   ```sql
   CREATE INDEX index_name
   ON table_name (column_name);
   ```

4. Create View
   ```sql
   CREATE VIEW view_name AS
   SELECT column1, column2
   FROM table_name
   [WHERE conditions]
   [GROUP BY column1]
   [HAVING conditions];
   ```

### ALTER Commands

1. Alter Table
   ```sql
   ALTER TABLE table_name
   [ADD column_name datatype]
   [MODIFY column_name datatype]
   [DROP COLUMN column_name]
   [ADD CONSTRAINT constraint_name];
   ```

### DROP Commands

1. Drop Table

   ```sql
   DROP TABLE table_name;
   ```

2. Drop Index

   ```sql
   DROP INDEX index_name;
   ```

3. Drop View
   ```sql
   DROP VIEW view_name;
   ```

## Data Manipulation Language (DML) Commands

### Basic Data Operations

1. Select Data

   ```sql
   SELECT [DISTINCT] column1, column2
   FROM table_name
   [WHERE conditions]
   [GROUP BY column1]
   [HAVING group_conditions]
   [ORDER BY column1 [ASC | DESC]];
   ```

2. Insert Data

   ```sql
   INSERT INTO table_name (column1, column2)
   VALUES (value1, value2);
   ```

3. Update Data

   ```sql
   UPDATE table_name
   SET column1 = value1,
       column2 = value2
   WHERE conditions;
   ```

4. Delete Data
   ```sql
   DELETE FROM table_name
   WHERE conditions;
   ```

## Transaction Control

1. Commit Changes

   ```sql
   COMMIT;
   ```

2. Rollback Changes

   ```sql
   ROLLBACK [TO SAVEPOINT savepoint_name];
   ```

3. Create Savepoint
   ```sql
   SAVEPOINT savepoint_name;
   ```

## Security Commands

1. Create User

   ```sql
   CREATE USER username
   IDENTIFIED BY password;
   ```

2. Grant Privileges

   ```sql
   GRANT privilege1, privilege2
   ON object_name
   TO user_name;
   ```

3. Revoke Privileges
   ```sql
   REVOKE privilege1, privilege2
   FROM user_name;
   ```

## Query Clauses

### Essential Clauses

1. WHERE Clause (Filtering)

   ```sql
   WHERE column1 = value1
   AND column2 IN (value2, value3)
   OR column3 LIKE 'pattern%';
   ```

2. GROUP BY Clause (Aggregating)

   ```sql
   GROUP BY column1, column2
   HAVING aggregate_function(column) operator value;
   ```

3. ORDER BY Clause (Sorting)
   ```sql
   ORDER BY column1 ASC,
            column2 DESC;
   ```

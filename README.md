# sqlCmmands
Some of The Most Important SQL Commands

-- Database-Level
DROP DATABASE databaseName                 -- Delete the database (irrecoverable!)
DROP DATABASE IF EXISTS databaseName       -- Delete if it exists
CREATE DATABASE databaseName               -- Create a new database
CREATE DATABASE IF NOT EXISTS databaseName -- Create only if it does not exists
SHOW DATABASES                             -- Show all the databases in this server
USE databaseName                           -- Set the default (current) database
SELECT DATABASE()                          -- Show the default database
SHOW CREATE DATABASE databaseName          -- Show the CREATE DATABASE statement
 
-- Table-Level
DROP TABLE [IF EXISTS] tableName, ...
CREATE TABLE [IF NOT EXISTS] tableName (
   columnName columnType columnAttribute, ...
   PRIMARY KEY(columnName),
   FOREIGN KEY (columnNmae) REFERENCES tableName (columnNmae)
)
SHOW TABLES                -- Show all the tables in the default database
DESCRIBE|DESC tableName    -- Describe the details for a table
ALTER TABLE tableName ...  -- Modify a table, e.g., ADD COLUMN and DROP COLUMN
ALTER TABLE tableName ADD columnDefinition
ALTER TABLE tableName DROP columnName
ALTER TABLE tableName ADD FOREIGN KEY (columnNmae) REFERENCES tableName (columnNmae)
ALTER TABLE tableName DROP FOREIGN KEY constraintName
SHOW CREATE TABLE tableName        -- Show the CREATE TABLE statement for this tableName
 
-- Row-Level
INSERT INTO tableName 
   VALUES (column1Value, column2Value,...)               -- Insert on all Columns
INSERT INTO tableName 
   VALUES (column1Value, column2Value,...), ...          -- Insert multiple rows
INSERT INTO tableName (column1Name, ..., columnNName)
   VALUES (column1Value, ..., columnNValue)              -- Insert on selected Columns
DELETE FROM tableName WHERE criteria
UPDATE tableName SET columnName = expr, ... WHERE criteria
SELECT * | column1Name AS alias1, ..., columnNName AS aliasN 
   FROM tableName
   WHERE criteria
   GROUP BY columnName
   ORDER BY columnName ASC|DESC, ...
   HAVING groupConstraints
   LIMIT count | offset count
 
-- Others
SHOW WARNINGS;   -- Show the warnings of the previous statement


INSERT INTO Syntax

INSERT INTO tableName VALUES (firstColumnValue, ..., lastColumnValue)  -- All columns

INSERT INTO tableName VALUES 
   (row1FirstColumnValue, ..., row1lastColumnValue),
   (row2FirstColumnValue, ..., row2lastColumnValue), 
   ...

-- Insert single record with selected columns
INSERT INTO tableName (column1Name, ..., columnNName) VALUES (column1Value, ..., columnNValue)
-- Alternately, use SET to set the values
INSERT INTO tableName SET column1=value1, column2=value2, ...
 
-- Insert multiple records
INSERT INTO tableName 
   (column1Name, ..., columnNName)
VALUES 
   (row1column1Value, ..., row2ColumnNValue),
   (row2column1Value, ..., row2ColumnNValue),
   ...

Querying the Database - SELECT

-- List all the rows of the specified columns
SELECT column1Name, column2Name, ... FROM tableName
   
-- List all the rows of ALL columns, * is a wildcard denoting all columns
SELECT * FROM tableName
  
-- List rows that meet the specified criteria in WHERE clause
SELECT column1Name, column2Name,... FROM tableName WHERE criteria
SELECT * FROM tableName WHERE criteria

ORDER BY Clause

SELECT ... FROM tableName
WHERE criteria
ORDER BY columnA ASC|DESC, columnB ASC|DESC, ...

Modifying Data - UPDATE

UPDATE tableName SET columnName = {value|NULL|DEFAULT}, ... WHERE criteria

Deleting Rows - DELETE FROM

-- Delete all rows from the table. Use with extreme care! Records are NOT recoverable!!!
DELETE FROM tableName
-- Delete only row(s) that meets the criteria
DELETE FROM tableName WHERE criteria

Loading/Exporting Data from/to a Text File

LOAD DATA LOCAL INFILE ... INTO TABLE ...

sample

products_in.csv:

\N,PEC,Pencil 3B,500,0.52
\N,PEC,Pencil 4B,200,0.62
\N,PEC,Pencil 5B,100,0.73
\N,PEC,Pencil 6B,500,0.47

-- Need to use forward-slash (instead of back-slash) as directory separator
mysql> LOAD DATA LOCAL INFILE 'd:/myProject/products_in.csv' INTO TABLE products
         COLUMNS TERMINATED BY ','
         LINES TERMINATED BY '\r\n';

mysql> SELECT * FROM products;
+-----------+-------------+-----------+----------+-------+
| productID | productCode | name      | quantity | price |
+-----------+-------------+-----------+----------+-------+
|      1007 | PEC         | Pencil 3B |      500 |  0.52 |
|      1008 | PEC         | Pencil 4B |      200 |  0.62 |
|      1009 | PEC         | Pencil 5B |      100 |  0.73 |
|      1010 | PEC         | Pencil 6B |      500 |  0.47 |
+-----------+-------------+-----------+----------+-------+


mysqlimport Utility Program

-- SYNTAX
> mysqlimport -u username -p --local databaseName tableName.tsv
   -- The raw data must be kept in a TSV (Tab-Separated Values) file with filename the same as tablename
 
-- EXAMPLES
-- Create a new file called "products.tsv" containing the following record,
--  and saved under "d:\myProject"
-- The values are separated by tab (not spaces).
\N  PEC  Pencil 3B  500  0.52
\N  PEC  Pencil 4B  200  0.62
\N  PEC  Pencil 5B  100  0.73
\N  PEC  Pencil 6B  500  0.47

> cd path-to-mysql-bin
> mysqlimport -u root -p --local southwind d:/myProject/products.tsv

SELECT ... INTO OUTFILE ...

mysql> SELECT * FROM products INTO OUTFILE 'd:/myProject/products_out.csv' 
         COLUMNS TERMINATED BY ','
         LINES TERMINATED BY '\r\n';

Running a SQL Script

load_products.sql:

DELETE FROM products;
INSERT INTO products VALUES (2001, 'PEC', 'Pencil 3B', 500, 0.52),
                            (NULL, 'PEC', 'Pencil 4B', 200, 0.62),
                            (NULL, 'PEC', 'Pencil 5B', 100, 0.73),
                            (NULL, 'PEC', 'Pencil 6B', 500, 0.47);
SELECT * FROM products;

mysql> source d:/myProject/load_products.sql
   -- Use Unix-style forward slash (/) as directory separator

 "batch mode" of the mysql client program, by re-directing the input from the script:

> cd path-to-mysql-bin
> mysql -u root -p southwind < d:\myProject\load_products.sql

More Than One Tables

One-To-Many Relationship

mysql> USE southwind;
   
mysql> DROP TABLE IF EXISTS suppliers;
   
mysql> CREATE TABLE suppliers (
         supplierID  INT UNSIGNED  NOT NULL AUTO_INCREMENT, 
         name        VARCHAR(30)   NOT NULL DEFAULT '', 
         phone       CHAR(8)       NOT NULL DEFAULT '',
         PRIMARY KEY (supplierID)
       );
   
mysql> DESCRIBE suppliers;
+------------+------------------+------+-----+---------+----------------+
| Field      | Type             | Null | Key | Default | Extra          |
+------------+------------------+------+-----+---------+----------------+
| supplierID | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| name       | varchar(30)      | NO   |     |         |                |
| phone      | char(8)          | NO   |     |         |                |
+------------+------------------+------+-----+---------+----------------+
   
mysql> INSERT INTO suppliers VALUE
          (501, 'ABC Traders', '88881111'), 
          (502, 'XYZ Company', '88882222'), 
          (503, 'QQ Corp', '88883333');
   
mysql> SELECT * FROM suppliers;
+------------+-------------+----------+
| supplierID | name        | phone    |
+------------+-------------+----------+
|        501 | ABC Traders | 88881111 |
|        502 | XYZ Company | 88882222 |
|        503 | QQ Corp     | 88883333 |
+------------+-------------+----------+


ALTER TABLE

mysql> ALTER TABLE products
       ADD COLUMN supplierID INT UNSIGNED NOT NULL;
Query OK, 4 rows affected (0.13 sec)
Records: 4  Duplicates: 0  Warnings: 0
   
mysql> DESCRIBE products;
+-------------+------------------+------+-----+------------+----------------+
| Field       | Type             | Null | Key | Default    | Extra          |
+-------------+------------------+------+-----+------------+----------------+
| productID   | int(10) unsigned | NO   | PRI | NULL       | auto_increment |
| productCode | char(3)          | NO   |     |            |                |
| name        | varchar(30)      | NO   |     |            |                |
| quantity    | int(10) unsigned | NO   |     | 0          |                |
| price       | decimal(10,2)    | NO   |     | 9999999.99 |                |
| supplierID  | int(10) unsigned | NO   |     | NULL       |                |
+-------------+------------------+------+-----+------------+----------------+

-- Set the supplierID of the existing records in "products" table to a VALID supplierID
--   of "suppliers" table
mysql> UPDATE products SET supplierID = 501;
 
-- Add a foreign key constrain
mysql> ALTER TABLE products
       ADD FOREIGN KEY (supplierID) REFERENCES suppliers (supplierID);
 
mysql> DESCRIBE products;
+-------------+------------------+------+-----+------------+----------------+
| Field       | Type             | Null | Key | Default    | Extra          |
+-------------+------------------+------+-----+------------+----------------+
  ......
| supplierID  | int(10) unsigned | NO   | MUL |            |                |
+-------------+------------------+------+-----+------------+----------------+
 
mysql> UPDATE products SET supplierID = 502 WHERE productID  = 2004;
  -- Choose a valid productID
   
mysql> SELECT * FROM products;
+-----------+-------------+-----------+----------+-------+------------+
| productID | productCode | name      | quantity | price | supplierID |
+-----------+-------------+-----------+----------+-------+------------+
|      2001 | PEC         | Pencil 3B |      500 |  0.52 |        501 |
|      2002 | PEC         | Pencil 4B |      200 |  0.62 |        501 |
|      2003 | PEC         | Pencil 5B |      100 |  0.73 |        501 |
|      2004 | PEC         | Pencil 6B |      500 |  0.47 |        502 |
+-----------+-------------+-----------+----------+-------+------------+

SELECT with JOIN

-- ANSI style: JOIN ... ON ...
mysql> SELECT products.name, price, suppliers.name 
       FROM products 
          JOIN suppliers ON products.supplierID = suppliers.supplierID
       WHERE price < 0.6;
+-----------+-------+-------------+
| name      | price | name        |
+-----------+-------+-------------+
| Pencil 3B |  0.52 | ABC Traders |
| Pencil 6B |  0.47 | XYZ Company |
+-----------+-------+-------------+
    -- Need to use products.name and suppliers.name to differentiate the two "names"
 
-- Join via WHERE clause (lagacy and not recommended)
mysql> SELECT products.name, price, suppliers.name 
       FROM products, suppliers 
       WHERE products.supplierID = suppliers.supplierID
          AND price < 0.6;
+-----------+-------+-------------+
| name      | price | name        |
+-----------+-------+-------------+
| Pencil 3B |  0.52 | ABC Traders |
| Pencil 6B |  0.47 | XYZ Company |
+-----------+-------+-------------+

-- Use aliases for column names for display
mysql> SELECT products.name AS `Product Name`, price, suppliers.name AS `Supplier Name` 
       FROM products 
          JOIN suppliers ON products.supplierID = suppliers.supplierID
       WHERE price < 0.6;
+--------------+-------+---------------+
| Product Name | price | Supplier Name |
+--------------+-------+---------------+
| Pencil 3B    |  0.52 | ABC Traders   |
| Pencil 6B    |  0.47 | XYZ Company   |
+--------------+-------+---------------+
 
-- Use aliases for table names too
mysql> SELECT p.name AS `Product Name`, p.price, s.name AS `Supplier Name` 
       FROM products AS p 
          JOIN suppliers AS s ON p.supplierID = s.supplierID
       WHERE p.price < 0.6;

Many-To-Many Relationship

mysql> CREATE TABLE products_suppliers (
         productID   INT UNSIGNED  NOT NULL,
         supplierID  INT UNSIGNED  NOT NULL,
                     -- Same data types as the parent tables
         PRIMARY KEY (productID, supplierID),
                     -- uniqueness
         FOREIGN KEY (productID)  REFERENCES products  (productID),
         FOREIGN KEY (supplierID) REFERENCES suppliers (supplierID)
       );
   
mysql> DESCRIBE products_suppliers;
+------------+------------------+------+-----+---------+-------+
| Field      | Type             | Null | Key | Default | Extra |
+------------+------------------+------+-----+---------+-------+
| productID  | int(10) unsigned | NO   | PRI | NULL    |       |
| supplierID | int(10) unsigned | NO   | PRI | NULL    |       |
+------------+------------------+------+-----+---------+-------+
  
mysql> INSERT INTO products_suppliers VALUES (2001, 501), (2002, 501),
       (2003, 501), (2004, 502), (2001, 503);
-- Values in the foreign-key columns (of the child table) must match 
--   valid values in the columns they reference (of the parent table)
   
mysql> SELECT * FROM products_suppliers;
+-----------+------------+
| productID | supplierID |
+-----------+------------+
|      2001 |        501 |
|      2002 |        501 |
|      2003 |        501 |
|      2004 |        502 |
|      2001 |        503 |
+-----------+------------+

mysql> SHOW CREATE TABLE products \G
Create Table: CREATE TABLE `products` (
  `productID`   int(10) unsigned  NOT NULL AUTO_INCREMENT,
  `productCode` char(3)           NOT NULL DEFAULT '',
  `name`        varchar(30)       NOT NULL DEFAULT '',
  `quantity`    int(10) unsigned  NOT NULL DEFAULT '0',
  `price`       decimal(7,2)      NOT NULL DEFAULT '99999.99',
  `supplierID`  int(10) unsigned   NOT NULL DEFAULT '501',
  PRIMARY KEY (`productID`),
  KEY `supplierID` (`supplierID`),
  CONSTRAINT `products_ibfk_1` FOREIGN KEY (`supplierID`) 
     REFERENCES `suppliers` (`supplierID`)
) ENGINE=InnoDB AUTO_INCREMENT=1006 DEFAULT CHARSET=latin1
 
mysql> ALTER TABLE products DROP FOREIGN KEY products_ibfk_1;

mysql> SHOW CREATE TABLE products \G

mysql> ALTER TABLE products DROP supplierID;
 
mysql> DESC products;


Querying


Similarly, we can use SELECT with JOIN to query data from the 3 tables, for examples,

mysql> SELECT products.name AS `Product Name`, price, suppliers.name AS `Supplier Name`
       FROM products_suppliers 
          JOIN products  ON products_suppliers.productID = products.productID
          JOIN suppliers ON products_suppliers.supplierID = suppliers.supplierID
       WHERE price < 0.6;
+--------------+-------+---------------+
| Product Name | price | Supplier Name |
+--------------+-------+---------------+
| Pencil 3B    |  0.52 | ABC Traders   |
| Pencil 3B    |  0.52 | QQ Corp       |
| Pencil 6B    |  0.47 | XYZ Company   |
+--------------+-------+---------------+

-- Define aliases for tablenames too 
mysql> SELECT p.name AS `Product Name`, s.name AS `Supplier Name`
       FROM products_suppliers AS ps 
          JOIN products AS p ON ps.productID = p.productID
          JOIN suppliers AS s ON ps.supplierID = s.supplierID
       WHERE p.name = 'Pencil 3B';
+--------------+---------------+
| Product Name | Supplier Name |
+--------------+---------------+
| Pencil 3B    | ABC Traders   |
| Pencil 3B    | QQ Corp       |
+--------------+---------------+
  
-- Using WHERE clause to join (legacy and not recommended)
mysql> SELECT p.name AS `Product Name`, s.name AS `Supplier Name`
       FROM products AS p, products_suppliers AS ps, suppliers AS s
       WHERE p.productID = ps.productID
          AND ps.supplierID = s.supplierID
          AND s.name = 'ABC Traders';
+--------------+---------------+
| Product Name | Supplier Name |
+--------------+---------------+
| Pencil 3B    | ABC Traders   |
| Pencil 4B    | ABC Traders   |
| Pencil 5B    | ABC Traders   |
+--------------+---------------+



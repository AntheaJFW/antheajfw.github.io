---
title: "Database Design"
collection: projects
tags:
    - python
    - notebook
--- 
## Methods of organising and managing data:

* Schemas: How should my data be logically organized?
* Normalization: Should my data have minimal dependency and redundancy?
* Views: What joins will be done most often?
* Access control: Should all users of the data have the same level of access?
* DBMS: How do I pick between all the SQL and noSQL options?

It depends on the intended use of data 
 
Approaches to processing data:

* OLTP - Online Transaction Processing
    * Oriented around *Transactions*
        * Informational: Keep track of the prices of books
        * Update latest customer transaction
        * Keep Track of employee hours
    * Focused on supporting day to day operations
* OLAP - Online Analytical Processing
    * Oriented around *Analytics*
        * Analytical: Calculate books with the best profit margin
        * Sophisticed analysis ie loyal customers
        * Decide employee of the month
    * Focused on business decision making 
 
## Ways to store data

1. Structured data
    * Follows a schema
    * Defined data types & relationships
        * Datatypes, tables and relationships between tables are defined.
Examples = SQL tables in relational database like foreign keys
    * Easier to organise
    * But not flexible
2. Unstructured data
    * Schemaless
    * Examples media files, photos etc
3. Semi-structured data
    * Does not follow larger schema
    * Self-describing strctured
    * Examples: NoSQL, JSON, XML 
 
* Traditional databases
    * Good for storing real-time relational structured data. Likely OLTP
* Data Warehouses
    * For analysing achived structured data OLAP
    * Optimising read-only analytics
    * Contains data from multiple sources
    * Massively parallel processing (MPP)
    * Typically uses a denormalized schema and dimensional modeling
    * Examples include: Amazon redshift, azure SQL data warehouse, google big
query
* Data Lakes
    * For storing data of all structures = flexible and scalable
    * For analysing big data
    * Cheaper. Uses object storage
    * Stores all types of data
        * ie IoT device logs, real-time, relational and non-relational
    * Retains all data and can take up petabyes
    * Uses Schema-on-read as opposed to schema-on-write
    * Need to catalog data otherwise it becomes data swamp
    * Run big data analytics using services such as **Apache spark and Hadoop**

May choose data warehouse over data lakes if it is known that it will be used
for analysis 
 
## Approaches to **storing** data - data flows
* ETL - Extract, Transform, Load
    * Data is extracted, and then transformed to fit the storages schema before
loaded into a data warehouse for later use
* ELT - Extract, Load, Transform
    * Data is stored in its **native** form in a datalake. Some of the data are
transformed later for use, including building a data warehouse 
 
When creating relational database, often it is best to have **Fact tables**
holding Facts about a certain index / occurance, and other **Dimensional
tables** holding tables of information of the attributes of that occurance.

For example, if the data may be about runs you've made, tables holding
information on their attributes may be:
- Park name
- Route name
- Location
Fact table may contain:
- Run duration
- Foreign key to route name...
etc

Because of its structure, the dimensional model usually requires queries
involving more than one table



Fact tables
- Holds records of a metric
- Changes regularly
- Connects to dimensions via foreign key

Dimension tables
- Holds descriptions of attributes
- Does not change often 
 
## Different schemas
- [Star Schemas](https://en.wikipedia.org/wiki/Star_schema)
    - Extends one dimension
- [Snowflake schema](https://en.wikipedia.org/wiki/Snowflake_schema)
    - Extends on more than one dimension
    - More *Normalized** 
 
Normalization is a database technique to divide tables into smaller tables and
connect them via relationships

Goal is to reduce redundancy and increase data integrity

Therefore this is done by identifying repeating groups of data, and create new
tables 
 
## Modifying databases to use Star / Snowflake schema

* Add foreign keys to the Fact table
    * ```mysql
    ALTER TABLE fact_table ADD CONSTRAINT constraint_name
    FOREIGN KEY (fact_table_foreign_key) REFERENCES foreign_table
(foreign_table_primary_key);```
* Create a new table to extend on a dimension
    * ```mysql
    -- Create a new table
    CREATE TABLE new_dimension (
        column_to_refer_to dtype NOT NULL
    )

    -- Insert data to extend the dimension with into the new table
    INSERT INTO new_dimension
    SELECT DISTINCT column_to_refer_to FROM original_table;

    -- Add a primary key to refer to the new table
    ALTER TABLE new_dimension ADD COLUMN primary_key_name SERIAL PRIMARY KEY;
    ``` 
 
## Pros / Cons of normalising databases
Pros:
- Normalization saves space
- Eliminates data redundancy
- Ensures data consistency and data integrity
    i.e. 'Singapore' can be inputted as 'SG' or etc
- Safer updating removing and inserting as there are less records to alter
- Easier to alter the database schema as you only have to update the smaller
tables

Cons:
- There will be considerably more joins
- Make queries more complicated
- Requires more CPU 
 
Goals of normalization:
- Be able to characterize the level of redundancy in relational schema
- Provide mechanisms for transforming schemas in order to remove redundancy 
 
## Normal forms (NF) 
 
Normalized forms (Ordered Least to most normalized form from
[wikipedia](https://en.wikipedia.org/wiki/Database_normalization)):
- UNF: Unnormalized form
- 1NF: First normal form
- 2NF: Second normal form
- 3NF: Third normal form
- EKNF: Elementary key normal form
- BCNF: Boyce–Codd normal form
- 4NF: Fourth normal form
- ETNF: Essential tuple normal form
- 5NF: Fifth normal form
- DKNF: Domain-key normal form
- 6NF: Sixth normal form 
 
1NF
- Each record must be unique - no duplicate rows
- Each cells must hold one value (ie. column with data like 'abc, def, feg' as a
value should be in a seperate table and assigned one row per entry)

2NF
- Satisfy 1NF AND:
    - Primary key in a column OR;
    - Each non-key column is dependent on **all** of the Composite primary key
        - If not dependent on **all** of the composite primary keys, then split
into another table

3NF
- Satisty 2NF
- No **transitive dependencies** = Non-key columns should not depend on other
non-key columns
 
 
## Why normalize?

Databases that are not normalized can exhibit:
1. Update anomaly
    - Data inconsistency caused by data redundancy when updating
        - ie If a table holds redundant records, **all** of these rows need to
be updated simultaneously
2. Insertion anomaly
    - Unable to add records due to missing attributes
3. Deletion anomaly
    - Deleting record may cause unintentional loss of data
        - i.e. deleting a certain record also deletes other records 
 
---
## Database Views 
 
Views are virtual tables that are not part of the physical schema. It is the
result set of a stored query on the data.

- Query is stored, and not data
- Data is then aggregated from the data in the tables
- Users may query like a regular database table
- Allows users to access common queries without querying the tables or altering
schemas 
 
Creating views:
```mysql
-- Save query as name:
CREATE VIEW view_name AS
-- Query to save:
SELECT col1, col2
FROM table
WHERE condition;
``` 
 
Accessing views available in your database (postgresql):
```mysql
SELECT * FROM INFORMATION_SCHEMA.views;
``` 
 
Pros of Views:
- Does not require storage
- Acts as a form of access control by allowing users to only access view
- Mask complex queries
    - Useful for highly normalized schemas 
 
Views should not be updated as updating views updates the tables for which the
views query against. 
 
## Altering / redefining a view
```mysql
CREATE OR REPLACE VIEW view_name AS new_query
```
- Can only replace if new_query generates the same Column names, Order and
datatypes
- Column output may be differnt
- New columns may be added to the end

Otherwise, drop existing view, and create a new view 
 
## Views can be created with other views
Motivation: When the schema is complex (e.g from being highly normalised),
having views help reduce the joins needed. 
 
## Granting and revoking user privileges to avoid unintended edits

[Granting (Postgresql)](https://www.postgresql.org/docs/9.1/sql-grant.html)
```mysql
GRANT update, insert ON {view/table} TO {username/public};
```
Other grant options include
{ SELECT | INSERT | UPDATE | DELETE | TRUNCATE | REFERENCES | TRIGGER }

[Revoking (Postgresql)](https://www.postgresql.org/docs/9.1/sql-revoke.html)
```mysql
REVOKE update, INSERT ON {view/table} FROM {username/public};
```
Other revoke options include
{ SELECT | INSERT | UPDATE | DELETE | TRUNCATE | REFERENCES | TRIGGER } 
 
## Materialised and Non-Materialised views
 

**In [None]:**

{% highlight python %}

{% endhighlight %}

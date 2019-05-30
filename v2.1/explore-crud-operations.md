---
title: Explore CockroachDB CRUD Operations
summary: Use a local database to explore CockroachDB CRUD operations.
toc: true
---

These steps use CockroachDB to create a table and run the four basic database
"CRUD" operations: Create, Read, Update, and Delete. In CockroachDB, you perform
these operations with the following SQL statements: [`INSERT
INTO`](insert.html), [`SELECT`](select-clause.html), [`UPDATE`](update.html),
and [`DELETE FROM`](delete.html).

## Before you begin

If you've not yet installed CockroachDB, [do so now](install-cockroachdb.html).

If you've run CockroachDB from a previous tutorial, stop any running instances.

## Step 1. Create a new directory for this tutorial and change to that directory

{% include copy-clipboard.html %}
~~~ shell
$ mkdir cockroachdb-sql-test
~~~

{% include copy-clipboard.html %}
~~~ shell
$ cd cockroachdb-sql-test
~~~

## Step 2. Start a single-node CockroachDB cluster

A single node is sufficient to test basic SQL statements; in production,
however, you would use a multi-node cluster. By listening only to localhost, you
can run the node in insecure mode and avoid having to create security
certificates. To run the node, issue the following:

{% include copy-clipboard.html %}
~~~ shell
$ cockroach start --insecure --listen-addr=localhost
~~~

The node starts and displays the [standard
output](start-a-node.html#standard-output), including the URL for the built-in
SQL client, which you will use in the next step.

{{site.data.alerts.callout_info}}
The SQL client is built into the `cockroach` binary. No separate install is needed.
{{site.data.alerts.end}}

## Step 3. Connect to the SQL client

Open a new terminal and connect to the [SQL client](use-the-built-in-sql-client.html):

{% include copy-clipboard.html %}
~~~ shell
$ cockroach sql --insecure
~~~

## Step 4. Create a database and a table

For this tutorial, you'll create a table of customer emails used by a retail shop.
Use [`CREATE DATABASE`](create-database.html) to create a database called "shop:"

{% include copy-clipboard.html %}
~~~ sql
> CREATE DATABASE shop;
~~~

Use [`CREATE TABLE`](create-table.html) to create the "clients" table with four
columns. Enter the column names and [data types](data-types.html) inside the
parentheses:

{% include copy-clipboard.html %}
~~~ sql
> CREATE TABLE shop.clients (id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
name STRING,
email STRING,
interests STRING[]);
~~~

The columns use the following [data types](data-types.html):

- As a best practice, the "id" column uses the [`UUID`](uuid.html) data type and
  sets the default value to the `gen_random_uuid()` function. This generates
  random unique IDs in parallel when new data is inserted and is recommended as
  a performance best practice. The "id" column is also the table's [primary
  key](primary-key.html).

- The "name" and "email" columns use the [`STRING`](string.html) data type.

- The "interests" column uses a STRING [`ARRAY`](array.html), allowing you to list multiple interests.

To view column information, issue the following:

{% include copy-clipboard.html %}
~~~ sql
> SHOW COLUMNS FROM shop.clients;
~~~

~~~
  column_name | data_type | is_nullable |  column_default   | generation_expression |  indices  | is_hidden  
+-------------+-----------+-------------+-------------------+-----------------------+-----------+-----------+
  id          | UUID      |    false    | gen_random_uuid() |                       | {primary} |   false    
  name        | STRING    |    true     | NULL              |                       | {}        |   false    
  email       | STRING    |    true     | NULL              |                       | {}        |   false    
  interests   | STRING[]  |    true     | NULL              |                       | {}        |   false    
(4 rows)
~~~

## Step 5. Insert data into the table

Use the [`INSERT INTO`](insert.html) statement to add your first row. The parentheses
after `VALUES` list the row's values in the order in which the columns appear in the table:

{% include copy-clipboard.html %}
~~~ sql
> INSERT INTO shop.clients VALUES (gen_random_uuid(), 'Elvin Jones', 'elvin@example.com', ARRAY['drums', 'books', 'jazz']);
~~~

View the table:

{% include copy-clipboard.html %}
~~~ sql
> SELECT * FROM shop.clients;
~~~

~~~
                   id                  |     name      |        email        |        interests         
+--------------------------------------+---------------+---------------------+-------------------------+
  9435f821-7d72-4151-aacb-c16ecc80f991 | Elvin Jones   | elvin@example.com   | {drums,music,jazz}       
(1 row)
~~~

Add two more rows using a single `INSERT INTO` statement. Adding multiple rows at once with one statement provides faster
performance than issuing multiple, separate statements:

{% include copy-clipboard.html %}
~~~ sql
> INSERT INTO shop.clients VALUES (gen_random_uuid(), 'Joe Jones', 'joe@example.com', ARRAY['drums', 'housewares', 'bepop']), (gen_random_uuid(), 'William Jones', 'william@example.com', ARRAY['drums', 'recording equipment']);
~~~

View the updated table:

{% include copy-clipboard.html %}
~~~ sql
> SELECT * FROM shop.clients;
~~~

~~~
                   id                  |     name      |        email        |        interests         
+--------------------------------------+---------------+---------------------+-------------------------+
  9435f821-7d72-4151-aacb-c16ecc80f991 | Elvin Jones   | elvin@example.com   | {drums,music,jazz}       
  e7f6ba28-ff31-411c-bdce-915b70361249 | Joe Jones     | joe@example.com     | {drums,recording,bepop}  
  fdbba65b-58b9-40c5-a08e-7da06f60d2b4 | William Jones | william@example.com | {drums,performance}      
(3 rows)
~~~

If you'd like, add more rows with your own values.

## Step 6. Read the data

Use the [`SELECT`](select-clause.html) statement to read the table's data. In the last step, 
you read *all* the table's data by issuing `SELECT` followed by the `*` wildcard
operator:

`SELECT * FROM shop.clients;`

To query for specific data but not all data, expand the `SELECT` clause. To
return only certain columns, add the column names after `SELECT`, separated by
commas:

{% include copy-clipboard.html %}
~~~ sql
> SELECT name, email
FROM shop.clients;
~~~

~~~
      name      |        email         
+---------------+---------------------+
  Elvin Jones   | elvin@example.com    
  Joe Jones     | joe@example.com      
  William Jones | william@example.com  
(3 rows)
~~~

To read only certain rows, add the `WHERE` clause. The following command returns the
row that contains information for Joe Jones:

{% include copy-clipboard.html %}
~~~ sql
> SELECT name, email
FROM shop.clients
WHERE name = 'Joe Jones';
~~~

~~~
    name    |      email       
+-----------+-----------------+
  Joe Jones | joe@example.com  
(1 row)
~~~

To find all clients who have the last name Jones, add `LIKE` to the `WHERE`
clause and use the `%` wildcard operator:

{% include copy-clipboard.html %}
~~~ sql
> SELECT name, email
FROM shop.clients
WHERE name LIKE '%Jones';
~~~

~~~
      name      |        email         
+---------------+---------------------+
  Elvin Jones   | elvin@example.com    
  Joe Jones     | joe@example.com      
  William Jones | william@example.com  
(3 rows)
~~~

For additional ways to filter data, see [`SELECT`](select-clause.html).

## Step 7. Update data

Use the [`UPDATE`](update.html) statement to change the table's data. The
following command changes William Jones's email address from william@example.com
to wjones@example.net. The `SET` clause provides the new address. The `WHERE`
clause identifies that the update is for William Jones:

{% include copy-clipboard.html %}
~~~ sql
> UPDATE shop.clients SET email = 'wjones@example.net' WHERE name = 'William Jones';
~~~

To view the change, use the `SELECT` statement:

{% include copy-clipboard.html %}
~~~ sql
> SELECT name, email
FROM shop.clients
WHERE name = 'William Jones';
~~~

~~~
      name      |       email        
+---------------+-------------------+
  William Jones | wjones@example.net  
(1 row)
~~~

## Step 8. Delete data

Use the [`DELETE FROM`](delete.html) statement to remove rows from the table.
Use the `WHERE` clause to identify the row to delete. The following command
deletes the row for Elvin Jones:

{% include copy-clipboard.html %}
~~~ sql
> DELETE FROM shop.clients
WHERE name = 'Elvin Jones';
~~~

Confirm the deletion by viewing all the table's data:

{% include copy-clipboard.html %}
~~~ sql
> SELECT * FROM shop.clients;
~~~

~~~
                   id                  |     name      |        email       |       interests          
+--------------------------------------+---------------+--------------------+-------------------------+
  e7f6ba28-ff31-411c-bdce-915b70361249 | Joe Jones     | joe@example.com    | {drums,recording,bepop}  
  fdbba65b-58b9-40c5-a08e-7da06f60d2b4 | William Jones | wjones@example.net | {drums,performance}      
(2 rows)
~~~

## What's next?

To learn more about SQL in CockroachDB, see [SQL
Statements](sql-statements.html) and the [SQL FAQs](sql-faqs).

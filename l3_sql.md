# 50.043 - SQL

## Learning Outcomes

By the end of this unit you should be able to use SQL to

1. create and alter databases and tables
2. create and alter table constraints
3. inject data into tables
4. retrieve data from tables
5. update/delete data from tables


## SQL 

SQL is a high-level language for data definition and data manipulation. By high-level, we mean it is 
* expressive
* closer to the programmers

SQL is a declarative programming language. By declarative, we use SQL
to specify what to do but not how to do. (The "how-to" parts are left to the underlying runtime to decide, i.e. the DBMS query operation module).

SQL is almost universal and cross platforms. Modern big data and non-relation databases extend to support SQL.

Note that different DBMSes have different subset of SQL statements. In this unit, we try to cover the common ones.

## Data definition language

Let's consider the DDL of SQL. 


### Create and drop database 

Note that in a DBMS, there may be many different databases, identified by their names. 

To create a database, we may use the following SQL statement.

```sql
create database if not exists db_name;
```

where `db_name` is the name of the database; `if not exists` means the create statement only applies when there is no existing database in the DBMS with name `db_name`. 

Note that SQL is case-insensitive. 

To drop a database

```sql
drop database if exists db_name;
```

To rename a database

```sql
alter database db_name rename to my_db; 
```

However in some DBMS, e.g. MySQL, the above statement is rejected. Instead, we need to dump the old database and load it into the new database. 


### Create table

```sql
create table if not exists my_db.article (
    id char(100) primary key,
    name char(100)
);
```

In the above statement, we create the `article` table in the database `my_db`. `article` has two attributes, `id` and `name`. Both attributes are of type `char(100)`, i.e. character sequence with max length = 100. `id` is set to be primary key of the table. 

Similarly, we create the `book`, `publisher` and `publish` tables.

```sql
create table if not exists  my_db.book (
    id char(100) primary key,
    name char(100)
);

create table if not exists my_db.publisher (
    id char(100) primary key,
    name char(100)
);
```

Finally, we create the table `publish` which was translated from a tertiary relationship.

```sql
create table if not exists my_db.publish (
    article_id char(100),
    book_id char(100),
    publisher_id char(100),
    primary key (article_id, book_id),
    foreign key (article_id) references my_db.article(id),
    foreign key (book_id) references my_db.book(id),
    foreign key (publisher_id) references my_db.publisher(id)
);
```
Since the primary key is a composite key, it is specified in a separate clause (line 5). In addition, we have to create three foreign key constraints on `article_id`, `book_id` and `publisher_id` to ensure their existence in the entity table `article`, `book` and `publisher`. 


### Alter table

We can alter a table (e.g. name, attribute name, attribute type, constraints) using the alter statement. For instance, 

```sql
-- drop primary key
alter table my_db.publish drop primary key;
-- recreate primary key
alter table my_db.publish add primary key (article_id, publisher_id);
```

The first alter statement drop the primary key, and the second one recreate a new primary using `(article_id, publisher_id)` composition. 

##### Some DBMS Implementation Specific Details
Note that in some DBMS implementation, we need to drop the foreign key constraints before we can drop the primary key. For instance MySQL, the foreign key constraint automatically create an index on the attribute, in this case `article_id` and `book_id`. It uses the existing index from the existing primary key constraint. 

In case of MySQL, we need to first find out the name of the foreign key constraints 

```sql
select table_schema, table_name, column_name, constraint_name  from information_schema.key_column_usage where table_name = 'publish';
+--------------+------------+--------------+-----------------+
| TABLE_SCHEMA | TABLE_NAME | COLUMN_NAME  | CONSTRAINT_NAME |
+--------------+------------+--------------+-----------------+
| my_db        | publish    | article_id   | PRIMARY         |
| my_db        | publish    | book_id      | PRIMARY         |
| my_db        | publish    | article_id   | publish_ibfk_1  |
| my_db        | publish    | book_id      | publish_ibfk_2  |
| my_db        | publish    | publisher_id | publish_ibfk_3  |
+--------------+------------+--------------+-----------------+
5 rows in set (0.00 sec)
```


Then execute the following statements before 
the `drop primary key` statement.

```sql
alter table my_db.publish drop foreign key publish_ibfk_1;
alter table my_db.publish drop foreign key publish_ibfk_2;
```

Then execute the following statements after the `add primary key` statement

```sql
alter table my_db.publish add foreign key (article_id) references my_db.article(id) ;
alter table my_db.publish add foreign key (book_id) references my_db.book(id);
```

For more detals of alter table statement for MySQL, refer to the MySQL [documentation](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html).

### Drop table

To drop a table, we could run

```sql
drop table my_db.publish; 
-- but don't do it, it is irreversible, instead we should probably to rename it to something else.
```

### Injecting value

To inject values, we use the insert statement.

```sql
insert into my_db.article (id, name) values 
('a1', 'article 1'),
('a2', 'article 2'); 

insert into my_db.book (id, name) values 
('b1', 'book 1'),
('b2', 'book 2');

insert into my_db.publisher (id, name) values
('p1', 'publisher 1'),
('p2', 'publisher 2');
```

Note that we can omit the schema `(id, name)` when values for all columsn are present. Furthermore, when inserting values into a table with foreign key constraint, e.g. `publish` references `article`, `book` and `publisher`, the values to be inserted must respect the existence of the referenced keys from the referenced tables.

```sql
insert into my_db.publish (article_id, book_id, publisher_id) values
('a1', 'b1', 'p1'),
('a2', 'b1', 'p1'),
('a1', 'b2', 'p2');
```

##### Mass importing and exporting

In some situation, it is inefficient to inject values one by one via insert statement. Many DBMS implementations offer means to import and export data from text file or other format. For MySQL, please refer to this [document](https://www.digitalocean.com/community/tutorials/how-to-import-and-export-databases-in-mysql-or-mariadb) and this [document](https://www.mysqltutorial.org/import-csv-file-mysql-table/).


### Querying table

To retrieve data stored in a table, we use the select statement.

```sql
select article_id, book_id, publisher_id  from my_db.publish;
```
In the above statement we retrieve all records (tuples) from the `publish` table. In this case, since we are retrieving all columns, we could re-write the above as follows,

```sql
select * from my_db.publish;
```

##### Export to CSV 

In some implemntation, such as MySQL, we could use the select statement to export the data in a table into a CSV file. 

```sql
select * from my_db.publish into outfile '/tmp/publish.csv' fields terminated by ','; 
```


### Join-Query

When querying multiple table, we would use the inner join.

```sql
select * from my_db.publish 
inner join my_db.article 
on my_db.publish.article_id = my_db.article.id;
```

For breivity, we could give aliases to the tables being joined.

```sql
select * from my_db.publish p 
inner join my_db.article a 
on p.article_id = a.id;
```
The above queries produce

```
+------------+---------+--------------+----+-----------+
| article_id | book_id | publisher_id | id | name      |
+------------+---------+--------------+----+-----------+
| a1         | b1      | p1           | a1 | article 1 |
| a1         | b2      | p2           | a1 | article 1 |
| a2         | b1      | p1           | a2 | article 2 |
+------------+---------+--------------+----+-----------+
```


The left- and right- outter join queries can be expressed in a similar way by replacing `inner` by `left` or `right`.


### Where clause 
Suppose we would like to find all the article names that are published by publisher `p1`.

```sql
select a.name from my_db.publish p 
inner join my_db.article a 
on p.article_id = a.id
where p.publisher_id = 'p1';
```
which produces

```
+-----------+
| name      |
+-----------+
| article 1 |
| article 2 |
+-----------+
```

Note that instead of `inner join`, we can rewrite the above query using `equi-join` which is pushing the id matching to the filtering operation.


```sql
select a.name from my_db.publish p, my_db.article a 
where p.article_id = a.id
and p.publisher_id = 'p1';
```

This is because the following equation holds in relational algebra

$$
publish \bowtie_{publish.article_id = article.id} article = \sigma_{publish.article_id = article.id}(publish \times article)
$$


### Self join

Suppose we want to find all articles that are published by both publisher `p1` and `p2`.

The following query 

```sql
select a.* from my_db.publish p, my_db.article a
where p.article_id = a.id
and p.publisher_id = 'p1'
and p.publisher_id = 'p2';
```
won't produce any result. This is due to the fac that the entire conjunction predicate in the `where` clause is applied to per tuple level. Since there is no tuple having `publisher_id` as `p1` and `p2` at the same time, the result is an empty set.

In such situation, we need to join a table to itself. 


```sql
select a.* from my_db.publish p1, my_db.publish p2, my_db.article a
where p1.article_id = a.id
and p2.article_id = a.id
and p1.publisher_id = 'p1'
and p2.publisher_id = 'p2';
```
In the above, we "clone" the `publish` table twice, then the join are performed among the two clones and the article table.

### Nested query

Alternatively to self-join, we could express the above query using nested query.

```sql
select a1.* from my_db.publish p1, my_db.article a1
where p1.article_id = a1.id
and p1.publisher_id = 'p1'
and a1.id in (select a2.id from my_db.publish p2, my_db.article a2
              where p2.article_id = a2.id
              and p2.publisher_id = 'p2' 
             );
```

In the above, we find a nested query, the outer query joins `publish` with `article` and filters out those tuples with `publisher_id` as `p1`. The last predicate checks the article id must be found in the result of the nested query. The nested query joins the clones of the two tables and filter tuples with `publisher_id` equal to `p2`. 


### Aggregation

For analytic purpose, we need to aggregate values by group.

For example, the following statement counts the number of tuples in the `publish` table.

```sql
select count(*) from my_db.publish;
```


Suppose we would like to counts the number of published article published by publisher `p1`.

```sql
select publisher_id, count(*) from my_db.publish 
group by publisher_id;
```
```
+--------------+----------+
| publisher_id | count(*) |
+--------------+----------+
| p1           |        2 |
| p2           |        1 |
+--------------+----------+
```

In the above the `group by` clause specifies the attribute `publisher_id` is the attribute the groups created by, i.e. tuples within each group should have the same `publisher_id`. 

The above SQL statement is equivalent to the following relational algebra express

$$
_{publisher\_id}\gamma_{count(*)}(publish)
$$

### Sorting

Suppose we want to sort the result of the last query by the counts in ascending order.

```sql
select publisher_id, count(*) as cnt from my_db.publish 
group by publisher_id
order by cnt asc;
```

In the above the `as cnt` creates an alias for the column `count(*)` for the ease of references. The `order by` clause specifies the order of the returned results.

```
+--------------+-----+
| publisher_id | cnt |
+--------------+-----+
| p2           |   1 |
| p1           |   2 |
+--------------+-----+
```

### Update and delete 

To update  tuples/records in table, we use the update statement.

```sql
update my_db.publisher set name = 'publisher one' 
where name = 'publisher 1';
```
The above SQL statement updates all tuples's name to `publisher one` in `publisher` table with the existing name as `publisher 1`.


To delete tuples/records, we use the delete statement.

```sql
delete from my_db.publisher where name = 'publisher 1';
```
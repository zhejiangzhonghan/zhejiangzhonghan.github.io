---
layout: post
title: "InnoDB(MySQL) Lock Gotchas"
date: 2016-05-10 10:36:16
comments: true
description: "Innodb(MySQL) Lock Gotchas"
keywords: "Innodb, MySQL, lock, tutorial, gotchas, deadlock"
categories:
- development
- tutorial
tags:
- mysql
- python
---
If you are not verify familiar with InnoDB, please go to the basics first:
[InnoDB(MySQL) Lock Tutorial](/2016/05/innodbmysql-lock-tutorial)

Let's start by reviewing the following sentence from [MySQL Doc](http://dev.mysql.com/doc/refman/5.7/en/innodb-locks-set.html)

> A locking read, a UPDATE, or a DELETE generally **set record locks on every index record that is scanned** in the processing of the SQL statement. 
> It **does not matter whether there are WHERE conditions in the statement** that would exclude the row. InnoDB does not remember the exact WHERE condition, but only knows which index ranges were scanned.
>

Ok, read through it again and keep those bold text in mind. 

## What does it means in real coding?

### Index!

Let's start with a very simple demo:

Here is our table and data

```sql
CREATE TABLE `demo1` (
`id` INT, 
`value` INT, 
PRIMARY KEY (`id`)) ENGINE = InnoDB;

insert into `demo1` value (1, 100);
insert into `demo1` value (2, 200);
insert into `demo1` value (3, 300);
insert into `demo1` value (4, 400);
```

Thread 1 starts a transaction and wants to grab lock on row with value 300:

```sql
start transaction;
select * from demo1 where value=300 for update;
```

It looks likes a very intuitive solution. 

Now Let's guess how many locks does this transaction have in InnoDB? The answer is **ALL ROWS** in demo table. 

What happened? It comes down to the fact that InnoDB locks very row it scans to get your lock. Since we don't have an index on `value`, the engine has to scan all rows in the table and as a result, the whole table gets locked.

The correct version of the table should be:

```sql
CREATE TABLE `demo1` (
`id` INT, 
`value` INT, 
KEY `demo_index` (`value`),
PRIMARY KEY (`id`)) ENGINE = InnoDB;
```

Now if you want to execute the same query again, you will find we are only locking one row as expected this time.

#### Conclusion: 
- Without a correct index, row lock is very dangerous as InnoDB will scan and lock every row it meets to parse your query. 
- So, never try to lock row without index on the filter condition

### Where? 

Here comes the second demo:

```sql
CREATE TABLE `demo1` (
`id` INT, 
`value` INT, 
KEY `demo_index` (`value`),
PRIMARY KEY (`id`)) ENGINE = InnoDB;

insert into `demo1` value (1, 100);
insert into `demo1` value (2, 200);
insert into `demo1` value (3, 300);
insert into `demo1` value (4, 400);
```

#### Not Equal
Thread starts a transaction and tries to lock rows except value 200:

```sql
start transaction;
select * from demo1 where value != 200 for update;
```
Number of locked rows: All rows!

#### Compare
Thread starts a transaction and tries to lock rows larger than 205:

```sql
start transaction;
select * from demo1 where value > 205 for update;
```
Number of locked rows: All rows!

#### Conclusion:
- Innodb locks all row in the scenarios of 'not equal', '>', '<'
- In coding, we should try to use “=” only if possible even if the column is indexed.
- One exception: `Is Null` and `Is Not Null` works as expect

### Table Join?
The behavior becomes even more complicated when it comes down to table join.

Let's start with two linked tables:

```sql
CREATE TABLE parent (
    id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
    name CHAR(60) NOT NULL,
    PRIMARY KEY (id)
)ENGINE=InnoDB ;

CREATE TABLE child (
    `id` SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
    `name` CHAR(60) NOT NULL,
    `parent_id` SMALLINT UNSIGNED,
    PRIMARY KEY (`id`),
    constraint `fk_parent_id_exit` foreign key (`parent_id`) references `parent`(`id`)
)ENGINE=InnoDB ;

insert into parent value (1, 'p1');
insert into parent value (2, 'p2');
insert into parent value (3, 'p3');
insert into child value(1, 'p3_c1', 3);
```

Let's do a left outer join from parent to child to see the current state:

```sql
mysql> select * from parent left outer join child on parent.id = child.parent_id;
+----+------+------+-------+-----------+
| id | name | id   | name  | parent_id |
+----+------+------+-------+-----------+
|  1 | p1   | NULL | NULL  |      NULL |
|  2 | p2   | NULL | NULL  |      NULL |
|  3 | p3   |    1 | p3_c1 |         3 |
+----+------+------+-------+-----------+
```

#### Good World

```sql
start transaction;
select * from parent right join child on parent.id = child.parent_id 
where child.parent_id = 3 for update;
```
Number of locked rows: one (parent id 3)

#### Bad World

```sql
start transaction;
select * from parent left outer join child on parent.id = child.parent_id 
where child.parent_id is null for update;
```
Number of locked rows: All rows!

#### Conclusion:
- InnoDB lock the correct row if you can specify the index value
- Innodb has difficult on handling locks for table join when asked to check null value and will lock everything instead



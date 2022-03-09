---
layout: post
title: "InnoDB(MySQL) Lock Tutorial"
date: 2016-05-10 10:35:42
comments: true
description: "Innodb(MySQL) Lock Tutorial"
keywords: "Innodb, MySQL, lock, tutorial, basic, deadlock"
categories:
- development
- tutorial
tags:
- mysql
- python
---

While I was working to solve some deadlock issues in our platform, I found some sql query we are using actually locks much more rows than we expected. I search through internet and couldn't find any good resource to explain what's going on. So I read through the official docs and tries a bunch of different example to find out how to correctly map sql query to the corresponding locks I see in the DB. I would like to share it out in case someone else is looking for details as I did last week.

I will begin with some basics and then cover the unexpected locking behavior from InnoDB. Although I think the following details are very important, if you think you are very familiar with it, you can skip to:
[InnoBD(MySQL) Lock Gotchas](/2016/05/innodbmysql-lock-gotchas)

Reference [MySQL official doc](http://dev.mysql.com/doc/refman/5.7/en/innodb-storage-engine.html)

## InnoDB basic
InnoDB is the storage engine for MySQL and it allows transaction feature and row-level locking. 

In InnoDB, all user activity occurs inside a transaction. If autocommit mode is enabled, each SQL statement forms a single transaction on its own. 

A session that has autocommit enabled can perform a multiple-statement transaction by starting it with an explicit START TRANSACTION or BEGIN statement and ending it with a COMMIT or ROLLBACK statement.

If autocommit mode is disabled within a session with SET autocommit = 0, the session always has a transaction open. A COMMIT or ROLLBACK statement ends the current transaction and a new one starts.

Both COMMIT and ROLLBACK release all InnoDB locks that were set during the current transaction

In row-level locking, InnoDB normally uses next-key locking (see below). That means that besides index records, InnoDB can also lock the gap preceding an index record to block insertions by other sessions where the indexed values would be inserted in that gap within the tree data structure. A next-key lock refers to a lock that locks an index record and the gap before it. A gap lock refers to a lock that locks only the gap before some index record.

## Isolation Level

|Level|description|
|---|---|
|REPEATABLE READ|all reads within a transaction are against the snapshot at the time of the first read in transaction|
|READ COMMITTED|reads will read committed data, even if it was committed after the first read in the transaction|
|READ UNCOMMITTED|SELECT statements are performed in a nonlocking fashion, but a possible earlier version of a row might be used. Thus, using this isolation level, such reads are not consistent. This is also called a dirty read. |
|SERIALIZABLE|InnoDB implicitly converts all plain SELECT statements to SELECT ... LOCK IN SHARE MODE|

## Lock Mode

- A shared (S) lock permits the transaction that holds the lock to read a row.
- An exclusive (X) lock permits the transaction that holds the lock to update or delete a row.
- Intention shared (IS): Transaction T intends to set S locks on individual rows in table t.
- Intention exclusive (IX): Transaction T intends to set X locks on those rows.

For example, SELECT ... LOCK IN SHARE MODE sets an IS lock and SELECT ... FOR UPDATE sets an IX lock.

The intention locking protocol is as follows:

- Before a transaction can acquire an S lock on a row in table t, it must first acquire an IS or stronger lock on t.

- Before a transaction can acquire an X lock on a row, it must first acquire an IX lock on t.

||X|IX|S|IS|
|---|---|---|---|---|
|X|Conflict|Conflict|Conflict|Conflict|
|IX|Conflict|Compatible|Conflict|Compatible|
|S|Conflict|Conflict|Compatible|Compatible|
|IS|Compatible|Compatible|Compatible|Compatible|

## Lock Type

- Record lock: This is a lock on an index record.

- Gap lock: This is a lock on a gap between index records, or a lock on the gap before the first or after the last index record. ( conflicting gap locks are allowed. that why two insertions are allowed at same time )

- Next-key lock: This is a combination of a record lock on the index record and a gap lock on the gap before the index record. (a next-key lock is an index-record lock plus a gap lock on the gap preceding the index record. If one session has a shared or exclusive lock on record R in an index, another session cannot insert a new index record in the gap immediately before R in the index order.)

### Locking Reads
This is something I would like to highlight, In InnoDB allows you to specify the lock type, whether you want to set exclusive lock, which means no one can access it or a shared lock, which means people can still read it but cannot change it at all. Most people only know the first type.

- SELECT ... LOCK IN SHARE MODE: sets a shared mode lock on any rows that are read. Other sessions can read the rows, but cannot modify them until your transaction commits.
- SELECT ... FOR UPDATE:  locks the rows and any associated index entries, the same as if you issued a UPDATE statement for those rows. Other transactions are blocked from updating those rows, from doing SELECT ... LOCK IN SHARE MODE, or from reading the data in certain transaction isolation levels

## Locks Set by Different SQL Statements in InnoDB

A locking read, a UPDATE, or a DELETE generally set record locks on **every index record that is scanned in the processing of the SQL statement**. It does not matter whether there are WHERE conditions in the statement that would exclude the row. InnoDB does not remember the exact WHERE condition, but only knows which index ranges were scanned. 

If a secondary index is used in a search and index record locks to be set are exclusive, InnoDB also retrieves the corresponding clustered index records and sets locks on them.

For repeated read:

|Statement|Lock Table|Lock Row|
|---|---|---|
|SELECT ... FROM| IS | No Lock|
|SELECT ... FROM ... LOCK IN SHARE MODE| IS | S - Share next-key lock|
|SELECT ... FROM ... FOR UPDATE | IX |  X - exclusive lock|
|UPDATE ... WHERE ... |IX |x - exclusive next-key lock|
|DELETE FROM ... WHERE ... |IX |x - exclusive next-key lock|
|INSERT | IX |x - exclusive lock on the inserted row. This lock is an index-record lock, not a next-key lock (**it tries to get a share gap lock first**)|

If a FOREIGN KEY constraint is defined on a table, any insert, update, or delete that requires the constraint condition to be checked sets shared record-level locks on the records that it looks at to check the constraint. InnoDB also sets these locks in the case where the constraint fails.

LOCK TABLES sets table locks, but it is the higher MySQL layer above the InnoDB layer that sets these locks.

## Next Step
Now let's cover the unexpected behavior which will surprise you! 
[InnoBD(MySQL) Lock Gotchas](/blog/2016/innodbmysql-lock-gotchas)

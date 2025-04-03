# db-from-scratch
This is a repo of my notes going through https://build-your-own.org/database/.

Some of it is me just typing out things in the book, so go read the book! 

## What it promises one will learn
Complex systems like databases are built on a few simple principles.

Atomicity & durability. A DB is more than just files!
Persist data with fsync.
Crash recovery.
KV store based on B-tree.
Disk-based data structures.
Space management with a free list.
Relational DB on top of KV.
How tables and indexes are mapped to low-level B-trees.
SQL-like query language; parser & interpreter.
Concurrency control for transactions.
Code a database in 3000 LoC, incrementally


## Topic 1: durability + atomicity 

Persisting data to disk is the #1 criteria of a traditional db. Why not use files if the database is just a file anyway? 

`Persisting` data means that at some point, the data added to the DB is guarenteed to persist even if the machine crashes. 

Most databases run on filesystems. So filesystems are somewhat similar to databases. However, the big difference is that typical filesystem use (writing to files) has no durability guarantee, resulting in data loss or corruption after a power loss, while typical database use guarantees durability.


The `fsync` syscall allows us to request and confirm durability from the filesystem. 

The DB will only return success to the client after the fsync call has confirmed durability. 

Atomicity is important here, because we can't have a partially durable operation. 
It has to have an all or nothing semantic.


## Topic 2: Indexing Datastructures 


### OLAP vs OLTP 
- Olap can invlove large amounts of data with aggregations and joins. Indexing can be limited or non-existent. They are mostly column based data stores. Used to execute ad-hoc analytical queries that are not sensitive to latency.
    - Snowflake
    - ClickHouse
    - Google BigQuery
- OLTP touches small amounts of data using indexes. Low latency and cost. Based on eitehr B+Trees, or LSM-trees. They are used to execute pre-programed user-facing  queries immediately or with low latency. 
    - MYSQL
    - Postgres 
    - SQLite

This project will be building out an OLTP database. 


### In memory vs on disk data structures 

When indexing data, you run into challenges if that data is stored on disk. 

Ram and disk have different properties, especially latency. When studying these properties, we are left with two types of trees. A B+Tree and a LSM Tree. Databases are limited to a few data structures. 

| Medium | Latency | 
--
| RAM | 50~100ns | 
| SSD | 50000-100000 ns| 
| HDD | 500000-100000000 ns| 
--



Indexing is one challenge, but then you need to persist data to disk. A database is just a data structure on disk. 

The final challenge is concurrency, as for in memory data its ok to serialize the data strcutre access with a mutex, but for disk based data, the io latency makes this impractical and requires more advanced concurrency management.


## Topic 3: Relational DB on KV 

SQL is a user interface, and not fundemental to a db. Relational DBs are built on top of KV-Like interfaces called `storage engines`. 

```go
type KV interface {
   // get, set, del
   Get(key []byte) (val []byte, ok bool)
   Set(key []byte, val []byte)
   Del(key []byte)
   // range query
   FindGreaterThan(key []byte) Iterator
   // ...
}
type Iterator interface {
   HasNext() bool
   Next() (key []byte, val []byte)
}
```

Then finally, you add query language or interpreter, to query your data structure.

---
layout: post
title: "Why PostgreSQL?"
date: 2010-10-14 22:49
comments: true
categories: 
 - Database
 - Development
 - Postgres
 - MySQL
---
Quick overview of what PostgreSQL brings to the table that is not available in MySQL.
<!-- more -->
* Uses MVCC for all tables providing:
	* Fully transactional including ACID compliance for consistency
    * Nested transactions
* SQL 2008 compliant
* Foreign keys for any table
* Advanced table partitioning
* Highly sophisticated query planner/optimizer
	* Can split up a query for execution across multiple CPUs simultaneously
	* Collects internal statistics for adaptive query planning
	* Special genetic query optimizer for queries with large numbers of joins
	* Supports multiple indexes per table per query
* Advanced support for query & results caching
* Hot/online backup
* Point-in-time-recovery
* Write-ahead logs for fault-tolerance
* Tablespaces for controlling physical disk layout
* Native asynchronous replication guaranteeing identical results on all machines. Supports both:
	* Streaming replication
	* Hot standby
* Partial indexes
* Index creation/removal does not lock table
* Full support for constraints
* Transactional DDL - changes like table modifications can placed inside a transaction and rolled back

Specific disadvantages to MySQL:

* Confusion with table types - MyISAM vs InnoDB
* Designed to scale out not up - does not utilize larger numbers of cores efficiently and cannot spread queries across cores
* Hot backup of is difficult for databases containing both InnoDB and MyISAM
* Replication is mediocre and error prone
* InnoDB stores the data with the primary key, so any queries using secondary indices are slower
* Subqueries not well optimized
* Only uses a single index per table per query
* Index creation/removal requires an exclusive write lock
* MyISAM only offers table level locking which causes severe performance degradation under heavy concurrency
* Limited support for constraints
* No transactional DDL - changes like table modifications are automatically committed and cannot be rolled back

MySQL offers the following advantages over PostgreSQL:

* MyISAM tables can offer better read performance, specifically for simple SELECT queries, but at the cost of no support for transactions, foreign keys or data guarantees
* COUNT(*) on MyISAM is very fast and slow on PostgreSQL
* INSERT IGNORE and INSERT...ON DUPLICATE UPDATE
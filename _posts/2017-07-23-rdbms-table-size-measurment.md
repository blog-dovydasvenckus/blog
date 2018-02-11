---
layout: post
title:  "Finding biggest tables in MySQL and PostgreSQL"
description: How to find biggest tables in MySQL and PostgreSQL relational database systems.
date:   2017-07-23 19:02:59 +0300
categories: rdbms
image: /assets/images/common/thumbnails/rdbms.png
---

In relational database systems, we store large amounts of data.
Ability to find biggest tables in a database is quite useful when you are searching
where your precious disk space is being used.

In this blog post, I'll show how to list biggest tables in MariaDB and PostgreSQL.

## MariaDB/MySQL
In MySQL and MariaDB information about table size is stored in table **TABLES**
which resides in **information_schema**.

{% highlight sql %}

SELECT
  TABLE_NAME AS 'Table',
  ROUND(DATA_LENGTH / 1024) AS 'Table Size (KB)',
  ROUND(INDEX_LENGTH / 1024) AS 'Index Size (KB)',
  ROUND(DATA_FREE / 1024) AS 'Data free (KB)',
  ROUND((DATA_LENGTH + INDEX_LENGTH + DATA_FREE) / 1024) AS 'Total Size (KB)'
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'your_schema_name'
ORDER BY (DATA_LENGTH + INDEX_LENGTH) DESC
LIMIT 10;
{% endhighlight %}

Before running this query don't forget to set **TABLE_SCHEMA** to your database name.

When you are querying table size you should analyze these 3 columns:
* DATA_LENGTH - this field shows how much space is occupied by data + [clustered index](https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html).
* INDEX_LENGTH - this field shows how much of space is used for [secondary indexes](https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html).
* DATA_FREE - shows allocated, but not used pages.

Example of query result:

    +---------+-----------------+-----------------+----------------+-----------------+
    | Table   | Table Size (KB) | Index Size (KB) | Data free (KB) | Total Size (KB) |
    +---------+-----------------+-----------------+----------------+-----------------+
    | article |           23056 |               0 |           4096 |           27152 |
    | client  |             176 |              64 |              0 |             240 |
    | item    |             112 |               0 |              0 |             112 |
    +---------+-----------------+-----------------+----------------+-----------------+




## PostgreSQL
In PostgreSQL required data is stored in **pg_class** table.

{% highlight sql %}
SELECT table_name,
    table_bytes / 1024 AS TABLE_KB,
    index_bytes / 1024 AS INDEX_KB,
    toast_bytes / 1024 AS TOAST_KB,
    total_bytes / 1024 AS TOTAL_KB
  FROM (
  SELECT *,
      total_bytes - index_bytes - toast_bytes
      AS table_bytes FROM (
      SELECT c.oid,nspname AS table_schema, relname AS TABLE_NAME,
          pg_total_relation_size(c.oid) AS total_bytes,
          pg_indexes_size(c.oid) AS index_bytes,
          COALESCE(pg_total_relation_size(reltoastrelid), 0) AS toast_bytes
          FROM pg_class c
          LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
          WHERE relkind = 'r' AND nspname='public'
  ) a
) a
ORDER BY total_bytes DESC
LIMIT 10;
{% endhighlight %}

When you are using this query, you should take into account these fields:
* TABLE_KB - this field shows how much of space is occupied by data.
* INDEX_KB - space used by index. Unlike MySQL, it includes space used by clustered indexes.
* TOAST_KB - Space used for TOAST. PostgreSQL stores long values such as strings and byte arrays in.
separate TOAST tables. This technique is called [TOAST](https://www.postgresql.org/docs/9.6/static/storage-toast.html).
* TOTAL_KB - this field shows the total space used for a table.


Example of this query result:

    table_name | table_kb | index_kb | toast_kb | total_kb
    -----------+----------+----------+----------+----------
    article    |      328 |       40 |     5352 |     5720
    client     |      176 |       96 |        8 |      280
    item       |      128 |       40 |        0 |      168


## Final thoughts
These queries were quite useful for my personal usage. I hope it will bring some
value to you.

For this article, I have used KB as measurement units. If you want to use different
measurement unit, feel free to modify these queries.

If you have some questions or you know a better way, please leave a comment.

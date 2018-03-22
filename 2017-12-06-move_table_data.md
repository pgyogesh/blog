---
layout: post
title:  "Moving table data from one table to another"
date:   2017-12-06 01:30:13 +0800
categories: SQL
read: 5 minutes read
tags: postgresql greenplum SQL
---


Yeah, we can move table rows from one table to another table. Here is how:

```sql
WITH deleted_rows AS (
DELETE FROM source_table WHERE id = 1
RETURNING *
)
INSERT INTO destination_table
SELECT * FROM deleted_rows;
```

## Example
+ #### Create two tables

```sql
SELECT * FROM test1 ;
 id |  name
----+--------
  1 | yogesh
  2 | Raunak
  3 | Varun
(3 rows)

SELECT * FROM test2;
 id | name
----+------
(0 rows)
```

+ #### Move rows from table test1 to table test2


```sql
WITH deleted_rows AS (
  DELETE FROM test1 WHERE id = 1
  RETURNING *
  )
  INSERT INTO test2
  SELECT * FROM deleted_rows;
INSERT 0 1
```

+ #### Verify

```sql
SELECT * FROM test2;
 id |  name
----+--------
  1 | yogesh
(1 row)

select * from test1;
 id |  name
----+--------
  2 | Raunak
  3 | Varun
 ```

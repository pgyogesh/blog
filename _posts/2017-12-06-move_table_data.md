---
layout: single
comments: true
excerpt: "A simple trick to move table data from one table to other table"
header:
  overlay_image: https://source.unsplash.com/random/1200x400?nature
  overlay_filter: 0
title:  "Moving table data from one table to another"
date:   2017-12-06 01:30:13 +0800
categories: SQL
read: 5 minutes read
tags: postgresql greenplum SQL
---


Yeah, we can move table rows from one table to another table. Here is how:

{% highlight sql linenos %}
WITH deleted_rows AS (
DELETE FROM source_table WHERE id = 1
RETURNING *
)
INSERT INTO destination_table
SELECT * FROM deleted_rows;
{% endhighlight %}


## Example
### Create two tables

{% highlight sql linenos %}
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
{% endhighlight %}


### Move rows from table test1 to table test2


{% highlight sql linenos %}
WITH deleted_rows AS (
  DELETE FROM test1 WHERE id = 1
  RETURNING *
  )
  INSERT INTO test2
  SELECT * FROM deleted_rows;
INSERT 0 1
{% endhighlight %}


### Verify

{% highlight sql linenos %}
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
{% endhighlight %}

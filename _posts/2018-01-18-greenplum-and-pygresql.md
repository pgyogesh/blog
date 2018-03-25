---
layout: single
comments: true
excerpt: "Give a basic idea of using pygresql Python module with Greenplum and PostgreSQL"
header:
  overlay_image: https://source.unsplash.com/random
  overlay_filter: 0.5
title:  "Using pygresql python module with Greenplum"
date:   2018-01-18 01:30:13 +0800
categories: Python
read: 3 minutes read
tags: github greenplum postgresql
---

Python is my favourite progamming langauge and Greenplum Database is my key skill. So, Using both to perform some task is no surprise. At first I tried to use [_psycopg2_](http://initd.org/psycopg/) but When I started looking in [Greenplum source code](https://github.com/greenplum-db/gpdb) I found that Pivotal is using [pygresql](http://www.pygresql.org). So, I too thought using the same.

##### So, What is pygresql?
Pygresql is Python interface for PostgreSQL. It allows PostgreSQL queries to run through Python script.

##### Importing pygresql module in Python program.

```python
from pygresql.pg import DB
```

##### Creating connection to database

```python
con = DB(dbname='gpadmin', host='localhost', port=5432, user='gpadmin', passwd='changeme')
```

##### Running Queries

```python
>>>a = con.query("select * from gp_segment_configuration where hostname=mdw")
>>>print(a) #This will print the result as same as psql utility
dbid|content|role|preferred_role|mode|status|port|hostname       |address|replication_port|san_mounts
----+-------+----+--------------+----+------+----+---------------+-------+----------------+----------
1   |-1     |p   |p             |s   |u     |5432|mdw            |mdw    |                |


>>> a.getresult() #To get result as Python tuples
[(1, -1, 'p', 'p', 's', 'u', 5432, 'mdw', 'mdw', None, None)]

>>> a.dictresult() #To get result as Python Dictionary
[{'status': 'u', 'replication_port': None, 'dbid': 1, 'hostname': 'mdw', 'preferred_role': 'p', 'content': -1, 'role': 'p', 'mode': 's', 'address': 'mdw', 'san_mounts': None, 'port': 5432}]
```



##### Closing the connection

```python
con.close()
```

Here is complete [tutorial](http://www.pygresql.org/contents/tutorial.html) for pygresql.

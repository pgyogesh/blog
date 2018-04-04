---
layout: single
comments: true
excerpt: "List of SQL statements which can be used by Database Administrators in day to day life"
header:
  image: https://source.unsplash.com/random/1200x400?nature
  overlay_filter: 0
title:  "Useful SQL scripts for Greenplum DBA"
date:   2017-12-06 01:30:13 +0800
categories: Greenplum
tags: postgresql greenplum SQL scripts
---

Here I'm sharing my collection of SQL Script for Greenplum DBA. And of course few of them are copied from somewhere on Internet :wink:

## User, roles and resource queue

### List of member of a role

{% highlight sql linenos %}
SELECT a.rolname
FROM pg_roles a
WHERE pg_has_role(a.oid,'your_rolname', 'member');
{% endhighlight %}


### List of roles and its members

{% highlight sql linenos %}
SELECT r.rolname,
	ARRAY(SELECT b.rolname
	FROM pg_catalog.pg_auth_members m
	JOIN pg_catalog.pg_roles b ON (m.roleid = b.oid)
	WHERE m.member = r.oid) as memberof
FROM pg_catalog.pg_roles r
WHERE r.rolname !~ '^pg_'
ORDER BY 1;
{% endhighlight %}


### Resource Queue of user

{% highlight sql linenos %}
SELECT *
	FROM gp_toolkit.gp_resq_role
	where rrrolname = 'rolename';
{% endhighlight %}


### Running queries or statements which are waiting in Resource Queues

{% highlight sql linenos %}
SELECT
	rolname
	,rsqname
	,pid
	,granted
	,current_query
	,datname
FROM pg_roles, gp_toolkit.gp_resqueue_status
,pg_locks, pg_stat_activity
WHERE pg_roles.rolresqueue=pg_locks.objid
	AND pg_locks.objid=gp_toolkit.gp_resqueue_status.queueid
	AND pg_stat_activity.procpid=pg_locks.pid
	AND pg_stat_activity.usename=pg_roles.rolname;
{% endhighlight %}



### List of users associated with Resource Queue

{% highlight sql linenos %}
SELECT rolname as RoleName
,case
	when rolsuper = 't' then
		'SUPERUSER'
	else
		'NORMAL'
	end as SuperUser
,case
	when rolcanlogin = 't' then
		'LOGIN'
	else
		'GROUP'
end as RoleType
,rsqname as RQName
FROM pg_roles, gp_toolkit.gp_resqueue_status
WHERE pg_roles.rolresqueue=gp_toolkit.gp_resqueue_status.queueid
order by rolname;
{% endhighlight %}


## Object Size and Workfiles

### Uncompressed size of table

{% highlight sql linenos %}
SELECT
	pg_size_pretty(SUM(sotusize)::BIGINT)
FROM gp_toolkit.gp_size_of_table_uncompressed where sotuschemaname = 'schema_name'  and sotutablename ='table_name';
{% endhighlight %}


### Uncompressed size of schema

{% highlight sql linenos %}
SELECT pg_size_pretty(SUM(sotusize)::BIGINT)
FROM gp_toolkit.gp_size_of_table_uncompressed
WHERE sotuschemaname = '<schema_name>';
{% endhighlight %}


### Uncompressed size of current database

{% highlight sql linenos %}
SELECT
	pg_size_pretty(SUM(sotusize)::BIGINT)
FROM gp_toolkit.gp_size_of_table_uncompressed;
{% endhighlight %}


### Top big tables in schema with owner name

{% highlight sql linenos %}
SELECT
	a.sotuschemaname as Schema,
	a.sotutablename as Table,
	pg_size_pretty(a.sotusize::BIGINT) as Size,
	b.tableowner
FROM gp_toolkit.gp_size_of_table_uncompressed a
JOIN pg_tables b ON (a.sotutablename = b.tablename)
WHERE a.sotuschemaname = 'public'
ORDER BY sotusize DESC
LIMIT 50;
{% endhighlight %}


### Workfiles per query

{% highlight sql linenos %}
SELECT
	g.datname "Database",
	g.procpid "Process",
	g.sess_id "Session",
	p.usename "User",
	SUBSTR(p.current_query,0,60) "Query",
	sum(g.size)/1024/1024::float "Total Spill Size(MB)",
	sum(g.numfiles) "Total Spill Files"
FROM gp_toolkit.gp_workfile_usage_per_query g
JOIN pg_stat_activity p on g.sess_id = p.sess_id
GROUP BY 1,2,3,4,p.current_query
ORDER BY 4 DESC;
{% endhighlight %}


### Work files details on each segment

{% highlight sql linenos %}
SELECT
				gwe.datname as DatabaseName
				,psa.usename as UserName
				,gwe.procpid as ProcessID
				,gwe.sess_id as SessionID
				,sc.hostname as HostName
				,sum(size)/1024::float as SizePerHost
				,sum(numfiles) NumOfFilesPerHost
FROM  gp_toolkit.gp_workfile_entries as gwe
inner join pg_stat_activity as psa
				on psa.procpid = gwe.procpid
				and psa.sess_id = gwe.sess_id,
gp_segment_configuration as sc
,pg_filespace_entry as fe
,pg_database as d
WHERE fe.fsedbid=sc.dbid
				AND gwe.segid=sc.content
				AND gwe.datname=d.datname
				AND sc.role='p'
group by
				gwe.datname
				,psa.usename
				,gwe.procpid
				,gwe.sess_id
				,sc.hostname
ORDER BY
				gwe.datname
				,psa.usename
				,gwe.procpid
				,gwe.sess_id
				,sc.hostname;
{% endhighlight %}


## Database Activities and Locks

### Database Activities

{% highlight sql linenos %}
SELECT
datname as Database,
procpid as Process_ID,
sess_id as Session_ID,
usename as Username,
SUBSTR(current_query,0,60) as Current_Query,
now() - query_start as Query_Duration,
now() - backend_start as Session_Duration,
waiting as Is_Waiting
FROM pg_stat_activity
WHERE current_query NOT ilike '%IDLE%'
ORDER BY 6 desc;
{% endhighlight %}


### Waiter's Information

{% highlight sql linenos %}
SELECT
	l.locktype                    AS  "Waiters locktype",
	d.datname                     AS  "Database",
	l.relation::regclass          AS  "Waiting Table",
	a.usename                     AS  "Waiting user",
	l.pid                         AS  "Waiters pid",
	l.mppsessionid                AS  "Waiters SessionID",
	l.mode                        AS  "Waiters lockmode",
	now()-a.query_start           AS  "Waiting duration",
	SUBSTR(current_query,0,60)    AS  "Waiters Query"
FROM
	pg_locks l,
	pg_stat_activity a,
	pg_database d
WHERE l.pid=a.procpid
AND l.database=d.oid
AND l.granted = 'f'
ORDER BY 3;
{% endhighlight %}


### Blocker's Information

{% highlight sql linenos %}
SELECT
	l.locktype                      AS  "Blocker locktype",
	d.datname                       AS  "Database",
	l.relation::regclass            AS  "Blocking Table",
	a.usename                       AS  "Blocking user",
	l.pid                           AS  "Blocker pid",
	l.mppsessionid                  AS  "Blockers SessionID",
	l.mode                          AS  "Blockers lockmode",
	now()-a.query_start             AS  "Blocked duration",
	SUBSTR(current_query,0,60)      AS  "Blocker Query"
FROM
	pg_locks l,
	pg_stat_activity a,
	pg_database d
WHERE l.pid=a.procpid
AND l.database=d.oid
AND l.granted = true
AND relation in ( select relation from pg_locks where granted='f')
ORDER BY 3;
{% endhighlight %}


### Waiter's and Blocker's Information

{% highlight sql linenos %}
SELECT
	kl.pid as blocking_pid,
	ka.usename as blocking_user,
	SUBSTR(ka.current_query,0,20) as blocking_query,
	bl.pid as blocked_pid,
	a.usename as blocked_user,
	SUBSTR(a.current_query,0,20) as blocked_query,
	to_char(age(now(), a.query_start),'HH24h:MIm:SSs') as age
FROM pg_catalog.pg_locks bl
	JOIN pg_catalog.pg_stat_activity a
		ON bl.pid = a.procpid
	JOIN pg_catalog.pg_locks kl
		ON bl.locktype = kl.locktype
		and bl.database is not distinct from kl.database
		and bl.relation is not distinct from kl.relation
		and bl.page is not distinct from kl.page
		and bl.tuple is not distinct from kl.tuple
		--and bl.virtualxid is not distinct from kl.virtualxid
		and bl.transactionid is not distinct from kl.transactionid
		and bl.classid is not distinct from kl.classid
		and bl.objid is not distinct from kl.objid
		and bl.objsubid is not distinct from kl.objsubid
		and bl.pid <> kl.pid
	JOIN pg_catalog.pg_stat_activity ka
		ON kl.pid = ka.procpid
WHERE kl.granted and not bl.granted
ORDER BY a.query_start;
{% endhighlight %}

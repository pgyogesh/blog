---
layout: single
comments: true
excerpt: "Give a basic idea for creating a log analyzer using Python Programming"
header:
  overlay_image: https://source.unsplash.com/random
  overlay_filter: 0.5
title:  "How to check on which segment backup was running longer (ddboost)"
date:   2017-11-24 01:30:13 +0800
categories: Greenplum
tags: greenplum backup ddboost
---

Yesterday, In one of our QA environment backup was running for more than 5 hours which usually completes in 1 hours. And as backup was completed we were unable to troubleshoot on why it was running longer.

Then I checked log files for few segments on found that below query runs on every segment instance at the end of backup job.

```sql
SELECT * FROM gp_read_backup_file('/backup/storage/directory', '20171123010034', 1)
```

Then I used `grep` utility to find query pattern using `gpssh` utility like below:

```bash
[mdw] gpssh -f /home/gpadmin/hostfile
=> grep -i gp_read_backup_file /data?/primary/gpseg??/pg_log/gpdb-2017-11-23_000000.csv
```

And I found that on gpseg76(sdw10) the query above mentioned is ran at `06:32` while it is completed around `1:40` in other segment instances.

### Snippet from grep result

```bash
=> [sdw13] /data1/primary/gpseg96/pg_log/gpdb-2017-11-23_000000.csv:2017-11-23 01:45:28.991835 EST,"gpadmin","qadb",p378235,th912250656,"172.28.8.250","21300",2017-11-23 01:00:57 EST,0,con1323569,cmd35,seg-1,,,,,"LOG","00000","duration: 66.624 ms",,,,,,"SELECT * FROM gp_read_backup_file('/backup/storage/directory', '20171123010034', 1)",0,,"postgres.c",1879,
=> [sdw13] /data1/primary/gpseg97/pg_log/gpdb-2017-11-23_000000.csv:2017-11-23 01:44:03.468291 EST,"gpadmin","qadb",p378233,th1452726048,"172.28.8.250","11510",2017-11-23 01:00:57 EST,0,con1323571,cmd34,seg-1,,,,,"LOG","00000","duration: 53.644 ms",,,,,,"SELECT * FROM gp_read_backup_file('/backup/storage/directory', '20171123010034', 1)",0,,"postgres.c",1879,
=> [sdw11] /data1/primary/gpseg82/pg_log/gpdb-2017-11-23_000000.csv:2017-11-23 01:45:03.430779 EST,"gpadmin","qadb",p378953,th881481504,"172.28.8.250","38507",2017-11-23 01:00:57 EST,0,con1323568,cmd34,seg-1,,,,,"LOG","00000","duration: 24.662 ms",,,,,,"SELECT * FROM gp_read_backup_file('/backup/storage/directory', '20171123010034', 1)",0,,"postgres.c",1879,
=> [sdw11] /data1/primary/gpseg83/pg_log/gpdb-2017-11-23_000000.csv:2017-11-23 01:44:38.269630 EST,"gpadmin","qadb",p378952,th-636340448,"172.28.8.250","56455",2017-11-23 01:00:57 EST,0,con1323568,cmd34,seg-1,,,,,"LOG","00000","duration: 18.985 ms",,,,,,"SELECT * FROM gp_read_backup_file('/backup/storage/directory', '20171123010034', 1)",0,,"postgres.c",1879,
=> [ sdw2] /data2/primary/gpseg14/pg_log/gpdb-2017-11-23_000000.csv:2017-11-23 01:38:02.261105 EST,"gpadmin","qadb",p434180,th278452000,"172.28.8.250","60334",2017-11-23 01:00:57 EST,0,con1240983,cmd31,seg-1,,,,,"LOG","00000","duration: 94.540 ms",,,,,,"SELECT * FROM gp_read_backup_file('/backup/storage/directory', '20171123010034', 1)",0,,"postgres.c",1879,
=> [ sdw2] /data2/primary/gpseg15/pg_log/gpdb-2017-11-23_000000.csv:2017-11-23 01:37:48.169687 EST,"gpadmin","qadb",p434178,th1485887264,"172.28.8.250","35144",2017-11-23 01:00:57 EST,0,con1240983,cmd31,seg-1,,,,,"LOG","00000","duration: 19.472 ms",,,,,,"SELECT * FROM gp_read_backup_file('/backup/storage/directory', '20171123010034', 1)",0,,"postgres.c",1879,
=> [ sdw8] /data1/primary/gpseg57/pg_log/gpdb-2017-11-23_000000.csv:2017-11-23 01:37:11.926575 EST,"gpadmin","qadb",p347557,th1154320160,"172.28.8.250","18289",2017-11-23 01:00:57 EST,0,con1056456,cmd31,seg-1,,,,,"LOG","00000","duration: 13.913 ms",,,,,,"SELECT * FROM gp_read_backup_file('/backup/storage/directory', '20171123010034', 1)",0,,"postgres.c",1879,
=> [ sdw8] /data1/primary/gpseg58/pg_log/gpdb-2017-11-23_000000.csv:2017-11-23 01:37:07.201256 EST,"gpadmin","qadb",p347555,th119101216,"172.28.8.250","35081",2017-11-23 01:00:57 EST,0,con1056456,cmd31,seg-1,,,,,"LOG","00000","duration: 21.343 ms",,,,,,"SELECT * FROM gp_read_backup_file('/backup/storage/directory', '20171123010034', 1)",0,,"postgres.c",1879,

=> [sdw10] /data2/primary/gpseg76/pg_log/gpdb-2017-11-23_000000.csv:2017-11-23 06:32:37.919721 EST,"gpadmin","qadb",p383258,th-266610912,"172.28.8.250","63198",2017-11-23 01:00:57 EST,0,con1323570,cmd178,seg-1,,,,,"LOG","00000","duration: 22.714 ms",,,,,,"SELECT * FROM gp_read_backup_file('/backup/storage/directory', '20171123010034', 1)",0,,"postgres.c",1879,

```

Then we concentrated our troubleshooting on sdw10 and found few hardware related issues. :smile:

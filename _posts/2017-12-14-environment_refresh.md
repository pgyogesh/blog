---
layout: single
comments: true
excerpt: "Python program to switch the roles between the environments"
header:
  overlay_image: https://source.unsplash.com/random/1200x400?nature
  overlay_filter: 0
title:  "How do we do Greenplum Database environment refresh"
date:   2017-12-14 01:30:13 +0800
categories: Greenplum
read: 5 minutes read
tags: greenplum scripts python SQL
---

For a Database Administrator, Data refresh between the environments is regular task. For example, Copying data from production environment to UAT environment, UAT environment to production environment.

So, for example, we have requirement to refresh UAT data from production environment, We simple take backup from production and restore the same in UAT environment. Simple right?

No, But changing the object permissions are big task here.

We are following below pattern for group role names

`<client-name>_<environment>_<permission>_rl`

For example, Say client name is yj, environment is prod and permissions are rw (i.e. Read and write). So the role name will be `yj_prod_rw_rl`

While taking the data backup from production environment, We also take `--schema-only` backup using _pg\_dump_ utility. So that we can extract GRANT, REVOKE, ALTER TABLE ..  OWNER TO statements from dump file.

And I have created a python script to rename the roles from dump file and generates GRANT, REVOKE and ALTER TABLE ..  OWNER TO statements with appropriate roles.

## Script

{% highlight python linenos %}
import optparse
import os
import sys
import re
import logging
from shutil import copyfile
parser = optparse.OptionParser()
logging.basicConfig(format='%(asctime)s:%(levelname)s:%(message)s',level=logging.DEBUG)
parser.add_option("-f","--file",dest="sql_file",
                  action="store",help="SQL dump file")
parser.add_option("--to_env",dest="to_env",
                  action="store",help="Environment to refresh")
parser.add_option("--from_env",dest="from_env",
                  action="store",help="Environment to refresh from")
parser.add_option("-s","--schema",dest="schema",
                  action="store",help="Schema name, This is just to set search_path in SQL file")

options, args = parser.parse_args()

#
# Checking if temp files already exists
#

temp_files = ['/tmp/grantfile.sql','/tmp/grantfile_temp.sql','/tmp/revokefile.sql','/tmp/revokefile_temp.sql','/tmp/ownerfile.sql','/tmp/ownerfile_temp.sql']
logging.info("Checking if temp files already exists")


for file in temp_files:
    if os.path.isfile(file):
        logging.error("%s file already exists! Exiting...!" %file)
        sys.exit()

v_sqlfile=open(options.sql_file)

#
# Final result will be in "/tmp/grantfile.sql"
#

logging.info("Fetching grant SQL statement from " + options.sql_file)
v_grantfile=open("/tmp/grantfile.sql","a")
for g_line in v_sqlfile:
    g_line = g_line.rstrip()
    if re.search('^GRANT',g_line):
       v_grantfile.writelines(g_line)
       v_grantfile.write('\n')
v_grantfile.close()
v_sqlfile.close()

#
# Final Result will be in "/tmp/ownerfile.sql"
#
v_sqlfile=open(options.sql_file)

logging.info("Fetching 'ALTER TABLE .. OWNER TO' SQL statement from " + options.sql_file)
v_ownerfile=open("/tmp/ownerfile.sql","a")
for o_line in v_sqlfile:
    o_line = o_line.rstrip()
    if re.search("OWNER TO",o_line):
       v_ownerfile.writelines(o_line)
       v_ownerfile.write('\n')
v_ownerfile.close()

#
# Final result will in "/tmp/revokefile_temp.sql"
#

logging.info("Generating revoke statements from " + options.sql_file)
v_grantfile=open("/tmp/grantfile.sql","r")
#copyfile("/tmp/grantfile.sql","/tmp/revokefile.sql")
v_revokefile=open("/tmp/revokefile.sql","a")
#v_revokefile_temp=open("/tmp/revokefile_temp.sql","a")
for r_line in v_grantfile:
    revoke_line = r_line.replace("GRANT","REVOKE")
    from_line = revoke_line.replace(" TO ", " FROM ")
    v_revokefile.writelines(from_line)
v_revokefile.close()
v_grantfile.close()
#
# Final Result will be in "/tmp/grantfile_temp.sql"
#

logging.info("Creating new GRANT statement's file with "+ options.to_env + " roles")
v_grantfile=open("/tmp/grantfile.sql","r")
v_grantfile_temp=open("/tmp/grantfile_temp.sql","a")
for r_line in v_grantfile:
    new_role_line = r_line.replace('_' + options.from_env + '_', '_' + options.to_env + '_')
    v_grantfile_temp.writelines(new_role_line)
    #v_grantfile_temp.write('\n')

v_grantfile.close()
v_grantfile_temp.close()


#
# Final result will be in "/tmp/ownerfile_temp.sql"
#

logging.info("Creating new 'ALTER TABLE .. OWNER TO' statement file with "+ options.to_env + " roles")
v_ownerfile=open("/tmp/ownerfile.sql","r")
v_ownerfile_temp=open("/tmp/ownerfile_temp.sql","a")
for o_line in v_ownerfile:
    new_role_line = o_line.replace('_' + options.from_env + '_', '_' + options.to_env + '_')
    v_ownerfile_temp.writelines(new_role_line)
    #v_ownerfile_temp.write('\n')

v_ownerfile.close()
v_ownerfile_temp.close()
#
# Gathering all statements into one SQL file
#

logging.info("Gathering all statements in one SQL file")
final_sql_file= options.to_env +"_refresh.sql"
revoke_sql=open("/tmp/revokefile.sql","r")
grant_sql=open("/tmp/grantfile_temp.sql","r")
owner_sql=open("/tmp/ownerfile_temp.sql","r")
sql_file=open(final_sql_file,"a+")

sql_file.write("set search_path to " + options.schema + ';')
sql_file.write("--------------------REVOKE STATEMENTS---------------------\n\n")
for line in revoke_sql:
    sql_file.writelines(line)

sql_file.write("\n\n--------------------GRANT STATEMENTS---------------------\n\n")
for line in grant_sql:
    sql_file.writelines(line)

sql_file.write("\n\n--------------------OWNER STATEMENTS---------------------\n\n")
for line in owner_sql:
    sql_file.writelines(line)

revoke_sql.close()
grant_sql.close()
owner_sql.close()

#
# Deleting temp files
#

logging.info("Deleting temporary files")
if os.path.isfile('/tmp/revokefile.sql'):
   os.remove('/tmp/revokefile.sql')
if os.path.isfile('/tmp/grantfile.sql'):
   os.remove('/tmp/grantfile.sql')
if os.path.isfile('/tmp/ownerfile.sql'):
   os.remove('/tmp/ownerfile.sql')
if os.path.isfile('/tmp/revokefile_temp.sql'):
   os.remove('/tmp/revokefile_temp.sql')
if os.path.isfile('/tmp/grantfile_temp.sql'):
   os.remove('/tmp/grantfile_temp.sql')
if os.path.isfile('/tmp/ownerfile_temp.sql'):
   os.remove('/tmp/ownerfile_temp.sql')
logging.info("SQL file created: " + final_sql_file)
{% endhighlight %}


## Script help:

{% highlight bash linenos %}
$ python env_refresh.py --help
Usage: env_refresh.py [options]

Options:
  -h, --help                      show this help message and exit
  -f SQL_FILE, --file=SQL_FILE
                                  SQL dump file
  --to_env=TO_ENV                 Environment to refresh
  --from_env=FROM_ENV             Environment to refresh from
  -s SCHEMA, --schema=SCHEMA      Schema name, This is just to set search_path in SQL file
{% endhighlight %}

## Sample dump file

{% highlight sql linenos %}
  CREATE TABLE tbl_movement (
    movementid bigint NOT NULL,
    fileid integer NOT NULL,
    clientid character varying(10),
    branchid character varying(6)
  ) DISTRIBUTED BY (movementid);

  CREATE SEQUENCE tbl_movement_movementid_seq
    INCREMENT BY 1
    NO MAXVALUE
    NO MINVALUE
    CACHE 1;

ALTER TABLE taxppcore.tbl_movement_movementid_seq OWNER TO yj_create_prod_rl;
ALTER SEQUENCE tbl_movement_movementid_seq OWNED BY tbl_movement.movementid;
ALTER TABLE tbl_movement ALTER COLUMN movementid SET DEFAULT nextval('tbl_movement_movementid_seq'::regclass);
REVOKE ALL ON TABLE tbl_movement FROM PUBLIC;
REVOKE ALL ON TABLE tbl_movement FROM yj_create_prod_rl;
GRANT ALL ON TABLE tbl_movement TO yj_create_prod_rl;
GRANT SELECT,INSERT,DELETE,UPDATE ON TABLE tbl_movement TO yj_wr_prod_rl;
GRANT SELECT ON TABLE tbl_movement TO yj_ro_prod_rl;
GRANT SELECT,INSERT,DELETE,UPDATE ON TABLE tbl_movement TO yj_wr_prod_rl;
GRANT SELECT ON TABLE tbl_movement TO yj_ro_prod_rl;
REVOKE ALL ON SEQUENCE tbl_movement_movementid_seq FROM PUBLIC;
REVOKE ALL ON SEQUENCE tbl_movement_movementid_seq FROM yj_create_prod_rl;
GRANT ALL ON SEQUENCE tbl_movement_movementid_seq TO yj_create_prod_rl;
GRANT SELECT ON SEQUENCE tbl_movement_movementid_seq TO yj_ro_prod_rl;
{% endhighlight %}


## Demo:

{% highlight bash linenos %}
$ python env_refresh.py -f tbl_filesinfo.sql --to_env uat --from_env prod --schema yj
2017-12-14 02:20:14,293:INFO:Checking if temp files already exists
2017-12-14 02:20:14,296:INFO:Fetching grant SQL statement from tbl_filesinfo.sql
2017-12-14 02:20:14,297:INFO:Fetching 'ALTER TABLE .. OWNER TO' SQL statement from tbl_filesinfo.sql
2017-12-14 02:20:14,298:INFO:Generating revoke statements from tbl_filesinfo.sql
2017-12-14 02:20:14,298:INFO:Creating new GRANT statements file with uat roles
2017-12-14 02:20:14,298:INFO:Creating new 'ALTER TABLE .. OWNER TO' statement file with uat roles
2017-12-14 02:20:14,298:INFO:Gathering all statements in one SQL file
2017-12-14 02:20:14,299:INFO:Deleting temporary files
2017-12-14 02:20:14,299:INFO:SQL file created: uat_refresh.sql
{% endhighlight %}


### SQL script generated by env\_refresh.py:

{% highlight sql linenos %}
set search_path to yj;

--------------------REVOKE STATEMENTS---------------------

REVOKE ALL ON TABLE tbl_movement FROM yj_create_prod_rl;
REVOKE SELECT,INSERT,DELETE,UPDATE ON TABLE tbl_movement FROM yj_wr_prod_rl;
REVOKE SELECT ON TABLE tbl_movement FROM yj_ro_prod_rl;
REVOKE SELECT,INSERT,DELETE,UPDATE ON TABLE tbl_movement FROM yj_wr_prod_rl;
REVOKE SELECT ON TABLE tbl_movement FROM yj_ro_prod_rl;
REVOKE ALL ON SEQUENCE tbl_movement_movementid_seq FROM yj_create_prod_rl;
REVOKE SELECT ON SEQUENCE tbl_movement_movementid_seq FROM yj_ro_prod_rl;


--------------------GRANT STATEMENTS---------------------

GRANT ALL ON TABLE tbl_movement TO yj_create_uat_rl;
GRANT SELECT,INSERT,DELETE,UPDATE ON TABLE tbl_movement TO yj_wr_uat_rl;
GRANT SELECT ON TABLE tbl_movement TO yj_ro_uat_rl;
GRANT SELECT,INSERT,DELETE,UPDATE ON TABLE tbl_movement TO yj_wr_uat_rl;
GRANT SELECT ON TABLE tbl_movement TO yj_ro_uat_rl;
GRANT ALL ON SEQUENCE tbl_movement_movementid_seq TO yj_create_uat_rl;
GRANT SELECT ON SEQUENCE tbl_movement_movementid_seq TO yj_ro_uat_rl;


--------------------OWNER STATEMENTS---------------------

ALTER TABLE taxppcore.tbl_movement OWNER TO gpadmin;
ALTER TABLE taxppcore.tbl_movement_movementid_seq OWNER TO yj_create_uat_rl;

{% endhighlight %}

Notice, REVOKE statements are created for prod roles and GRANT statements are created for uat roles.
Now just run the new SQL script against UAT database to have appropriate roles for objects.

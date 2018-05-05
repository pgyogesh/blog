---
layout: single
excerpt: "Python program to run analyze on multiple tables in parallel for PostgreSQL Database."
comments: true
header:
  overlay_image: https://source.unsplash.com/random/1200x400?elephant
  overlay_filter: 0.5
title:  "Running parallel analyze on PostgreSQL Database"
date:   2018-05-06 10:10:00 +0800
categories: Postgresql Python
tags: python postgresql greenplum analyze
---

I have written a python program to run analyze parallel on postgres tables. This script will reduce lot of time compared to running analyze on database with analyze SQL command.

## Progam

{% highlight python linenos %}
from multiprocessing import Pool, Value
from pg import DB
import re
import logging
import optparse

logging.basicConfig(format='%(asctime)s:%(levelname)s:%(message)s',level=logging.DEBUG)

parser = optparse.OptionParser()
parser.add_option("-d", "--database", dest = "database", action = "store", help = "Specify target database to analyze")
parser.add_option("-n", "--schema", dest = "schema", action = "store", help = "Specify schema to analyze")
parser.add_option("-p", "--parallel-processes", dest = "parallel", action = "store",default = 1,help = "Specify number of parallel-processes")
parser.add_option("--host", dest = "host", action = "store", default = 'localhost', help = "Specify the target host")
parser.add_option("--user-tables", dest = "usertables", action = "store_true", help = "Specify if you want to analyze only user table")

options, args = parser.parse_args()

# Getting command line options

if options.database:
    vDatabase = options.database
else:
    logging.error("database not supplied... exiting...")
    sys.exit()

con = DB(dbname='template1', host=options.host)
if vDatabase in con.get_databases():
    pass
else:
    logging.error("Database doesn't exists... exiting")
    sys.exit()
con.close()

vProcesses = int(options.parallel)
vHost = options.host
vSchema = options.schema

# Function to get list of table

def get_tables():
    db = DB(dbname = vDatabase, host = vHost)
    table_list = []
    if options.usertables:
        table_list = db.get_tables()
    else:
        table_list = db.get_tables('system')
    db.close()

    if vSchema:
        tables = []
        regex = "^" + vSchema + "\."
        for table in table_list:
            if re.match(regex, table, re.I):
                tables.append(table)
    else:
        tables = table_list
    return tables

counter = Value('i', 0)
total_tables = len(get_tables())

# Function to run analyze

def run_analyze(table):
    global counter
    db = DB(dbname = vDatabase, host = vHost)
    db.query('analyze %s' %table)
    with counter.get_lock():
        counter.value += 1
    if counter.value % 10 == 0 or counter.value == total_tables:
        logging.info(str(counter.value) + " tables completed out of " + str(total_tables) + " tables")
    db.close()

# For count
def init(args):
    ''' store the counter for later use '''
    global counter
    counter = args

# Forking new #n processes for run_analyze() Function

logging.info("Running analyze on " + str(total_tables) + " tables")
pool = Pool(initializer=init, initargs=(counter, ), processes=vProcesses)
pool.map(run_analyze, get_tables())

pool.close()  # worker processes will terminate when all work already assigned has completed.
pool.join()  # to wait for the worker processes to terminate.
{% endhighlight %}

## Program help

{% highlight bash linenos %}

$ ./analyzedb --help
Usage: analyzedb [options]

Options:
  -h, --help            show this help message and exit
  -d DATABASE, --database=DATABASE
                        Specify target database to analyze
  -n SCHEMA, --schema=SCHEMA
                        Specify schema to analyze
  -p PARALLEL, --parallel-processes=PARALLEL
                        Specify number of parallel-processes
  --host=HOST           Specify the target host
  --user-tables         Specify if you want to analyze only user table
  
 {% endhighlight %}

#### `-d, --database`

This option is to specify the database to analyze. If database doesn't exists in environment, this program exists immediately.

#### `-n, --schema`

This option is to specify the schema to analyze. We can specify only one schema at a time

#### `-p, --parallel_processes`

This option is to specify the number of parallel processes to run. The default value is 1.

#### `--host`

This option is to specify the target host. The default value is `localhost`

#### `--user-tables`

Specify this option if you want to analyze on user tables and not system tables. By default this program analyzes system tables.

## Run log 1 (Complete database)

{% highlight bash linenos %}

$ ./analyzedb -d jadhavy -p 10
2018-05-06 00:23:33,680:INFO:Running analyze on 76 tables
2018-05-06 00:23:33,799:INFO:10 tables completed out of 76 tables
2018-05-06 00:23:33,844:INFO:21 tables completed out of 76 tables
2018-05-06 00:23:33,886:INFO:30 tables completed out of 76 tables
2018-05-06 00:23:33,923:INFO:40 tables completed out of 76 tables
2018-05-06 00:23:33,972:INFO:50 tables completed out of 76 tables
2018-05-06 00:23:34,018:INFO:60 tables completed out of 76 tables
2018-05-06 00:23:34,062:INFO:70 tables completed out of 76 tables
2018-05-06 00:23:34,113:INFO:76 tables completed out of 76 tables
{% endhighlight %}

## Run log 2 (Specific schema)

{% highlight bash linenos %}
$ ./analyzedb -d jadhavy -p 10 -n test
2018-05-06 00:23:52,681:INFO:Running analyze on 6 tables
2018-05-06 00:23:52,755:INFO:6 tables completed out of 6 tables
{% endhighlight %}

## Run log 3 (User tables only)

{% highlight bash linenos %}
$ ./analyzedb -d jadhavy -p 10 --user-tables
2018-05-06 00:24:25,641:INFO:Running analyze on 14 tables
2018-05-06 00:24:25,737:INFO:10 tables completed out of 14 tables
2018-05-06 00:24:25,745:INFO:14 tables completed out of 14 tables
 {% endhighlight %}

## Download binary

You can download binary of this program from [here](https://github.com/pgyogesh/postgres/blob/master/dist/analyzedb?raw=true).

If you have any suggestions or issue, let me know in comment box below :smile:



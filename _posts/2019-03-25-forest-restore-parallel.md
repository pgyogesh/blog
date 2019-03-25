---
layout: single
excerpt: "Python program to restore multiple MarkLogic Forests simultaneously. This script can be used when you have different number of forests than database forests or you have different backup forests names than database forests"
comments: true
header:
  overlay_image: https://source.unsplash.com/random/1200x400?nature,technology,city
  overlay_filter: 0.5
title:  "Restore multiple MarkLogic Forests simultaneously using Python"
date:   2019-03-25 10:10:00 +0800
categories: Marklogic Python
tags: marklogic python backup restore parallel
---

I have written a python program to restore multiple MarkLogic Forests simultaneously. This script can be used when you have different number of forests than database forests or you have different backup forests names than database forests.

## Problem

It is quite difficult when we have to perform forest level restores when you have more number of forests to restore. You have to open forest page in admin console for each forest and then perform the restore.

## Solution

This is script uses python liabraries like requests and multiprocessing to perform the restore simultaneously.

## Help

{% highlight bash %}
Yogeshs-MacBook-Air:~ yogeshjadhav$ python ml_restore.py --help
Usage: ml_restore_new.py [options]

Options:
  -h, --help            show this help message and exit
  --config-file=CONFIGFILE
                        Specify the config file
  --user=USERNAME       Specify the username
  --password=PASSWORD   Specify the user password
  --max-threads=MAXTHREADS
                        Specify maximum parallel forest restore
{% endhighlight %}

## Program:

{% highlight python linenos %} 
from multiprocessing import Pool, Value
from requests.auth import HTTPDigestAuth
import optparse
import requests
import ConfigParser
import logging

# logging
logging.basicConfig(format='%(asctime)s:%(levelname)s:%(message)s',level=logging.INFO)

# Command line argument parsing
parser = optparse.OptionParser()
parser.add_option("--config-file", dest="configfile", action="store", help="Specify the config file")
parser.add_option("--user", dest="username", action="store", help="Specify the username")
parser.add_option("--password", dest="password", action="store", help="Specify the user password")
parser.add_option("--max-threads",dest="maxthreads", action="store", help="Specify maximum parallel forest restore")
options, args = parser.parse_args()

link = "http://localhost:8000/v1/eval"
failed = []
backup_configs=[]
f = open(options.configfile)

logging.info("Reading configuration file")
for line in f:
    backup_configs.append(line)

def run_restore(backup_config):
    forest_name = backup_config.split(':')[0]
    backup_path = backup_config.split(':')[1]
    logging.info(forest_name + " restore started from " + backup_path)
    script = """xquery=
    xquery version "1.0-ml";
    import module namespace admin = "http://marklogic.com/xdmp/admin"
    		  at "/MarkLogic/admin.xqy";
    xdmp:forest-restore(admin:forest-get-id(admin:get-configuration(), \"""" + forest_name +"""\"), \"""" + backup_path.rstrip() + """\")"""
    r=requests.post(link, data=script, auth=HTTPDigestAuth(options.username, options.password))
    if r.status_code == 200:
        logging.info(forest_name + " forest restore COMPLETED")
    else:
        logging.error(forest_name + " forest restore FAILED")

pool = Pool(processes=int(options.maxthreads))
pool.map(run_restore, backup_configs)
pool.close()  # worker processes will terminate when all work already assigned has completed.
pool.join()  # to wait for the worker processes to terminate.

logging.info("DONE")
{% endhighlight %}


## Configuration file:

This is configuration file where you have to mention forest name and restore path separated by ':'. Below is sample configuration file.

```
testdb1:/mlbackup/20190325-0040001458910/Forests/testdb1
testdb2:/mlbackup/20190325-0040001458910/Forests/testdb2
testdb3:/mlbackup/20190325-0040001458910/Forests/testdb3
```


## Test runs:

### Success: (3 parallel restores)

```
Yogeshs-MacBook-Air:~ yogeshjadhav$ python ml_restore_new.py --config-file config.file --user admin --password changeme --max-threads 3
2019-03-25 02:08:52,372:INFO:Reading configuration file
2019-03-25 02:08:52,394:INFO:testdb1 restore started from /mlbackup/20190325-0040001458910/Forests/testdb1
2019-03-25 02:08:52,394:INFO:testdb2 restore started from /mlbackup/20190325-0040001458910/Forests/testdb2
2019-03-25 02:08:52,396:INFO:testdb3 restore started from /mlbackup/20190325-0040001458910/Forests/testdb3
2019-03-25 02:09:19,011:INFO:testdb1 forest restore COMPLETED
2019-03-25 02:09:19,011:INFO:testdb2 forest restore COMPLETED
2019-03-25 02:09:21,529:INFO:testdb3 forest restore COMPLETED
2019-03-25 02:09:21,574:INFO:DONE
```

### Success: (2 parallel restore)

```
Yogeshs-MacBook-Air:~ yogeshjadhav$ python ml_restore_new.py --config-file config.file --user admin --password changeme --max-threads 2
2019-03-25 02:21:37,650:INFO:Reading configuration file
2019-03-25 02:21:37,673:INFO:testdb1 restore started from /mlbackup/20190325-0040001458910/Forests/testdb1
2019-03-25 02:21:37,673:INFO:testdb2 restore started from /mlbackup/20190325-0040001458910/Forests/testdb2
2019-03-25 02:22:02,597:INFO:testdb1 forest restore COMPLETED
2019-03-25 02:22:02,598:INFO:testdb3 restore started from /mlbackup/20190325-0040001458910/Forests/testdb3
2019-03-25 02:22:02,876:INFO:testdb2 forest restore COMPLETED
2019-03-25 02:22:30,254:INFO:testdb3 forest restore COMPLETED
2019-03-25 02:22:30,355:INFO:DONE
```

### Failed: (I used backup directory which doesn't exists)

```
Yogeshs-MacBook-Air:~ yogeshjadhav$ python ml_restore_new.py --config-file config.file --user admin --password changeme --max-threads 3
2019-03-25 02:19:15,374:INFO:Reading configuration file
2019-03-25 02:19:15,397:INFO:testdb1 restore started from /mlbackup/20190325-0040001458910/Forests/testdb1
2019-03-25 02:19:15,398:INFO:testdb2 restore started from /mlbackup/20190325-0040001458910/Forests/testdb4
2019-03-25 02:19:15,399:INFO:testdb3 restore started from /mlbackup/20190325-0040001458910/Forests/testdb3
2019-03-25 02:19:15,785:ERROR:testdb2 forest restore FAILED
2019-03-25 02:19:40,773:INFO:testdb1 forest restore COMPLETED
2019-03-25 02:19:43,668:INFO:testdb3 forest restore COMPLETED
2019-03-25 02:19:43,706:INFO:DONE
```

Please let me know if you have something to say about this in comment box below :smile:



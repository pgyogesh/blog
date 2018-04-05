---
layout: single
comments: true
excerpt: "Python program send email notification if query is running for longer time."
header:
  overlay_image: https://source.unsplash.com/random/1200x400?nature
  overlay_filter: 0
title:  "Script: Getting email alert for long running queries"
date:   2018-02-09 10:10:10 +0800
categories: Postgresql Greenplum
read: 10 Mins
tags: postgresql greenplum scripts python
---

Hello... I have written a script to get long running queries in postgresql and greenplum. This script will check if there are any long running queries and if found it will send emails to receivers.

### Things to modify:

  - Change ENVIRONMENT to match with environment running script. This environment will be added in your email subject and sender name
      Example:

        If you set ENVIRONMENT = yjdev then your subject and sender will be like below.

        Subject = Long running queries in yjdev

        Sender = YJDEV-gpadmin@yourdomain.com
  - Change MAX_TIME for query duration (In minutes). This is string type. So, Always have this in qoutes
  - Change SENDER to match with your domain name
  - Change RECIEVERS to add multiple receivers with quoma separator. (Ex: RECEIVERS = 'yogesh.jadhav@yourdomain.com;DBA-Greenplum@yourdomain.com')

### Script

{% highlight python linenos %}
#!/usr/bin/python
import os
import sys
import smtplib
import logging

logging.basicConfig(format='%(asctime)s:%(levelname)s:%(message)s',level=logging.DEBUG)
logging.info("long_running_queries.py started")

# Declaring Environment Variables

PGDATABASE='postgres'
PGPORT=5432
PGUSER='gpadmin'
ENVIRONMENT = 'YJDEV'
MAX_TIME = '5'
SENDER = '%s-gpadmin@yourdomain.com' %ENVIRONMENT
RECEIVERS = 'DBA-Greenplum@yourdomain.com'

# Variables for SQL and Output files
LONG_QUERY_PID='psql -A -t -c "SELECT procpid FROM pg_stat_activity WHERE age(clock_timestamp(),query_start) > interval ' + '\'%s minutes\'' %MAX_TIME + ' AND current_query NOT ilike \'%IDLE%\'"'
LONG_QUERY_PSQL_STRING="""psql -H -c \"SELECT
                                        datname as Database,
                                        procpid as Process_ID,
                                        sess_id as Session_ID,
                                        usename as Username,
                                        SUBSTR(current_query,0,60) as Current_Query,
                                        now() - query_start as Query_Duration,
                                        now() - backend_start as Session_Duration,
                                        waiting as Is_Waiting FROM pg_stat_activity
                                     WHERE
                                        age(clock_timestamp(),query_start) > interval """ + '\'%s minutes\'' %MAX_TIME + """ AND
                                        current_query NOT ilike \'%IDLE%\'
                                     ORDER BY 6 desc;\" """
def sendemail():
    sender = SENDER
    receivers = RECEIVERS

    message = """From: """ + SENDER + """
To: """ + RECEIVERS + """
MIME-Version: 1.0
Content-type: text/html
Subject: Long running queries in """ + ENVIRONMENT + """\n"""
    LONG_RUNNING_QUERIES = os.popen(LONG_QUERY_PSQL_STRING).read()
    message = message + LONG_RUNNING_QUERIES
    try:
        smtpObj = smtplib.SMTP('localhost')
        smtpObj.sendmail(sender, receivers, message)
        logging.info("Successfully sent email")
    except SMTPException:
        logging.error("Unable to send email")

logging.info("Checking if queries are running for long time")
IS_LONG_RUNNING=os.popen(LONG_QUERY_PID).read()
if IS_LONG_RUNNING:
        logging.info("One or more queries running from long time")
        os.popen(LONG_QUERY_PSQL_STRING)
        sendemail()
else:
        logging.info("No queries are running for longer time. No need to send email")
logging.info("Completed Successfully")

{% endhighlight %}

If you have any issues or suggestions for this script, Hit it [here](https://github.com/pgyogesh/greenplum_dba/issues/new?title=Issue%20long%20running%20queries%20script:)

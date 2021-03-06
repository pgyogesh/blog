---
layout: single
comments: true
excerpt: "Gives a basic idea using S3 bucket as external table in Greenplum Database"
header:
  overlay_image: https://source.unsplash.com/random/1200x400?nature
  overlay_filter: 0.5
title:  "Guide to create external table in greenplum using Amazon S3 Buckets"
date:   2017-11-19 01:30:13 +0800
categories: Greenplum
read: 10 minutes read
tags: greenplum aws ETL
---

I just created external table in greenplum using Amazon S3 web service. So here I'm writing how I did it.

First we have to create two function to write to S3 and to read from S3 like below:

## Creating Functions and Protocol

### Functions

{% highlight sql linenos %}
CREATE OR REPLACE FUNCTION read_from_s3()
RETURNS integer
AS  '$libdir/gps3ext.so', 's3_import'
LANGUAGE C STABLE;
{% endhighlight %}


{% highlight sql linenos %}
CREATE OR REPLACE FUNCTION read_from_s3()
RETURNS integer
AS '$libdir/gps3ext.so', 's3_import'
LANGUAGE C STABLE;
{% endhighlight %}


Then create protocol to access S3 from greenplum database.

### Protocol

{% highlight sql linenos %}
CREATE PROTOCOL s3
(writefunc = write_to_s3, readfunc = read_from_s3);
{% endhighlight %}


## Creating Configuration File

You can create sample configuration file using *gpcloudcheck* utility like below and then you can edit it as per your setting.

{% highlight bash linenos %}
gpcheckcloud -t > s3_config.conf
{% endhighlight %}
###### Configuration file Example

{% highlight bash linenos %}
  secret = "Your AWS Secret ID"
  accessid = "Your AWS Access ID"
  threadnum = 4
  chunksize = 67108864
  low_speed_limit = 10240
  low_speed_time = 60
  encryption = true
  version = 1
  proxy = ""
  autocompress = true
  verifycert = true
  server_side_encryption = ""
  # gpcheckcloud config
  gpcheckcloud_newline = "\n"
  {% endhighlight %}

## Creating S3 bucket on AWS

  + You can follow [this](http://docs.aws.amazon.com/AmazonS3/latest/gsg/CreatingABucket.html) guide to to create S3 bucket.


## Testing configuration using gpcheckcloud utility.


You can use command line options like below:

{% highlight bash linenos %}
gpcheckcload -c "s3://S3_region_endpoint/bucket_name/[prefix] config=/path/to/config_file"
{% endhighlight %}

You can check your region endpoint [here](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region)

### Example

{% highlight bash linenos %}
$ gpcheckcloud -c "s3://s3-ap-south-1.amazonaws.com/greenplumext config=/home/gpadmin/s3.conf"
File: Screen Shot 2017-01-23 at 1.14.21 AM.png, Size: 110279
File: sample_1.csv, Size: 43
File: sample_2.csv, Size: 43

Your configuration works well.
{% endhighlight %}

Once your configuration works, You can create external table with S3 location and start access S3 files for GPDB.



## Creating and accessing S3 external tables

### Example

{% highlight sql linenos %}
CREATE EXTERNAL TABLE name_external(Firstname text, Lastname text)
LOCATION ('s3://s3-ap-south-1.amazonaws.com/greenplumext/ config=/home/gpadmin/s3_root_v1.conf')
FORMAT 'csv';

{% endhighlight %}

{% highlight sql linenos %}
SELECT * FROM name_external
WHERE firstname='Yogesh';                                                                                                

firstname | lastname
-----------+----------
 Yogesh    | Jadhav
 {% endhighlight %}

## Important Links


+ [S3 Protocol](https://gpdb.docs.pivotal.io/510/admin_guide/external/g-s3-protocol.html)
+ [Using gpcheckcloud utility](https://gpdb.docs.pivotal.io/510/admin_guide/external/g-s3-protocol.html#amazon-emr__s3chkcfg_utility)

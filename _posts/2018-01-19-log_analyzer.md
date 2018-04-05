---
layout: single
comments: true
excerpt: "Give a basic idea for creating a log analyzer using Python Programming"
header:
  overlay_image: https://source.unsplash.com/random/1200x400?nature
  overlay_filter: 0.5

title:  "Log analyzer using Python Programming Langauge"
date:   2018-01-19 23:10:10 +0800
categories: Python
read: 5 minutes read
tags: python programming scripts
---

As a MarkLogic DBA(also Greenplum DBA), We had to look at error files frequently and looking such a huge file was not a easy. So, I decided to create a log analyzer script using python. here I'll be demonstrating this script with sample log file(Not MarkLogic)

## Script:

{% highlight python linenos %}

import re
import optparse

parser = optparse.OptionParser()
parser.add_option("-f","--file",dest="log_file",
                          action="store",help="Specify log file to be parsed")
options, args = parser.parse_args()
vLogFile=options.log_file

counter = 0
hour = ['00','01','02','03','04','05','06','07','08','09','10','11','12','13','14','15','16','17','18','19','20','21','22','23']
for h in hour:
    error_regex= '^%s:\d\d-Error' %h
    file = open(vLogFile,"r")
    for line in file:
        for match in re.finditer(error_regex,line,re.S):
            counter = counter + 1
    if counter != 0:
        print(h,':',counter * '*',counter)
    counter = 0
    file.close()
{% endhighlight %}

## Demo:

<script src="https://asciinema.org/a/KDb3eEYVWjDDzEyY3uY7ztN8A.js" id="asciicast-KDb3eEYVWjDDzEyY3uY7ztN8A" async></script>

Note: Actually, Script will give result like below but I'm not sure why it resulted in asciinema recorder. So dont worry :smile:

{% highlight bash linenos %}
$ python log_analyzer.py -f sample.log
00 : * 1
09 : *** 3
10 : * 1
11 : *********** 11
18 : * 1
23 : * 1
{% endhighlight %}

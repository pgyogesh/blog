---
layout: single
comments: true
excerpt: "Give a basic idea for creating a log analyzer using Python Programming"
header:
  image: https://source.unsplash.com/random/1200x400?nature
  overlay_filter: 0

title:  "Log analyzer using Python Programming Langauge"
date:   2018-01-19 23:10:10 +0800
categories: Python
read: 5 minutes read
tags: python programming scripts
---

As a MarkLogic DBA(also Greenplum DBA), We had to look at error files frequently and looking such a huge file was not a easy. So, I decided to create a log analyzer script using python. here I'll be demonstrating this script with sample log file(Not MarkLogic)

Here is script:

<script src="https://gist.github.com/pgyogesh/43c93aa87adb815e7eef2c112f72c3c3.js"></script>

Here is demo:

<script src="https://asciinema.org/a/KDb3eEYVWjDDzEyY3uY7ztN8A.js" id="asciicast-KDb3eEYVWjDDzEyY3uY7ztN8A" async></script>

Note: Actually, Script will give result like below but I'm not sure why it resulted in asciinema recorder. So dont worry :smile:

```bash
$ python log_analyzer.py -f sample.log
00 : * 1
09 : *** 3
10 : * 1
11 : *********** 11
18 : * 1
23 : * 1
```

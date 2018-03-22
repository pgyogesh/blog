---
layout: post
title:  "My first GitHub repository for Greenplum DBA"
date:   2017-04-20 01:30:13 +0800
categories: GitHub
read: 5 minutes read
tags: github greenplum postgresql
---

Recently I have created a github repository for Greenplum and PostgreSQL DBA's to simplify day to day activities. It includes dotfiles, Shell scripts, Python scripts and SQL scripts. Few of them I have created myself and few I have copied from somewhere on Internet.

You can visit [this link](https://github.com/pgyogesh/greenplum_dba) and see or contribute to this repository.

How to install:


First you should check if you have git installed. You can check it using below command:

`which git`

It should give you the path for git command. If not, You can install it using by following [this guide](https://git-scm.com/download/linux).

Once you have git installed run below commands step by step.

`mkdir $HOME/git`

`cd $HOME/git`

`git clone https://github.com/pgyogesh/greenplum_dba.git`

`cd greenplum`

`chmod +x install.sh`

`./install.sh`

It will take some time to install completely as this will download few repositories related to vim integration but you don't have to worry about this install.sh will take care of this. You can seat back and relax :)

Features:

:one: All required environment exports

:two: Aliases to make work easier

:three: Python autocomplete for vim

:four: Python Script to get notified through email with waiters and blockers information if queries are in waiting state more that 10 minutes (Using gmail,outlook etc)

:five: Shell script to get notified through email with waiters and blockers information if queries are in waiting state more that 10 minutes (Using Unix Mail Utility)

:six: Shell script to archive the GPDB log files in pg_log/YEAR/MONTH directory

:seven: Greenplum Utility like logging using gplog.py from gpdb source code.

:eight: And More... :blush:

:confused: Have a issue..? Please raise [here](https://github.com/pgyogesh/greenplum_dba/issues/new)

Here is installation demo;

<script src="https://asciinema.org/a/119139.js" id="asciicast-119139" async></script>

Note: In above demo you will see repo name greenplum but later I have renamed it to greenplum_dba for some reasons.

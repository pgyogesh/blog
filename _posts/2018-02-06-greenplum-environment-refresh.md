---
layout: single
excerpt: "Python script to refresh data between two environments and switch the roles"
header:
  overlay_image: /images/unsplash-image-04.jpg
  overlay_filter: 0.5
title:  "Script: Greenplum environment refresh"
date:   2018-02-06 10:10:10 +0800
categories: Greenplum
read: 15 mins
tags: greenplum postgresql python scripts
---

Previously I wrote a [post](http://pgyogesh.com/greenplum/2017/12/14/environment_refresh.html){:target="_blank"} which is for switching the roles from one environment to other environment.

This is just a extended script of same post. This script takes care for taking backup and restoring it into target database and then switches role like previous [post](http://pgyogesh.com/greenplum/2017/12/14/environment_refresh.html){:target="_blank"}

You can fork these from my [github site](https://github.com/pgyogesh/greenplum-environment-refresh){:target="_blank"}

#### Here is main script:

<script src="https://gist.github.com/pgyogesh/aa6a5948c4ee0da597c4776b859ffd4a.js"></script>

#### Here is example of sample config file

<script src="https://gist.github.com/pgyogesh/f51d16ca06479a9633654a37316b89dd.js"></script>

#### Here is sample of schema list

<script src="https://gist.github.com/pgyogesh/d581b75bb3ec53e5881dddb785d86ec1.js"></script>

Let me know if you have issues or suggestion for this script [here](https://github.com/pgyogesh/greenplum-environment-refresh/issues/new){:target="_blank"} or comment below in comment box.

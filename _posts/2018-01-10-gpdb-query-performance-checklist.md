---
layout: single
comments: true
excerpt: "Lists basic troubleshooting of performance issues in Greenplum"
header:
  overlay_image: /images/unsplash-07.jpg
  overlay_filter: 0.3
title:  "Greenplum Database Query Performance Checklist"
date:   2018-01-10 01:30:13 +0800
categories: Greenplum
read: 20 minutes read
tags: greenplum postgresql SQL
---

As a Greenplum Database system is MPP there are lot of things we can check during performance issues. Here are few things that Greenplum Database Administrator can check while troubleshooting.

* If users are claiming the query is running for longer time, Check if query is waiting on [locks](http://www.pgyogesh.com/gpdb/2017/12/06/important-gpdb-sqls.html#waiters-information){:target="_blank"} or [Resource Queue](http://www.pgyogesh.com/gpdb/2017/12/06/important-gpdb-sqls.html#find-running-queries-or-statements-which-are-waiting-in-resource-queues){:target="_blank"}
* Try to run the same query from master instance and check if queries are really running for longer time. Sometimes, It's network between master server and application causes more time to send the query result to application.


* Check if there are any hardware issues by running `dcacheck` utility if you are using EMC DCA.

* Check if there is load on master and segment server using `uptime` utility. This is quick way to check the load average. Though it will not display lot of information but it can give high level idea to move forward on investigation.

   - ### Example:
      Below result displays load averages for last one minute, 5 minutes and 15 minutes respectively.

      ```sh
      $ uptime
       13:15  up  2:23, 2 users, load averages: 1.38 1.81 2.04
      ```

   Run `uptime` using `gpssh` on all hosts, if there is load unexpected load on and server and do some hardware checks like network and storage.


### Network:
* Check for network issues using `ethtool` and `ifconfig` utilities.

    - #### Example:

      ```sh
      $ ethtool eth0
      Settings for enp0s3:
	   Supported ports: [ TP ]
	   Supported link modes:   10baseT/Half 10baseT/Full
	                           100baseT/Half 100baseT/Full
	                           1000baseT/Full
	   Supported pause frame use: No
	   Supports auto-negotiation: Yes
	   Advertised link modes:  10baseT/Half 10baseT/Full
	                           100baseT/Half 100baseT/Full
	                           1000baseT/Full
	   Advertised pause frame use: No
	   Advertised auto-negotiation: Yes
	   Speed: 10000Mb/s <===
	   Duplex: Full <===
	   Port: Twisted Pair
	   PHYAD: 0
	   Transceiver: internal
	   Auto-negotiation: on
	   MDI-X: off (auto)
	   Supports Wake-on: umbg
	   Wake-on: d
	   Current message level: 0x00000007 (7)
		   	       drv probe link
	   Link detected: yes
      $
      $
      $ ifconfig eth0
      eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::a00:27ff:fe9d:c7fc  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:9d:c7:fc  txqueuelen 1000  (Ethernet)
        RX packets 34686  bytes 32923726 (31.3 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0 <===
        TX packets 12508  bytes 6625616 (6.3 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0 <===
      ```
      In ethtool check for duplex and speed. It should be full and 10GB(Recommended for GPDB). And In ifconfig, check if counts for errors and dropped packets are increasing by running several times.


### Hardware:

* Check if any virtual drive cache policy has changed to 'write through'.


* Check recent `dmesg` errors that can cause perfomance issue.

	- #### Example:

	```bash
	[root@mdw ~]# dmesg | tail
	igb 0000:02:00.2: eth2: igb: eth2 NIC Link is Down
	bond1: making interface eth2 the new active one
	bond1: link status definitely down for interface eth2, disabling it
	bond1: now running without any active interface!
	igb 0000:02:00.2: eth2: igb: eth2 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
	igb 0000:02:00.1: eth1: igb: eth1 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: RX
	bond1: link status definitely up for interface eth1, 1000 Mbps full duplex
	bond1: making interface eth1 the new active one
	bond1: first active interface up!
	bond1: link status definitely up for interface eth2, 1000 Mbps full duplex
	```

* Check virtual memory stats usings `vmstat 1`. With argument 1, It runs each second until we cancel it. Remember, First line will have average stats from boot instead for last second. It will display average stats across all CPUs.

	- #### Columns to check:

	**r:** Number of processes running on CPU and waiting for a turn. This provides a better signal than load averages for determining CPU saturation, as it does not include I/O.

	**si,so**: Swap-ins and swap-outs. If these are non-zero, you’re out of memory.

	**us:** User time on CPU.

	**sy:** Systerm time on CPU.

	**id:** Idle time on CPU.

* Check disk performance using `iostat -xz 1`

	- #### Example:

	```
	[root@mdw ~]# iostat -xz 1
	Linux 2.6.32-642.15.1.el6.x86_64 (mdw.gphd.local)       01/08/2018      _x86_64_        (32 CPU)

	avg-cpu:  %user   %nice %system %iowait  %steal   %idle
        	   0.40    0.00    0.18    0.00    0.00   99.42

	Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
	sdd               0.00     0.01    0.00    0.00     0.00     0.05   159.46     0.00    0.94    1.21    0.88   0.42   0.00
	sde               0.00     1.47    0.14    3.16    71.32   212.20    85.79     0.01    3.92   12.16    3.55   0.19   0.06
	sda               0.00     3.27    0.01    3.89     1.80    57.31    15.14     0.00    0.08    4.60    0.06   0.05   0.02
	sdc               0.00     0.00    0.00    0.00     0.00     0.00     8.77     0.00    1.41    1.41    0.00   1.41   0.00
	sdb               0.00     0.02    0.00    0.02     0.00     0.26    14.71     0.00    0.28    4.39    0.25   0.27   0.00

	avg-cpu:  %user   %nice %system %iowait  %steal   %idle
	           0.03    0.00    0.19    0.03    0.00   99.75

	Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util

	avg-cpu:  %user   %nice %system %iowait  %steal   %idle
	           0.03    0.00    0.06    0.00    0.00   99.91

	Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
	sde               0.00     4.00    0.00   27.00     0.00  1624.00    60.15     0.01    0.30    0.00    0.30   0.04   0.10

	avg-cpu:  %user   %nice %system %iowait  %steal   %idle
	           0.00    0.00    0.09    0.00    0.00   99.91

	Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
	sda               0.00     0.00    0.00    1.00     0.00     8.00     8.00     0.00    3.00    0.00    3.00   3.00   0.30

	avg-cpu:  %user   %nice %system %iowait  %steal   %idle
	           0.06    0.00    0.06    0.00    0.00   99.87

	Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util

	avg-cpu:  %user   %nice %system %iowait  %steal   %idle
	           0.09    0.00    0.13    0.00    0.00   99.78

	Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
	sda               0.00    95.00    0.00  137.00     0.00  1856.00    13.55     0.01    0.07    0.00    0.07   0.07   0.90
	```
	- #### Columnd to check:

	**r/s, w/s, rkB/s, wkB/s:** These are the read and writes from the device.

	**await:** This the average time for the IO in miliseconds. Larger than expected value can be indicator of device problem.

	**%util:** Displays the device utlization.

* Check `free -m` for used and free memory.

* Check system activity report using `sar 1` which displays stats for every second. This is my favorite.

	- #### Example:

	```
	[root@mdw ~]# sar 1
	Linux 2.6.32-642.15.1.el6.x86_64 (mdw.gphd.local)       01/08/2018      _x86_64_        (32 CPU)

	05:48:28 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
	05:48:29 AM     all      0.03      0.00      0.34      0.00      0.00     99.62
	05:48:30 AM     all      1.06      0.00      0.53      0.00      0.00     98.41
	05:48:31 AM     all      0.41      0.00      0.34      0.00      0.00     99.25
	05:48:32 AM     all      0.03      0.00      0.34      0.00      0.00     99.62
	05:48:33 AM     all      0.16      0.00      0.38      0.00      0.00     99.47
	05:48:34 AM     all      1.31      0.00      0.50      0.00      0.00     98.19
	05:48:35 AM     all      0.16      0.00      0.94      0.00      0.00     98.90
	```

	- #### Columns to check

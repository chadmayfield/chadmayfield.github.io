---
layout:     post
title:      Monitoring a Dell PERC 6/i
date:       2017-04-24 11:21:00
summary:    chk_raid.sh is a small shell script designed to be run from cron that will monitor the health of your PERC array and warn (via email) of any errors.
#categories: linux admin dell perc raid
---

It seems like my life is full of monitoring things, my kids, my home, my cars, my job.  As with most developers, I can get a bit obsesive about system health.  But as with everything else do I really want to have to monitor my system health or let my servers do that themselves?

A few years ago I recycled a couple of old servers of mine.  I kept the three Dell PERC 6/i RAID adapters from those servers thinking I would find a use for them some day.  Then a couple of years ago I used two of them in a new small server build of mine.  Put old cards in a new server?  Sure, I didn't need super speed and I wanted to keep costs down, so why not?

So I started thinking about how to monitor the arrays on the PERC adapters.  Now there are plenty of [PERC](https://blog.frehi.be/2011/09/12/megacli-useful-commands/) [tutorials](http://thatlinuxbox.com/blog/article.php/lsi-megaraid-megacli) and command cheatsheets out there, and even [scripts](https://calomel.org/megacli_lsi_commands.html) to view info about your array. But nothing suited my needs.  Most what I found were great and I have even used them in the past, but I wanted more.  So I decided to create a simple cronjob to run some commands to check for health.  Basically just a couple of grep's on commands that were run.  The commands I was most concerned about are;

```
# display adapter information
/opt/MegaRAID/MegaCli/MegaCli64 -AdpAllInfo -aAll

# display logical drive information
/opt/MegaRAID/MegaCli/MegaCli64 -LDInfo -LALL -aAll

# display physical drive information
/opt/MegaRAID/MegaCli/MegaCli64 -PDList -aAll

# in the case of a rebuild, view it's status
/opt/MegaRAID/MegaCli/MegaCli64 -PDRbld -ShowProg -PhysDrv [?:1] -aAll
```

So I installed the MegaCli64 command line tool and started creating a one line cronjob that, when run, would email me any problems.  The problem is as I was writing it, it became more and more complex and less and less a one liner.  Before I knew it I had written a whole script to take care of displaying useful information as well as monitoring health.  You can get the finished product in my [scriptlets repository](https://github.com/chadmayfield/scriptlets) on GitHub, it is the [chk_raid.sh](https://github.com/chadmayfield/scriptlets/blob/master/chk_raid.sh) script.  Check it out, play with it, modify it to suit your needs.

There are only two arguments to the script, `info` and `monitor`.  When running `info` you'll see information returned to the user about Model, Firmware versions, Disks, and State.

```
[root@myhost ~]# ./chk_raid.sh info
Product Name           PERC 6/i Adapter
Serial No              1122334455667788
FW Package Build       6.3.1-0003
FW Version             1.22.32-1371
BIOS Version           2.04.00
Host Interface         PCIE
Memory Size            256MB
Supported Drives       SAS, SATA
Virtual Drives         1
  Degraded             0
  Offline              0
Physical Devices       4
  Disks                4
  Critical Disks       0
  Failed Disks         0
Virtual Drive Info
  RAID Level           Primary-5, Secondary-0, RAID Level Qualifier-3
  Size                 4.091 TB
  Sector Size          512
  Strip Size           64 KB
  Number Of Drives     4
  Span Depth           1
Drive Status           OPTIMAL
  Slot Number 0        Online, Spun Up      9VS12A34ST1500DM003-9YN16G
  Slot Number 1        Online, Spun Up      9VS12B34ST1500DM003-9YN16G
  Slot Number 2        Online, Spun Up      9VS12C34ST1500DM003-9YN16G
  Slot Number 3        Online, Spun Up      9VS12D34ST1500DM003-9YN16G
```

When running to monitor an array, it is best to set it as a cronjob.  Personally I run it hourly.  If there's a problem with either a degraded array or even a  failed or citical disk you'll get an email like this;

```
Subject
-------
WARNING: Problems with RAID array on file.lomiz.com!

Body
----
STATE:  Degraded
ERROR:  1 Disks Degraded
ERROR:  1 Disks Offline
ERROR:  0 Critical Disks
ERROR:  0 Failed Disks
--------------------
State            Degraded
Degraded         1
Offline          1
Disks            4
Critical Disks   0
Failed Disks     0
```

Hopefully someone will find this useful, if so, or for questions, please [email me](https://chadmayfield.com/contact/) or find me on [Twitter](https://twitter.com/chadrmayfield).


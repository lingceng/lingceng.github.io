---
layout: post
title: "Run Command Every 5 Hours On Linux"
date: 2013-11-30 16:56
comments: true
categories: [linux]
---

Here's my crontab on ubuntu12.04:

    > crontab -l
    PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

    # m h dom mon dow usercommand
    0 */5 * * * /path/test.sh >> /path/test.log 2>&1

I set **PATH** variable here as crontab **PATH** is different with **PATH** in login shell.  
It seems that crontab ignores ~/.bashrc or ~/.profile.  
Here's the result of `env` command in crontab:

    HOME=/home/lingceng
    LOGNAME=lingceng
    PATH=/usr/bin:/bin
    LANG=en_US.UTF-8
    SHELL=/bin/sh
    PWD=/home/lingceng

And more, the comamnd runs at **0, 5, 15, 20 o'clock**, not 5 hours after the crontab set.



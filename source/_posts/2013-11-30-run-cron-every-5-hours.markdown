---
layout: post
title: "Run Cron Every 5 Hours"
date: 2013-11-30 16:56
comments: true
categories: 
---

\> crontab -e

    # set the shell
    SHELL=/bin/sh
    PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

    # m h dom mon dow usercommand
    0 */5 * * * /path/hello.sh >> /path/hello.log 2>&1
    
<!-- more -->
#### Setting the PATH is Always Necessary

*Crontab ignores login configuration*, which leads to a command not found. 
I have tested *env* command with crontab on my Ubuntu12.04 and I got the following result:

    HOME=/home/lingceng
    LOGNAME=lingceng
    PATH=/usr/bin:/bin
    LANG=en_US.UTF-8
    SHELL=/bin/sh
    PWD=/home/lingceng

#### Not Really Every 5 Hours
The demo shell script **just runs at 0, 5, 15, 20 o'clock**, 20 to 0 is just 4 hour.

### get more help
\> [man 5 crontab](http://unixhelp.ed.ac.uk/CGI/man-cgi?crontab+5)


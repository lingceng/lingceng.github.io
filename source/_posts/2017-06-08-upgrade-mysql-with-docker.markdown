---
layout: post
title: "Upgrade MySQL With Docker"
date: 2017-06-08 23:18
comments: true
categories: ["rails", "mysql", "docker"]
---

### The search problem

Recently a friend and I made [a comic App](http://www.cookacg.com).
I crawled about 70,000 comics and saved them into MySQL.
Search for comic name started to slow down.
It takes about 2000ms.

    SELECT `comics`.* FROM `comics` WHERE `comics`.`name` LIKE '%text%' LIMIT 21 OFFSET 0

I had added an index for name field. 
But an index won't help text matching with a leading wildcard, an index can be used for:

    LIKE 'text%'

Query with `LIKE 'text%'` is much faster. But comic name such as "sometext" won't match.
It does not fit my need.

**I found out, that the same query was much faster on my MacBook, which only took about 200ms.**
Here was the difference:

    EVN        | CPU                   | Memory | OS           | MySQL Version
    ---        | ---                   | ---    | ---          | ---
    MacBook    | 2.7 GHz Intel Core i5 | 8G     | MacOS Sierra | 5.7.17
    Production | 2.6 GHZ 1 Core        | 2G     | CentOS 6.5   | 5.1.73

**After some tests, I found out MySQL Version was the key point.**
Why 5.7 is so much faster than 5.1? I don't know.

### Try to upgrade MySQL
So I needed to upgrade MySQL from 5.1 to 5.7.
I had 3 choices:

+ Using mysql_upgrade
+ Installing MySQL5.7 manually on the same machine
+ Installing MySQL5.7 in docker

Installing MySQL.7 in docker seemed much safer and simpler for me. 
And docker is an useful tool to help deploying.
I tried to install the latest stable docker but failed, as the latest Docker CE is only supported on CentOS 7.3 64-bit.
I installed the [docker 1.7](https://docs.docker.com/v1.7/docker/installation/centos/) which is supported on CentOS 6.5.

Run MySQL in docker and publish to host port 6603:

    sudo docker run --detach --name=comic-mysql --env="MYSQL_ROOT_PASSWORD=mypassword" --publish 6603:3306 mysql

Connect MySQL in docker with host MySQL client:

    mysql -uroot -p -h 127.0.0.1 -P 6603

Create database:

    CREATE DATABASE comic_production CHARACTER SET utf8 COLLATE utf8_general_ci;

Import sql.gz file:

    zcat ~/backup/comic_production.20170606112459.sql.gz  | sudo docker exec -i comic-mysql mysql -uroot -pmypassword comic_production

Configure my rails database.yml:

    default: &default
      adapter: mysql2
      pool: 5
      username: root
      password: mypassword
      host: 127.0.0.1
      port: 6603

It worked!
I restarted the rails server, the database changed.
The users got much faster response for searching.

### References
+ https://stackoverflow.com/questions/2042269/how-to-speed-up-select-like-queries-in-mysql-on-multiple-columns
+ https://dev.mysql.com/doc/refman/5.7/en/multiple-servers.html

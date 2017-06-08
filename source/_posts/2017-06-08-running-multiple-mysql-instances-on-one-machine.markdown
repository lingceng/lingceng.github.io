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
Search for comic name starts to slow down.
It takes about 2000ms.

    SELECT `comics`.* FROM `comics` WHERE `comics`.`name` LIKE '%text%' LIMIT 21 OFFSET 0

I have added an index for name field. 
But an index won't help text matching with a leading wildcard, an index can be used for:

    LIKE 'text%'

The query gets much faster with 'name' index, but comic name such as "hitext" won't match.
It's not the solution.

I find out, that the same query is much faster on my MacBook, which only takes about 200ms.
Here's the diff:

    EVN        | CPU                   | Memory | OS           | MySQL Version
    ---        | ---                   | ---    | ---          | ---
    MacBook    | 2.7 GHz Intel Core i5 | 8G     | MacOS Sierra | 5.7.17
    Production | 2.6 GHZ 1 Core        | 2G     | CentOS 6.5   | 5.1.73

After some tests, I find out MySQL Version is the key point. 
Why 5.7 is so much faster than 5.1? I don't know.

### Try to upgrade MySQL
So I need to upgrade MySQL from 5.1 to 5.7.
I have 3 choices:

+ Use mysql_upgrade
+ Install MySQL5.7 manually on the same machine and import data
+ Install MySQL5.7 in docker and import data

Install MySQL.7 in docker seems much safer and simpler.
I tried to install the latest stable docker but failed, as the lastest Docker CE is only supported on CentOS 7.3 64-bit.
I installed the [docker1.7](https://docs.docker.com/v1.7/docker/installation/centos/) which is supported on CentOS6.5.

Install MySQL and publish to host port 6603:

    sudo docker run --detach --name=comic-mysql --env="MYSQL_ROOT_PASSWORD=mypassword" --publish 6603:3306 mysql

Connect with host MySQL client, it works:

    mysql -uroot -p -h 127.0.0.1 -P 6603

Create database:

    CREATE DATABASE comic_production CHARACTER SET utf8 COLLATE utf8_general_ci;

Import sql.gz file:

    zcat ~/backup/comic_production.20170606112459.sql.gz  | sudo docker exec -i comic-mysql mysql -uroot -pmypassword comic_production

Config my rails database.yml:

    default: &default
      adapter: mysql2
      pool: 5
      username: root
      password: mypassword
      host: 127.0.0.1
      port: 6603

### References
+ https://stackoverflow.com/questions/2042269/how-to-speed-up-select-like-queries-in-mysql-on-multiple-columns
+ https://dev.mysql.com/doc/refman/5.7/en/multiple-servers.html

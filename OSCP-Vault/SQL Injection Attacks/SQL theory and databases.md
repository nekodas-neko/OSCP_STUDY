---
tags:
  - phase/exploitation
  - sqli
  - web
---

# SQL theory and databases

10.1.1. SQL theory refresher
Structured Query Language (SQL) has been developed specifically to manage and interact with data stored inside relational databases. SQL can be employed to query, insert, modify, or even delete data, and, in some cases, execute operating system commands. Since the SQL instance offers so many administrative privileges, we'll soon observe how arbitrary SQL queries can pose a significant security risk.

We can use the SELECT statement to instruct the database that we want to retrieve all (*) the records from a specific location defined via the FROM keyword and followed by the target, in this case, the users table. Finally, we'll direct the database to filter only for records belonging to the user leon.

> [!note]- Screenshot
> ```
> We can use the SELECT statement to instruct the database that we want to
> retrieve all (*) the records from a specific location defined via the FROM keyword
> and followed by the target, in this case, the users table. Finally, we'll direct the
> database to filter only for records belonging to the user leon.
> | SELECT * FROM users MERE user_nane~‘1eon’
> Listing 1- SQL query that parses the users table
> 
> To automate functionality, web applications often embed SQL queries within their
> source code.
> We can better understand this concept by examining the following backend PHP
> code portion that is responsible for verifying user-submitted credentials during
> login:
> 
> <2php
> 
> $uname = $_POST[ “uname” ];
> 
> $passwd = $ POST["password'];
> 
> $sql_query = “SELECT * FROM users WHERE user_name= ‘$uname’ AND password='$passw
> 
> an;
> 
> $result = mysqli_query($con, $sq1_query);
> 
> »
> 
> Listing 2 - SQL Query Embedded in PHP Login Source Code
> ```

Highlighted above is a semi-precompiled SQL query that searches the users table for the provided username and its respective password, which are saved into the $uname and $passwd variables. The query string is then stored in sql_query and used to perform the query against the local database through the mysqli_query function, which saves the result of the query in $result.

Let's consider an example. When the user types leon, the SQL server searches for the username "leon" and returns the result. To search the database, the SQL server runs the query SELECT * FROM users WHERE user_name= leon. If, instead, the user enters "leon '+!@#$", the SQL server will run the query SELECT * FROM users WHERE user_name= leon'+!@#$. Nothing in our code block checks for these special characters, and it's this lack of filtering that causes the vulnerability.

We'll discover how these types of scenarios can be abused in the next sections.

10.1.2. DB types and characteristics

MySQL is one of the most deployed database variants, along with MariaDB, an open-source fork of MySQL.

To explore MySQL basics, we can connect to the remote MySQL instance from our local Kali machine.

Using the mysql command, we'll connect to the remote SQL instance by specifying root as username and password, along with the default MySQL server port 3306.

> [!note]- Screenshot
> ```
> kaligkali:~$ mysql -u root -p'root’ -h 192.168.50.16 -P 3306 --skip-ssl-verify-s
> erver-cert
> Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
> Type ‘help;* or "\h’ for help. Type *\c’ to clear the current input statement.
> MySQL [(none)]>
> Listing 3 - Connecting to the remote MySQL. instance
> © Info
> If ERROR 2026 (HY000) TLS/SSL error shows up, we can append | --skip-
> ssl |at the end of the command to connect to the database.
> From the MySQL console shell, we can run the version() function to retrieve the
> version of the running SQL instance.
> ```


```sh
mysql -u root -p'root' -h 192.168.50.16 -P 3306 --skip-ssl-verify-server-cert
```


> [!note]- Screenshot
> ```
> From the MySQL console shell, we can run the version() function to retrieve the
> version of the running SQL instance.
> 
> MySQL [(none)]> select version();
> 
> gessesssssectt
> 
> | version() |
> 
> gessssssssscit
> 
> | 8.0.21 |
> 
> gessssssssscit
> 
> 1 row in set (0.107 sec)
> 
> Listing 4 - Retrieving the version of a MySQL database
> 
> We can also verify the current database user for the ongoing session via the
> system_user() function, which returns the current username and hostname for the
> MySQL connection.
> 
> MySQL [(none)]> select system _user();
> 
> gesceesssssesssssssecd
> 
> | system_user() I
> 
> gessesesssssessssssest
> 
> | root@192.168.20.50 |
> 
> gessesssssssessssssect
> 
> 1 row in set (0.104 sec)
> 
> Listing 5 - Inspecting the current session's user
> The database query we ran confirmed that we are logged in as the database root
> user through a remote connection from 192.168.20.50.
> The root user in this example is the database-specific root user,
> not the system-wide administrative root user.
> ```

After listing databases with:
SQLSHOW DATABASES;Show more lines
you switch to one using the USE command:
SQLUSE mysql;Show more lines
If successful, MySQL will respond:
Database changed

Now your prompt will look like:
MySQL [mysql]>

That’s how the example got into the mysql database before running the SELECT.

> [!note]- Screenshot
> ```
> We can now collect a list of all databases running in the MySQL session by
> issuing the show command, followed by the databases keyword.
> 
> MySQL [(none)]> show databases;
> 
> gesceesssssesssssssecd
> 
> | Database I
> 
> gessesssssssesssssse
> 
> | information_schema |
> 
> | mysql |
> 
> | performance_schema |
> 
> I sys I
> 
> | test I
> 
> gessesssssssessssssect
> 
> 5 rows in set (0.107 sec)
> 
> Listing 6 - Listing all Available Databases
> 
> As an example, let's retrieve the password of the offsec user present in the mysq/
> database.
> Within the mysqI database, we'll filter using a SELECT statement for the user and
> authentication_string value belonging to the user table. Next, we'll filter all the
> results via a WHERE clause that matches only the offsec user.
> 
> MySQL [mysql]> SELECT user, authentication_string FROM mysql.user WHERE user =
> 
> “of fsec’ 5
> 
> gesseessajecsesssccssssscessssscessscecessssscessssssessssssessssssesssssssesest
> 
> =o
> 
> | user | authentication_string
> 
> I
> 
> gessecssafjecssssecessscccesscsscessecscessssscessssscessssssessssssesssssssessse
> 
> =o
> 
> | offsec | $2$005$?qvorPpS#1TKH154xuw4CS5VsXeSIAal cFUYdQMiBXQVEZZG9XNd/e6 |
> 
> gescesssa{jecsessecessssscessscscescscecessssscessssssessssssecsssssesssssssessse
> 
> =o
> 
> 1 row in set (0.106 sec)
> 
> Listing 7 - Inspecting user's encrypted password
> ```


```sh
SELECT user FROM mysql.user;
SELECT user, authentication_string FROM mysql.user WHERE user = 'offsec';
```

To improve its security, the user's password is stored in the authentication_string field as a Caching-SHA-256 algorithm.

A password hash is a ciphered representation of the original plain-text password. In later Learning Modules, we'll learn how password hashing is performed and how a hash can be reversed or cracked to obtain the original password.

MSSQL is a database management system that natively integrates into the Windows ecosystem.

Windows has a built-in command-line tool named SQLCMD, that allows SQL queries to be run through the Windows command prompt or even remotely from another machine.

Kali Linux includes Impacket, a Python framework that enables network protocol interactions. Among many other protocols, it supports Tabular Data Stream (TDS), the protocol adopted by MSSQL that is implemented in the impacket-mssqlclient tool.

We can run impacket-mssqlclient to connect to the remote Windows machine running MSSQL by providing a username, a password, and the remote IP, together with the -windows-auth keyword. This forces NTLM authentication (as opposed to Kerberos). We'll explore Windows authentication in more depth in upcoming Learning Modules.

> [!note]- Screenshot
> ```
> kaligkali:~§ impacket-mssqlclient Administrator :Lab123@192.168.50.18 -windows-au
> th
> Impacket v@.9.24 - Copyright 2021 SecureAuth Corporation
> [*] Encryption required, switching to TLS
> [*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
> [*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
> [*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
> [*] INFO(SQL@1\SQLEXPRESS): Line 1: Changed database context to ‘master’.
> [*] INFO(SQLO1\SQLEXPRESS): Line 1: Changed language setting to us_english.
> [*] ACK: Result: 1 - Microsoft SQL Server (15@ 7268)
> [!] Press help for extra shell commands
> SQL (SQLPLAYGROUND\Administrator dbo@master)>
> Listing 8 - Connecting to the Remote MSSQL instance via Impacket
> To begin, let's inspect the current version of the underlying operating system by
> selecting the @@version.
> Every database management system has its own syntax that we
> should take into consideration when enumerating a target during
> a penetration test.
> ```


```sh
impacket-mssqlclient Administrator:Lab123@192.168.50.18 -windows-auth
```


> [!note]- Screenshot
> ```
> SQL (SQLPLAYGROUND\Administrator dbogmaster)> SELECT @@version;
> Microsoft SQL Server 2019 (RTM) - 15.0.2000.5 (x64)
> Sep 24 2019 13:48:23
> Copyright (C) 2019 Microsoft Corporation
> Express Edition (64-bit) on Windows Server 2022 Standard 10.0 <x64> (Build 2
> 0248: ) (Hypervisor)
> Listing 9 - Retrieving the Windows OS Version
> Our query returned valuable information about the running version of the MSSQL
> server along with the Windows Server version, including its build number.
> When using an SQL Server command line tool like sqicmd, we
> must submit our SQL statement ending with a semicolon followed
> by GO ona separate line. However, when running the command
> remotely, we can omit the GO statement since it's not part of the
> MSSQL TDS protocol.
> To list all the available databases, we can select all names from the system
> catalog.
> SQL (SQLPLAYGROUND\Administrator dbo@master)> SELECT name FROM sys.databases;
> name
> master
> tempdb
> model
> msdb
> offsec
> soL>
> Listing 10 - Inspecting the Available Databases
> ```


```sh
SELECT name FROM sys.databases;
```

Since master, tempdb, model, and msdb are default databases, we want to explore the custom offsec database because it might contain data belonging to our target. We can review this database by querying the tables table in the corresponding information_schema.

> [!note]- Screenshot
> ```
> SQL (SQLPLAYGROUND\Administrator dbo@master)> SELECT * FROM offsec.information_
> 
> schema. tables;
> 
> TABLE_CATALOG TABLE SCHEMA TABLE NAME TABLE_TYPE
> 
> offsec dbo users b°BASE TABLE’
> 
> SQL (SQLPLAYGROUND\Administrator dbogmaster)>
> 
> Listing 11 - Inspecting the Available Tables in the offsec Database
> 
> Our query returned the users table as the only one available in the database, so
> let's inspect it by selecting all of its records. We'll need to specify the dbo table
> schema between the database and the table names.
> 
> SQL> select * from offsec.dbo.userss
> 
> username password
> 
> admin lab
> 
> guest guest
> 
> Listing 12 - Exploring Users Table Records
> ```

The users table contains the columns, user, and password, and two rows. Our query returned the clear text password for both usernames.

Having covered the basic syntax peculiarities for MySQL and MSSQL databases, next, we'll learn how to manually exploit SQL injection vulnerabilities.

## EXAMPLE:

QUESTION:
From your Kali Linux VM, connect to the remote MySQL instance on VM 1 and replicate the steps to enumerate the MySQL database. Then explore all values assigned to the user offsec. Which plugin value is used as a password authentication scheme?

-- Connect to MySQL remotely
mysql -h <target-ip> -u <username> -p
i.e

## mysql -h 192.168.105.16 -P 3306 -u root -p'root' --skip-ssl-verify-server-cert

-- Enumerate databases
SHOW DATABASES;

-- Select mysql system DB
USE mysql;

-- List tables
SHOW TABLES;

-- List users
SELECT user FROM mysql.user;

-- Inspect specific user
SELECT user, authentication_string, plugin 
FROM mysql.user 
WHERE user = 'offsec';

+--------+------------------------------------------------------------------------+-----------------------+
| user   | authentication_string                                                  | plugin                |
+--------+------------------------------------------------------------------------+-----------------------+
| offsec | $A$005$?qvo▒rPp8#lTKH1j54xuw4C5VsXe5IAa1cFUYdQMiBxQVEzZG9XWd/e6 | caching_sha2_password |
+--------+------------------------------------------------------------------------+-----------------------+

## EXAMPLE:

QUESTION:
From your Kali Linux VM, connect to the remote MSSQL instance on VM 2 and replicate the steps to enumerate the MSSQL database. Then explore the records of the sysusers table inside the master database. What is the value of the first user listed?

✅ Step 1 — Connect to MSSQL from Kali
impacket-mssqlclient <username>:<password>@<target-ip>

✅ Step 2 — Enumerate databases
SELECT name FROM sys.databases

✅ Step 3 — Switch to the master database
USE master

✅ Step 4 — (Optional but good practice) List tables
SQLSELECT name FROM sys.tables
👉 Confirms accessible tables (though sysusers is a system table)

✅ Step 5 — Enumerate users from sysusers
SQLSELECT * FROM sysusers

✅ Step 6 — Identify the first user listed
Look at the first row of the output, for example:
uid   status   name
----  ------   -----
0     0        public   ← FIRST USER


✅ ✅ Final Answer

SQL (SQLPLAYGROUND\Administrator  dbo@master)> select uid,status,name from sysusers;
  uid   status   name                                
-----   ------   ---------------------------------   
    0        0   public                              
    1        0   dbo           

The value of the first user listed is:
public


⚠️ Important Exam Notes

✅ Always use SELECT * when asked for “first entry”
❌ Do NOT rely on:
SELECT name FROM sysusers;
Any output without guaranteed order
✅ SQL Server does not guarantee ordering unless ORDER BY is used

## EXAMPLE:

QUESTION:
From your Kali Linux VM, connect to the remote MySQL instance on VM 3 and explore the users table present in one of the databases to get the flag.

=== MySQL Enumeration – Exam Workflow ===

# Step 1 – Connect to MySQL
mysql -h <target-ip> -u <username> -p

## mysql -h 192.168.105.16 -P 3306 -u root -p'root' --skip-ssl-verify-server-cert

# Step 2 – List all databases
SHOW DATABASES;

# Step 3 – Identify non-system databases
# Ignore:
# information_schema
# mysql
# performance_schema
# sys
# Focus on:
# test (or any custom DB)

# Step 4 – Switch to target database
USE test;

# Step 5 – List tables
SHOW TABLES;

# Step 6 – Look for interesting tables
# Common names:
# users
# accounts
# credentials

# Step 7 – Dump table contents
SELECT * FROM users;

# Step 8 – Identify the flag in output

=== Example Result ===
OS{d00bd704b17b3259660c4ed38876da07}

=== Final Answer ===
OS{d00bd704b17b3259660c4ed38876da07}

---
%% graph-links %%
## Related
- [[UNION-based payloads]]
- [[Identifying SQLi via error-based payloads]]
- [[Blind SQL injections]]

> [!info] Navigation
> Section: [[SQL Injection Attacks/_index|SQL Injection Attacks]] · Home: [[🏠 Home]]


---
title: MySQL Database replication
layout: default
---

# MySQL Master-Slave Database Replication

Replicating the database to a slave instan
We might replicate the master database to a slave instance to maintai, or as a read-only database on which the application can run read (SELECT) query.

## Changes in the master instance

To replicate the master database, we need to make the following 3 changes in the master DB configuration file - `/etc/mysql/my.cnf`.

Change the interface on which the server listens. Out of the box, MySQL only listens on the localhost. For the slave instances to connect to it, it needs to listen on all network interfaces.

```
bind-address            = 0.0.0.0
bind-address            = 127.0.0.1
```

Uncomment the following 2 lines. The first line is used to identify the server. The second line enables the binary logging facility. Binary logs are used to copy replication steps from the master instance to the slave instances.

```
server-id               = 1
log_bin                 = /var/log/mysql/mysql-bin.log
```

Its highly recommended that you create a separate user for replication. It is not secure to use a privileged user to replicate the databases. Run the following MySQL statements to create an user.

```mysql
mysql> CREATE USER 'repluser'@'%' IDENTIFIED BY 'strongpassword';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repluser'@'%';
mysql> FLUSH PRIVILEGES;
```

Open a MySQL session to read-lock the entire database.


+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      496 
+------------------+----------+--------------+------------------+-------------------+

```mysql
mysql> FLUSH TABLES WITH READ LOCK;
```

In a separate terminal, open another session. Use this terminal to get the current binary logging position and to dump the database.

```bash
$ mysqldump --all-databases -uroot -p > alldb.sql
```

```mysql
mysql> SHOW MASTER STATUS;
```

+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      496 
+------------------+----------+--------------+------------------+-------------------+





Session 1:

```mysql
mysql> UNLOCK TABLES;
```

## Changes in the slave instance

Copy the dump of the master database to the slave server and load it.
```bash
$ mysql -uroot -p < alldb.sql
```
 
Slave:
Open vi /etc/mysql/my.cnf
Add this line:
```
server-id               = 2
```

```mysql
mysql> CHANGE MASTER TO
	MASTER_HOST='123.45.67.89',
	MASTER_USER='repluser',
	MASTER_PASSWORD='strongpassword',
	MASTER_LOG_FILE='mysql-bin.000001',
	MASTER_LOG_POS=496;
mysql> START SLAVE;
```

mysql> SHOW SLAVE STATUS;
mysql> SET GLOBAL SQL_SLAVE_SKIP_COUNTER = 1;
mysql> SLAVE START; 
# Домашнее задание 44: MySQL

## Задания
В материалах приложены ссылки на вагрант для репликации и дамп базы bet.dmp<br>

Базу развернуть на мастере и настроить так, чтобы реплицировались таблицы:<br>


| bookmaker          |<br>
| competition        |<br>
 market              |<br>
| odds               |<br>
| outcome<br>

Настроить GTID репликацию<br>

варианты которые принимаются к сдаче<br>

рабочий вагрантафайл<br>
скрины или логи SHOW TABLES<br>
конфиги*<br>

пример в логе изменения строки и появления строки на реплике*<br>



## Выполнение

### Я решила поднять MySQL на docker контейнерах с ОС Almalinux 9
### Настройка master ноды 
```
dilyam@MacBook-Pro-Dilya-2 project % docker exec -it dbmaster bash

[root@75214da549e2 /]# yum install -y https://repo.percona.com/yum/percona-release-latest.noarch.rpm

[root@75214da549e2 /]# percona-release setup ps80
* Disabling all Percona Repositories
* Enabling the Percona Server for MySQL 8.0 repository
<*> All done!

[root@75214da549e2 /]# yum install -y percona-server-server

dilyam@MacBook-Pro-Dilya-2 project % docker cp '/Users/dilyam/linux prof/Урок 44/project/stands-mysql/conf/conf.d/.' dbmaster:/etc/my.cnf.d/


[root@75214da549e2 /]# ls -la /etc/my.cnf.d
total 32
drwxr-xr-x 2 root root  4096 Sep 24 14:13 .
drwxr-xr-x 1 root root  4096 Sep 24 13:25 ..
-rw-r--r-- 1  501 games  207 Sep 24 14:03 01-base.cnf
-rw-r--r-- 1  501 games   48 Sep 24 14:03 02-max-connections.cnf
-rw-r--r-- 1  501 games  487 Sep 24 14:03 03-performance.cnf
-rw-r--r-- 1  501 games   66 Sep 24 14:03 04-slow-query.cnf
-rw-r--r-- 1  501 games  385 Sep 24 14:03 05-binlog.cnf

[root@75214da549e2 log]#  cat /etc/my.cnf
# Percona Server template configuration
#
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove the leading "# " to disable binary logging
# Binary logging captures changes between backups and is enabled by
# default. It's default setting is log_bin=binlog
# disable_log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
#
# Remove leading # to revert to previous value for default_authentication_plugin,
# this will increase compatibility with older clients. For background, see:
# https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin
# default-authentication-plugin=mysql_native_password

datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

!includedir /etc/my.cnf.d/

[root@75214da549e2 log]# mysqld --initialize --user=mysql


[root@75214da549e2 log]# cat /var/log/mysqld.log 
2026-09-24T14:29:59.904752Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.46-37) initializing of server in progress as process 4968
2026-09-24T14:29:59.918323Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2026-09-24T14:30:00.163505Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2026-09-24T14:30:00.591028Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: y4H&hjxCV6m3


[root@75214da549e2 log]# mysqld --user=mysql --daemonize
mysqld will log errors to /var/log/mysqld.log
mysqld is running as pid 5016


[root@75214da549e2 log]# mysql -uroot -p'y4H&hjxCV6m3'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.46-37

Copyright (c) 2009-2026 Percona LLC and/or its affiliates
Copyright (c) 2000, 2026, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER USER() IDENTIFIED BY 'YourStrongPassword';
Query OK, 0 rows affected (0.02 sec)

mysql>  SELECT @@server_id;
+-------------+
| @@server_id |
+-------------+
|           1 |
+-------------+
1 row in set (0.01 sec)


mysql> SHOW VARIABLES LIKE 'gtid_mode';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| gtid_mode     | ON    |
+---------------+-------+
1 row in set (0.01 sec)


[root@75214da549e2 /]# mysql -uroot -p -D bet < /root/bet-224190-d906e5.dmp
Enter password: 


[root@75214da549e2 /]# mysql -uroot -p'YourStrongPassword'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 8.0.46-37 Percona Server (GPL), Release 37, Revision 39e2b60e

Copyright (c) 2009-2026 Percona LLC and/or its affiliates
Copyright (c) 2000, 2026, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> USE bet;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SHOW TABLES;
+------------------+
| Tables_in_bet    |
+------------------+
| bookmaker        |
| competition      |
| events_on_demand |
| market           |
| odds             |
| outcome          |
| v_same_event     |
+------------------+
7 rows in set (0.00 sec)

mysql> 


mysql> CREATE USER 'repl'@'%' IDENTIFIED BY '!OtusLinux2018';
Query OK, 0 rows affected (0.03 sec)

mysql> SELECT user,host FROM mysql.user where user='repl';
+------+------+
| user | host |
+------+------+
| repl | %    |
+------+------+
1 row in set (0.00 sec)


mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
Query OK, 0 rows affected (0.01 sec)


mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)


mysql> SHOW GRANTS FOR 'repl'@'%';
+----------------------------------------------+
| Grants for repl@%                            |
+----------------------------------------------+
| GRANT REPLICATION SLAVE ON *.* TO `repl`@`%` |
+----------------------------------------------+
1 row in set (0.00 sec)



[root@75214da549e2 /]# mysqldump -uroot -p --single-transaction --routines --triggers --set-gtid-purged=ON bet > mybet.sql
Enter password: 
Warning: A partial dump from a server that has GTIDs will by default include the GTIDs of all transactions, even those that changed suppressed parts of the database. If you don't want to restore GTIDs, pass --set-gtid-purged=OFF. To make a complete dump, pass --all-databases --triggers --routines --events. 

[root@75214da549e2 /]# ls -la
total 1556
drwxr-xr-x   1 root root    4096 Sep 25 10:53 .
drwxr-xr-x   1 root root    4096 Sep 25 10:53 ..
-rwxr-xr-x   1 root root       0 Sep 24 13:06 .dockerenv
dr-xr-xr-x   2 root root    4096 Oct  2  2024 afs
lrwxrwxrwx   1 root root       7 Oct  2  2024 bin -> usr/bin
drwxr-xr-x   5 root root     360 Sep 25 08:16 dev
drwxr-xr-x   1 root root    4096 Sep 25 10:28 etc
drwxr-xr-x   2 root root    4096 Oct  2  2024 home
lrwxrwxrwx   1 root root       7 Oct  2  2024 lib -> usr/lib
lrwxrwxrwx   1 root root       9 Oct  2  2024 lib64 -> usr/lib64
-rw-r--r--   1 root root 1403699 Sep 24 15:11 master.sql
drwxr-xr-x   2 root root    4096 Oct  2  2024 media
drwxr-xr-x   2 root root    4096 Oct  2  2024 mnt
-rw-r--r--   1 root root  118139 Sep 25 10:53 mybet.sql


```

### Скопировала бекап БД
```
dilyam@MacBook-Pro-Dilya-2 project % docker cp dbmaster:/mybet.sql ./mybet.sql
Successfully copied 120kB to /Users/dilyam/linux prof/Урок 44/project/mybet.sql

dilyam@MacBook-Pro-Dilya-2 project % docker cp ./mybet.sql dbslave:/mybet.sql
Successfully copied 120kB to dbslave:/mybet.sql
```


### Настройка slave ноды
```
dilyam@MacBook-Pro-Dilya-2 project % docker exec -it dbslave bash 

[root@f926ff704200 /]# yum install -y https://repo.percona.com/yum/percona-release-latest.noarch.rpm

[root@f926ff704200 /]# percona-release setup ps80
* Disabling all Percona Repositories
* Enabling the Percona Server for MySQL 8.0 repository
<*> All done!

[root@f926ff704200 /]# yum install -y percona-server-server

[root@f926ff704200 /]# ls -la /etc/my.cnf.d/
total 32
drwxr-xr-x 2 root root  4096 Sep 25 09:10 .
drwxr-xr-x 1 root root  4096 Sep 24 13:24 ..
-rw-r--r-- 1  501 games  207 Sep 25 09:10 01-base.cnf
-rw-r--r-- 1  501 games   48 Sep 24 14:03 02-max-connections.cnf
-rw-r--r-- 1  501 games  487 Sep 24 14:03 03-performance.cnf
-rw-r--r-- 1  501 games   66 Sep 24 14:03 04-slow-query.cnf
-rw-r--r-- 1  501 games  385 Sep 24 14:03 05-binlog.cnf

[root@f926ff704200 /]# mysqld --initialize --user=mysql

[root@f926ff704200 /]# cat /var/log/mysqld.log
2026-09-25T09:21:49.216619Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.46-37) initializing of server in progress as process 44
2026-09-25T09:21:49.236926Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2026-09-25T09:21:49.428188Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2026-09-25T09:21:49.941348Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: gGNkB6O(<N6*

[root@f926ff704200 /]# mysqld --user=mysql --daemonize
mysqld will log errors to /var/log/mysqld.log
mysqld is running as pid 90

[root@f926ff704200 /]# mysql -uroot -p'gGNkB6O(<N6*'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.46-37

Copyright (c) 2009-2026 Percona LLC and/or its affiliates
Copyright (c) 2000, 2026, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.



[root@f926ff704200 /]# cat /etc/my.cnf.d/01-base.cnf 
[mysqld]
pid-file=/var/run/mysqld/mysqld.pid
log-error=/var/log/mysqld.log
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
symbolic-links=0

server-id = 2
innodb_file_per_table = 1
skip-name-resolve



[root@f926ff704200 /]# mysqld --verbose --help 2>/dev/null | grep -A2 "Default options"
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf 
The following groups are read: mysql_cluster mysqld server mysqld-8.0

[root@f926ff704200 /]# vi /etc/my.cnf.d/03-performance.cnf 
[root@f926ff704200 /]# vi /etc/my.cnf

[root@f926ff704200 /]# cat /etc/my.cnf.d/03-performance.cnf 
[mysqld]
skip-external-locking
key-buffer-size = 384M
max-allowed-packet = 16M
table-open-cache = 5000
sort-buffer-size = 64M
join-buffer-size = 64M
read-buffer-size = 2M
read-rnd-buffer-size = 8M
myisam-sort-buffer-size = 64M
thread-cache-size = 8
tmp-table-size = 1024M
max-heap-table-size = 1024M
#thread-concurrency = 8 # Из за этого параметра на Vagrant-овской виртуалке mysql не взлетает



[root@f926ff704200 /]# cat /etc/my.cnf
# Percona Server template configuration
#
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove the leading "# " to disable binary logging
# Binary logging captures changes between backups and is enabled by
# default. It's default setting is log_bin=binlog
# disable_log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
#
# Remove leading # to revert to previous value for default_authentication_plugin,
# this will increase compatibility with older clients. For background, see:
# https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin
# default-authentication-plugin=mysql_native_password

datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

!includedir /etc/my.cnf.d/



[root@f926ff704200 /]# mysqladmin -uroot -p'YourStrongPassword' shutdown
mysqladmin: [Warning] Using a password on the command line interface can be insecure.

[root@f926ff704200 /]# mysqld --user=mysql --daemonize
mysqld will log errors to /var/log/mysqld.log
mysqld is running as pid 155

[root@f926ff704200 /]# mysql -uroot -p'YourStrongPassword'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.46-37 Percona Server (GPL), Release 37, Revision 39e2b60e

Copyright (c) 2009-2026 Percona LLC and/or its affiliates
Copyright (c) 2000, 2026, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SELECT @@server_id;
+-------------+
| @@server_id |
+-------------+
|           2 |
+-------------+
1 row in set (0.00 sec)

mysql> SHOW VARIABLES LIKE 'gtid_mode';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| gtid_mode     | ON    |
+---------------+-------+
1 row in set (0.01 sec)

mysql> 
```


### Поднимаем БД с бекапа и настраиваем репликацию
```
mysql> CREATE DATABASE bet;
Query OK, 1 row affected (0.01 sec)

mysql> USE bet;
Database changed

mysql> SOURCE /mybet.sql

mysql> SHOW TABLES;
+------------------+
| Tables_in_bet    |
+------------------+
| bookmaker        |
| competition      |
| events_on_demand |
| market           |
| odds             |
| outcome          |
| v_same_event     |
+------------------+
7 rows in set (0.00 sec)

mysql> 


mysql> CHANGE REPLICATION SOURCE TO
  SOURCE_HOST='172.19.0.6',
  SOURCE_USER='repl',
  SOURCE_PASSWORD='!OtusLinux2018',
  SOURCE_AUTO_POSITION=1,
  GET_SOURCE_PUBLIC_KEY=1;
Query OK, 0 rows affected, 2 warnings (0.03 sec)

mysql> START REPLICA;
Query OK, 0 rows affected (0.05 sec)

mysql> SHOW REPLICA STATUS\G
*************************** 1. row ***************************
             Replica_IO_State: Waiting for source to send event
                  Source_Host: 172.19.0.6
                  Source_User: repl
                  Source_Port: 3306
                Connect_Retry: 60
              Source_Log_File: mysql-bin.000001
          Read_Source_Log_Pos: 475
               Relay_Log_File: f926ff704200-relay-bin.000002
                Relay_Log_Pos: 420
        Relay_Source_Log_File: mysql-bin.000001
           Replica_IO_Running: Yes
          Replica_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: bet.events_on_demand,bet.v_same_event
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Source_Log_Pos: 475
              Relay_Log_Space: 637
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Source_SSL_Allowed: No
           Source_SSL_CA_File: 
           Source_SSL_CA_Path: 
              Source_SSL_Cert: 
            Source_SSL_Cipher: 
               Source_SSL_Key: 
        Seconds_Behind_Source: 0
Source_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Source_Server_Id: 1
                  Source_UUID: 6c37dcfc-b824-11f1-93b2-be7bd0e5280e
             Source_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
    Replica_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Source_Retry_Count: 86400
                  Source_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Source_SSL_Crl: 
           Source_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 6c37dcfc-b824-11f1-93b2-be7bd0e5280e:1-41,
895305e2-b8c2-11f1-91a9-7ad6d108990b:1-2
                Auto_Position: 1
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Source_TLS_Version: 
       Source_public_key_path: 
        Get_Source_public_key: 1
            Network_Namespace: 
1 row in set (0.00 sec)

mysql> 
```


### Делаем изменение в БД на master ноде
```
mysql> USE bet;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SHOW TABLES;
+------------------+
| Tables_in_bet    |
+------------------+
| bookmaker        |
| competition      |
| events_on_demand |
| market           |
| odds             |
| outcome          |
| v_same_event     |
+------------------+
7 rows in set (0.01 sec)

mysql> SELECT * FROM bookmaker;
+----+----------------+
| id | bookmaker_name |
+----+----------------+
|  2 | 100xbet        |
|  1 | 1xbet          |
|  4 | betway         |
|  5 | bwin           |
|  6 | ladbrokes      |
|  3 | unibet         |
+----+----------------+
6 rows in set (0.00 sec)


mysql> INSERT INTO bookmaker (id,bookmaker_name) VALUES(7,'info');
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM bookmaker;
+----+----------------+
| id | bookmaker_name |
+----+----------------+
|  2 | 100xbet        |
|  1 | 1xbet          |
|  4 | betway         |
|  5 | bwin           |
|  7 | info           |
|  6 | ladbrokes      |
|  3 | unibet         |
+----+----------------+
7 rows in set (0.00 sec)

mysql> 

```



### Проверяем работу репликации
```
mysql> SELECT @@GLOBAL.gtid_executed, @@GLOBAL.gtid_purged;
+-------------------------------------------------------------------------------------+-------------------------------------------+
| @@GLOBAL.gtid_executed                                                              | @@GLOBAL.gtid_purged                      |
+-------------------------------------------------------------------------------------+-------------------------------------------+
| 6c37dcfc-b824-11f1-93b2-be7bd0e5280e:1-42,
895305e2-b8c2-11f1-91a9-7ad6d108990b:1-2 | 6c37dcfc-b824-11f1-93b2-be7bd0e5280e:1-41 |
+-------------------------------------------------------------------------------------+-------------------------------------------+
1 row in set (0.01 sec)

mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 172.19.0.6
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 790
               Relay_Log_File: f926ff704200-relay-bin.000002
                Relay_Log_Pos: 735
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: bet.events_on_demand,bet.v_same_event
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 790
              Relay_Log_Space: 952
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: 6c37dcfc-b824-11f1-93b2-be7bd0e5280e
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 6c37dcfc-b824-11f1-93b2-be7bd0e5280e:42
            Executed_Gtid_Set: 6c37dcfc-b824-11f1-93b2-be7bd0e5280e:1-42,
895305e2-b8c2-11f1-91a9-7ad6d108990b:1-2
                Auto_Position: 1
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
       Master_public_key_path: 
        Get_master_public_key: 1
            Network_Namespace: 
1 row in set, 1 warning (0.00 sec)

mysql> SELECT * FROM bookmaker;                  
+----+----------------+
| id | bookmaker_name |
+----+----------------+
|  2 | 100xbet        |
|  1 | 1xbet          |
|  4 | betway         |
|  5 | bwin           |
|  7 | info           |
|  6 | ladbrokes      |
|  3 | unibet         |
+----+----------------+
7 rows in set (0.01 sec)

mysql> 

```

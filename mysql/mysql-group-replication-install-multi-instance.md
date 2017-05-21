# MySQL Group Replication （组复制）单机多实例安装笔记

MySQL Group Replication （组复制）单机多实例安装非常简单。


实验的[机器环境][1]如下。

```
HOSTNAME: svr-147
IP: 192.168.128.147
CPU model: model name : Intel Core i7 9xx (Nehalem Class Core i7)
CPU processor: 4
MemTotal: 8059376 kB
MemFree: 331012 kB
OS: Red Hat Enterprise Linux Server release 6.6 (Santiago)
Load Average: 0.00 0.01 0.00 1/713 4143
Core: Linux svr-147 2.6.32-504.el6.x86_64 #1 SMP Tue Sep 16 01:56:35 EDT 2014 x86_64 x86_64 x86_64 GNU/Linux
```

安装三个实例：

port | datadir | conf | replication server_id | replication port | 
----|----|----|----|----|
3306 | /data/mysql/mysql_3306/data | /data/mysql/mysql_3306/my3306.cnf | 1013306 | 33061 | 
3307 | /data/mysql/mysql_3307/data | /data/mysql/mysql_3307/my3307.cnf | 1013307 | 33071 | 
3308 | /data/mysql/mysql_3308/data | /data/mysql/mysql_3308/my3308.cnf | 1013308 | 33081 | 

## MySQL 二进制文件准备

```
# mkdir /opt/apps
# cd /opt/apps/
# wget http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.18-linux-glibc2.5-x86_64.tar.gz
# ll -h
-rw-r--r-- 1 root hadoop 625M 5月  19 17:43 mysql-5.7.18-linux-glibc2.5-x86_64.tar.gz
# tar zxvf mysql-5.7.18-linux-glibc2.5-x86_64.tar.gz
# ln -s /opt/apps/mysql-5.7.18-linux-glibc2.5-x86_64 /usr/local/mysql
```

## Group Replication 多实例目录准备

```
# mkdir -p /data/mysql/{mysql_3306,mysql_3307,mysql_3308}/{data,logs,tmp}
# chown -R mysql.mysql /data/mysql/
```

## 创建每个实例的配置文件

```
# vim /data/mysql/mysql_3306/my3306.cnf
# vim /data/mysql/mysql_3307/my3307.cnf
# vim /data/mysql/mysql_3308/my3308.cnf

```

三个实例的配置文件内容放在了文章最后。



## 启动一个 MySQL 实例

使用`--defaults-file`参数以指定配置文件位置的方式**初始化**一个MySQL实例。

```
# /usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/mysql_3306/my3306.cnf --initialize-insecure
```

使用`--defaults-file`参数启动一个MySQL实例，进入MySQL客户端也需要指定配置文件。

```
# /usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/mysql_3306/my3306.cnf &
[1] 11290
# pidof mysqld
11290

# /usr/local/mysql/bin/mysql --defaults-file=/data/mysql/mysql_3306/my3306.cnf
Welcome to the MySQL monitor.  Commands end with ; or \g.
...
mysql [(none)]>show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)

mysql [(none)]>quit
Bye
```



## 启动 Group Replication 的第一个节点

编写 `/opt/apps/create_rpl_user.sql` 脚本，为 group replication 创建内部通信用户。

```
# cat /opt/apps/create_rpl_user.sql
SET SQL_LOG_BIN=0;
CREATE USER rpl_user@'%';
GRANT REPLICATION SLAVE ON *.* TO rpl_user@'%' IDENTIFIED BY 'rpl_pass';
SET SQL_LOG_BIN=1;
CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='rpl_pass' FOR CHANNEL 'group_replication_recovery';

# /usr/local/mysql/bin/mysql --defaults-file=/data/mysql/mysql_3306/my3306.cnf < /opt/apps/create_rpl_user.sql

# /usr/local/mysql/bin/mysql --defaults-file=/data/mysql/mysql_3306/my3306.cnf -e " select host, user from mysql.user" 
+-----------+-----------+
| host      | user      |
+-----------+-----------+
| %         | rpl_user  |
| localhost | mysql.sys |
| localhost | root      |
+-----------+-----------+

```

为上述已经启动的MySQL实例安装 Group Replication 插件。

```
# /usr/local/mysql/bin/mysql --defaults-file=/data/mysql/mysql_3306/my3306.cnf

mysql [(none)]>INSTALL PLUGIN group_replication SONAME 'group_replication.so';
Query OK, 0 rows affected (0.20 sec)

mysql [(none)]>SHOW PLUGINS;
+----------------------------+----------+--------------------+----------------------+---------+
| Name                       | Status   | Type               | Library              | License |
+----------------------------+----------+--------------------+----------------------+---------+
| binlog                     | ACTIVE   | STORAGE ENGINE     | NULL                 | GPL     |
| mysql_native_password      | ACTIVE   | AUTHENTICATION     | NULL                 | GPL     |
| .....                      | ......   | ........           | ....                 | GPL     |
| group_replication          | ACTIVE   | GROUP REPLICATION  | group_replication.so | GPL     |
+----------------------------+----------+--------------------+----------------------+---------+
```

启动 Group Replication 的第一个节点。
`SET GLOBAL group_replication_bootstrap_group=ON;`只在第一个节点上执行。

```
mysql [(none)]>SET GLOBAL group_replication_bootstrap_group=ON;
Query OK, 0 rows affected (0.00 sec)

mysql [(none)]>START GROUP_REPLICATION;
Query OK, 0 rows affected (1.52 sec)

mysql [(none)]>SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| group_replication_applier | d0319486-3e13-11e7-b2da-5254001ecdb7 | svr-146     |        3306 | ONLINE       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
1 row in set (0.03 sec)

mysql [(none)]>
```

## 启动 Group Replication 剩余节点

```
# /usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/mysql_3307/my3307.cnf --initialize-insecure
# /usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/mysql_3307/my3307.cnf &
# pidof mysqld
14461 11290

# /usr/local/mysql/bin/mysql --defaults-file=/data/mysql/mysql_3307/my3307.cnf < /opt/apps/create_rpl_user.sql

# /usr/local/mysql/bin/mysql --defaults-file=/data/mysql/mysql_3307/my3307.cnf

mysql [(none)]>INSTALL PLUGIN group_replication SONAME 'group_replication.so';
mysql [(none)]>START GROUP_REPLICATION;

mysql [(none)]>SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| group_replication_applier | d0319486-3e13-11e7-b2da-5254001ecdb7 | svr-146     |        3306 | ONLINE       |
| group_replication_applier | ee095ba0-3e18-11e7-84ab-5254001ecdb7 | svr-146     |        3307 | ONLINE       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+



# /usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/mysql_3308/my3308.cnf --initialize-insecure
# /usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/mysql_3308/my3308.cnf &
[3] 15431
# pidof mysqld
15431 14461 11290
# /usr/local/mysql/bin/mysql --defaults-file=/data/mysql/mysql_3308/my3308.cnf < /opt/apps/create_rpl_user.sql

# /usr/local/mysql/bin/mysql --defaults-file=/data/mysql/mysql_3308/my3308.cnf

mysql [(none)]>INSTALL PLUGIN group_replication SONAME 'group_replication.so';
mysql [(none)]>START GROUP_REPLICATION;

mysql [(none)]>SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| group_replication_applier | 3d8cbe36-3e1a-11e7-be5a-5254001ecdb7 | svr-146     |        3308 | ONLINE       |
| group_replication_applier | d0319486-3e13-11e7-b2da-5254001ecdb7 | svr-146     |        3306 | ONLINE       |
| group_replication_applier | ee095ba0-3e18-11e7-84ab-5254001ecdb7 | svr-146     |        3307 | ONLINE       |
+-------------------------- -+--------------------------------------+-------------+-------------+--------------+

```


## 实例配置文件

### 3306实例

实例注意区分端口和目录路径。

```
[client]
port            = 3306
socket          = /tmp/mysql3306.sock

[mysql]
prompt                         = mysql [\d]>
default_character_set          = utf8
no-auto-rehash

[mysqld]
#misc
user = mysql
basedir = /usr/local/mysql
datadir = /data/mysql/mysql_3306/data
port = 3306
socket = /tmp/mysql3306.sock
event_scheduler = 0

tmpdir=/data/mysql/mysql_3306/tmp
#timeout
interactive_timeout = 43200
wait_timeout = 43200

#character set
character-set-server = utf8

open_files_limit = 65535
max_connections = 100
max_connect_errors = 100000
#
explicit_defaults_for_timestamp
#logs
log-output=file
slow_query_log = 1
slow_query_log_file = slow.log
log-error = error.log
log_error_verbosity=3
pid-file = mysql.pid
long_query_time = 1
#log-slow-admin-statements = 1
#log-queries-not-using-indexes = 1
log-slow-slave-statements = 1

#binlog
binlog_format = row
server-id = 1343306
log-bin = /data/mysql/mysql_3306/logs/mysql-bin
binlog_cache_size = 1M
max_binlog_size = 200M
max_binlog_cache_size = 2G
sync_binlog = 0
expire_logs_days = 10


#group replication
server_id=1013306
gtid_mode=ON
enforce_gtid_consistency=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
binlog_checksum=NONE
log_slave_updates=ON
binlog_format=ROW

transaction_write_set_extraction=XXHASH64
loose-group_replication_group_name="13855fca-d2ab-11e6-8f37-005056b8286c"
loose-group_replication_start_on_boot=off
loose-group_replication_local_address= "192.168.128.147:33061"
loose-group_replication_group_seeds= "192.168.128.147:33061,192.168.128.147:33071,192.168.128.147:33081"
loose-group_replication_bootstrap_group= off

loose-group_replication_single_primary_mode=off
loose-group_replication_enforce_update_everywhere_checks=on

#relay log
skip_slave_start = 1
max_relay_log_size = 500M
relay_log_purge = 1
relay_log_recovery = 1
#slave-skip-errors=1032,1053,1062

#buffers & cache
table_open_cache = 2048
table_definition_cache = 2048
table_open_cache = 2048
max_heap_table_size = 96M
sort_buffer_size = 2M
join_buffer_size = 2M
thread_cache_size = 256
query_cache_size = 0
query_cache_type = 0
query_cache_limit = 256K
query_cache_min_res_unit = 512
thread_stack = 192K
tmp_table_size = 96M
key_buffer_size = 8M
read_buffer_size = 2M
read_rnd_buffer_size = 16M
bulk_insert_buffer_size = 32M

#myisam
myisam_sort_buffer_size = 128M
myisam_max_sort_file_size = 10G
myisam_repair_threads = 1

#innodb
innodb_buffer_pool_size = 500M
innodb_buffer_pool_instances = 1
innodb_data_file_path = ibdata1:100M:autoextend
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 64M
innodb_log_file_size = 256M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_file_per_table = 1
innodb_rollback_on_timeout
innodb_status_file = 1
innodb_io_capacity = 2000
transaction_isolation = READ-COMMITTED
innodb_flush_method = O_DIRECT
```

### 3307实例

```
[client]
port            = 3307
socket          = /tmp/mysql3307.sock

[mysql]
prompt                         = mysql [\d]>
default_character_set          = utf8
no-auto-rehash

[mysqld]
#misc
user = mysql
basedir = /usr/local/mysql
datadir = /data/mysql/mysql_3307/data
port = 3307
socket = /tmp/mysql3307.sock
event_scheduler = 0

tmpdir=/data/mysql/mysql_3307/tmp
#timeout
interactive_timeout = 43200
wait_timeout = 43200

#character set
character-set-server = utf8

open_files_limit = 65535
max_connections = 100
max_connect_errors = 100000
#
explicit_defaults_for_timestamp
#logs
log-output=file
slow_query_log = 1
slow_query_log_file = slow.log
log-error = error.log
log_error_verbosity=3
pid-file = mysql.pid
long_query_time = 1
#log-slow-admin-statements = 1
#log-queries-not-using-indexes = 1
log-slow-slave-statements = 1

#binlog
binlog_format = row
server-id = 1343307
log-bin = /data/mysql/mysql_3307/logs/mysql-bin
binlog_cache_size = 1M
max_binlog_size = 200M
max_binlog_cache_size = 2G
sync_binlog = 0
expire_logs_days = 10


#group replication
server_id=1013307
gtid_mode=ON
enforce_gtid_consistency=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
binlog_checksum=NONE
log_slave_updates=ON
binlog_format=ROW

transaction_write_set_extraction=XXHASH64
loose-group_replication_group_name="13855fca-d2ab-11e6-8f37-005056b8286c"
loose-group_replication_start_on_boot=off
loose-group_replication_local_address= "192.168.128.147:33071"
loose-group_replication_group_seeds= "192.168.128.147:33061,192.168.128.147:33071,192.168.128.147:33081"
loose-group_replication_bootstrap_group= off

loose-group_replication_single_primary_mode=off
loose-group_replication_enforce_update_everywhere_checks=on

#relay log
skip_slave_start = 1
max_relay_log_size = 500M
relay_log_purge = 1
relay_log_recovery = 1
#slave-skip-errors=1032,1053,1062

#buffers & cache
table_open_cache = 2048
table_definition_cache = 2048
table_open_cache = 2048
max_heap_table_size = 96M
sort_buffer_size = 2M
join_buffer_size = 2M
thread_cache_size = 256
query_cache_size = 0
query_cache_type = 0
query_cache_limit = 256K
query_cache_min_res_unit = 512
thread_stack = 192K
tmp_table_size = 96M
key_buffer_size = 8M
read_buffer_size = 2M
read_rnd_buffer_size = 16M
bulk_insert_buffer_size = 32M

#myisam
myisam_sort_buffer_size = 128M
myisam_max_sort_file_size = 10G
myisam_repair_threads = 1

#innodb
innodb_buffer_pool_size = 500M
innodb_buffer_pool_instances = 1
innodb_data_file_path = ibdata1:100M:autoextend
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 64M
innodb_log_file_size = 256M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_file_per_table = 1
innodb_rollback_on_timeout
innodb_status_file = 1
innodb_io_capacity = 2000
transaction_isolation = READ-COMMITTED
innodb_flush_method = O_DIRECT
```

### 3308实例

```
[client]
port            = 3308
socket          = /tmp/mysql3308.sock

[mysql]
prompt                         = mysql [\d]>
default_character_set          = utf8
no-auto-rehash

[mysqld]
#misc
user = mysql
basedir = /usr/local/mysql
datadir = /data/mysql/mysql_3308/data
port = 3308
socket = /tmp/mysql3308.sock
event_scheduler = 0

tmpdir=/data/mysql/mysql_3308/tmp
#timeout
interactive_timeout = 43200
wait_timeout = 43200

#character set
character-set-server = utf8

open_files_limit = 65535
max_connections = 100
max_connect_errors = 100000
#
explicit_defaults_for_timestamp
#logs
log-output=file
slow_query_log = 1
slow_query_log_file = slow.log
log-error = error.log
log_error_verbosity=3
pid-file = mysql.pid
long_query_time = 1
#log-slow-admin-statements = 1
#log-queries-not-using-indexes = 1
log-slow-slave-statements = 1

#binlog
binlog_format = row
server-id = 1343308
log-bin = /data/mysql/mysql_3308/logs/mysql-bin
binlog_cache_size = 1M
max_binlog_size = 200M
max_binlog_cache_size = 2G
sync_binlog = 0
expire_logs_days = 10


#group replication
server_id=1013308
gtid_mode=ON
enforce_gtid_consistency=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
binlog_checksum=NONE
log_slave_updates=ON
binlog_format=ROW

transaction_write_set_extraction=XXHASH64
loose-group_replication_group_name="13855fca-d2ab-11e6-8f37-005056b8286c"
loose-group_replication_start_on_boot=off
loose-group_replication_local_address= "192.168.128.147:33081"
loose-group_replication_group_seeds= "192.168.128.147:33061,192.168.128.147:33071,192.168.128.147:33081"
loose-group_replication_bootstrap_group= off

loose-group_replication_single_primary_mode=off
loose-group_replication_enforce_update_everywhere_checks=on

#relay log
skip_slave_start = 1
max_relay_log_size = 500M
relay_log_purge = 1
relay_log_recovery = 1
#slave-skip-errors=1032,1053,1062

#buffers & cache
table_open_cache = 2048
table_definition_cache = 2048
table_open_cache = 2048
max_heap_table_size = 96M
sort_buffer_size = 2M
join_buffer_size = 2M
thread_cache_size = 256
query_cache_size = 0
query_cache_type = 0
query_cache_limit = 256K
query_cache_min_res_unit = 512
thread_stack = 192K
tmp_table_size = 96M
key_buffer_size = 8M
read_buffer_size = 2M
read_rnd_buffer_size = 16M
bulk_insert_buffer_size = 32M

#myisam
myisam_sort_buffer_size = 128M
myisam_max_sort_file_size = 10G
myisam_repair_threads = 1

#innodb
innodb_buffer_pool_size = 500M
innodb_buffer_pool_instances = 1
innodb_data_file_path = ibdata1:100M:autoextend
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 64M
innodb_log_file_size = 256M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_file_per_table = 1
innodb_rollback_on_timeout
innodb_status_file = 1
innodb_io_capacity = 2000
transaction_isolation = READ-COMMITTED
innodb_flush_method = O_DIRECT
```

## 功能测试

为了方便操作，三个实例都配置下alias别名，方便访问。

```
# vim ~/.bash_profile 

alias mysql3308="/usr/local/mysql/bin/mysql --defaults-file=/data/mysql/mysql_3308/my3308.cnf"
alias mysql3307="/usr/local/mysql/bin/mysql --defaults-file=/data/mysql/mysql_3307/my3307.cnf"
alias mysql3306="/usr/local/mysql/bin/mysql --defaults-file=/data/mysql/mysql_3306/my3306.cnf"

# source .bash_profile
```

测试复制

```
# mysql3306 -e "show databases;"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
# mysql3307 -e "show databases;"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
# mysql3308 -e "show databases;"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
# mysql3306 -e "create database test;"
# mysql3307 -e "show databases;"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
# mysql3308 -e "show databases;"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
# mysql3306 -e "use test; create table employee(id int not null, name varchar(32), primary key(id)); insert into employee(id, name) values(1, 'futeng');"
# mysql3306 -e "select * from test.employee;"
+----+--------+
| id | name   |
+----+--------+
|  1 | futeng |
+----+--------+
# mysql3307 -e "select * from test.employee;"
+----+--------+
| id | name   |
+----+--------+
|  1 | futeng |
+----+--------+
# mysql3308 -e "select * from test.employee;"
+----+--------+
| id | name   |
+----+--------+
|  1 | futeng |
+----+--------+
# mysql3308 -e "drop table test.employee;"
# mysql3307 -e "select * from test.employee;"
ERROR 1146 (42S02) at line 1: Table 'test.employee' doesn't exist
# mysql3308 -e "use test; create table employee(id int not null, name varchar(32), primary key(id)); insert into employee(id, name) values(1, 'futeng');"
# mysql3307 -e "select * from test.employee;"
+----+--------+
| id | name   |
+----+--------+
|  1 | futeng |
+----+--------+

```


[1]:https://gist.github.com/futeng/5b5906b7b74184e71c11c8970da51278

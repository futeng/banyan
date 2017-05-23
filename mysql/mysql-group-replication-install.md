# MySQL Group Replication实例安装

MySQL Group Replication （组复制）安装非常简单。

单机也可以模拟（[MySQL Group Replication （组复制）单机多实例安装笔记][2]）。

本例是将单机安装的步骤封装成脚本来批量安装。

安装三个实例：

port | datadir | conf | replication server_id | replication port | 
----|----|----|----|----|
192.168.128.130:3306 | /data/mysql/mysql_3306/data | /data/mysql/mysql_3306/my3306.cnf | 1013306 | 33061 | 
192.168.128.131:3307 | /data/mysql/mysql_3307/data | /data/mysql/mysql_3307/my3307.cnf | 1013307 | 33071 | 
192.168.128.132:3308 | /data/mysql/mysql_3308/data | /data/mysql/mysql_3308/my3308.cnf | 1013308 | 33081 | 

## 环境准备

可执行 `getMySQLGroupReplicationScripts.sh` 脚本，从我的Github Gist下载文件：

1.  `mgr-init.sh`：用于设置datadir,tmp等目录；
2.  `mgr-start.sh`：初始化 mysql ；
3.  `create_rpl_user.sql` ：创建 mysql group replication 通信用户；
4.  `my3306.cnf`
5.  `my3307.cnf`
6.  `my3308.cnf`
7.  `mysql-5.7.18-linux-glibc2.5-x86_64.tar.gz`：MySQL二进制安装包。

以上所有文件需要放在同一个目录下。


```
# wget https://gist.githubusercontent.com/futeng/7e4e7c15ec53d09290437ea3aa2d4f36/raw/9b6640d2f8dd57fea6899d6ba80f5d7008011e61/getMySQLGroupReplicationScripts.sh

# ./getMySQLGRInstallScript.sh

# tree .
.
├── create_rpl_user.sql
├── getMySQLGRInstallScript.sh
├── mgr-init.sh
├── mgr-start.sh
├── my3306.cnf
├── my3307.cnf
├── my3308.cnf
└── mysql-5.7.18-linux-glibc2.5-x86_64.tar.gz

0 directories, 8 files
```

## 安装主节点

第一个安装的节点多一个配置，暂且称之为主节点，本例为 `192.168.128.130:3306`.

```
# 1. 创建以3306命名的目录，并解压mysql二进制压缩包
# ./mgr-init.sh 3306

# 2. 修改配置文件，注意IP应该是不同的
# vim ./my3306.cnf
# mv ./my3306.cnf /data/mysql/mysql_3306/my3306.cnf

# 3. 初始化MySQL，会打印出 `rpl_user` 用户信息
# ./mgr-start.sh 3306
```

以上步骤将完成MySQL的搭建，在剩余节点均如此安装，注意更换端口。

以下步骤将开启 `group_replication ` 插件。

```
[root@bd-130 ~]# /usr/local/mysql/bin/mysql --defaults-file=/data/mysql/mysql_3306/my3306.cnf
mysql [(none)]>INSTALL PLUGIN group_replication SONAME 'group_replication.so’;

mysql [(none)]>SET GLOBAL group_replication_bootstrap_group=ON;
Query OK, 0 rows affected (0.00 sec)

mysql [(none)]>START GROUP_REPLICATION;
Query OK, 0 rows affected (1.57 sec)

mysql [(none)]>SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| group_replication_applier | d13ca853-3f96-11e7-acad-525400532045 | bd-130      |        3306 | ONLINE       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
1 row in set (0.01 sec)
```


## 安装剩余节点

关于MySQL的搭建同第一个步骤。

开启 `group_replication ` 插件，也仅仅是少其中一步。


```
[root@bd-132 ~]# /usr/local/mysql/bin/mysql --defaults-file=/data/mysql/mysql_3308/my3308.cnf
mysql [(none)]>INSTALL PLUGIN group_replication SONAME 'group_replication.so’;

mysql [(none)]>START GROUP_REPLICATION;
Query OK, 0 rows affected (1.58 sec)

mysql [(none)]>SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
| group_replication_applier | 53c86fe8-3f9a-11e7-873d-5254008ec9bb | bd-131      |        3307 | ONLINE       |
| group_replication_applier | caf7d4fc-3f9b-11e7-a47d-5254005cddcf | bd-132      |        3308 | ONLINE       |
| group_replication_applier | d13ca853-3f96-11e7-acad-525400532045 | bd-130      |        3306 | ONLINE       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+
3 rows in set (0.00 sec)
```

## 测试

默认配置下任意节点都可写可读。

简单在任意节点创建数据库DB、表和插入数据等，都可以在其他节点几乎同步看到。

功能测试很简单，就不在贴图。性能测试可使用 sysbench。


[1]:https://gist.github.com/futeng/5b5906b7b74184e71c11c8970da51278
[2]:./mysql-group-replication-install-multi-instance.html
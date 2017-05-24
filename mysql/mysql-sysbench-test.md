# 使用Sysbench 测试MySQL性能


使用Sysbench测试MySQL性能。

环境简述（详细内容见附录一）：

机器 | 用途 | 性能
----|----|----
svr-146 | Sysbench所在服务器 | 性能一般，同台机器不影响测试结论
bd-133 | 5.6.16 MySQL 单实例数据库 | 性能与Group组相同，且内存闲置较多
bd-130,bd-131,bd-132 | 5.7.18 MySQL Group Replication测试组机器 | 性能与单实例组相同，但是内存紧张

[MySQL Group Replication 环境如下][3]：

port | datadir | conf | replication server_id | replication port | 
----|----|----|----|----|
192.168.128.130:3306 | /data/mysql/mysql_3306/data | /data/mysql/mysql_3306/my3306.cnf | 1013306 | 33061 | 
192.168.128.131:3307 | /data/mysql/mysql_3307/data | /data/mysql/mysql_3307/my3307.cnf | 1013307 | 33071 | 
192.168.128.132:3308 | /data/mysql/mysql_3308/data | /data/mysql/mysql_3308/my3308.cnf | 1013308 | 33081 | 

Sysbench 测试结论（详细内容见附录二）：

测试对象平均响应时间(ms) | CPU | 内存随机读写 | 内存顺序读写| FileIO 随机读写 | FileIO 顺序读写
----|----|----|----|----|----
单实例 | 5.39 | 0.02 | 0| 53.03| 6.58
Group Replication |5.39 |0.02 | 0| 54.23 | 5.45

**结论** ：MySQL Group Replication 方式与单机服务版本比较起来性能略低，但微乎其微。


## 附录一 安装 Sysbench

[官方安装文档][2]

```
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh | sudo bash
sudo yum -y install sysbench

# sysbench --version
sysbench 1.0.7
```

## 附录二 测试环境说明

[环境介绍][1]

### Sysbenc所在机器

```
[root@svr-146 ft]# ./getHostInfo.sh
Core: Linux svr-146 2.6.32-504.el6.x86_64 #1 SMP Tue Sep 16 01:56:35 EDT 2014 x86_64 x86_64 x86_64 GNU/Linux
OS: Red Hat Enterprise Linux Server release 6.6 (Santiago)
CPU model: model name : Intel Core i7 9xx (Nehalem Class Core i7)
CPU processor: 4
MemTotal: 8059376 kB
MemFree: 205612 kB
hostname: svr-146
IP: 192.168.128.146
Load Average: 0.15 0.04 0.01 2/693 8986
```

### MySQL Group Replication 所在服务器环境

```
mysql [sbtest]>status
--------------
/usr/local/mysql/bin/mysql  Ver 14.14 Distrib 5.7.18, for linux-glibc2.5 (x86_64) using  EditLine wrapper

Connection id:		44
Current database:	sbtest
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server version:		5.7.18-log MySQL Community Server (GPL)
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	utf8
Db     characterset:	utf8
Client characterset:	utf8
Conn.  characterset:	utf8
UNIX socket:		/tmp/mysql3306.sock
Uptime:			22 hours 33 min 33 sec

Threads: 15  Questions: 381223  Slow queries: 127  Opens: 283  Flush tables: 2  Open tables: 140  Queries per second avg: 4.694

[root@bd-130 ~]# ./getHostInfo.sh
Core: Linux bd-130 2.6.32-504.el6.x86_64 #1 SMP Tue Sep 16 01:56:35 EDT 2014 x86_64 x86_64 x86_64 GNU/Linux
OS: Red Hat Enterprise Linux Server release 6.6 (Santiago)
CPU model: model name : Intel Xeon E312xx (Sandy Bridge)
CPU processor: 12
MemTotal: 32877952 kB
MemFree: 404088 kB
hostname: bd-130
IP: 192.168.128.130
Load Average: 0.08 0.12 0.09 1/2493 24770
```


### MySQL 单机环境如下

```
mysql> status
--------------
mysql  Ver 14.14 Distrib 5.6.16, for Linux (x86_64) using  EditLine wrapper

Connection id:		11365
Current database:
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server version:		5.6.16 MySQL Community Server (GPL)
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	latin1
Db     characterset:	latin1
Client characterset:	utf8
Conn.  characterset:	utf8
UNIX socket:		/var/lib/mysql/mysql.sock
Uptime:			128 days 1 hour 2 min 56 sec

Threads: 49  Questions: 132832535  Slow queries: 0  Opens: 2748  Flush tables: 1  Open tables: 258  Queries per second avg: 12.006
--------------

[root@bd-133 ~]# ./getHostInfo.sh
Core: Linux bd-133 2.6.32-504.el6.x86_64 #1 SMP Tue Sep 16 01:56:35 EDT 2014 x86_64 x86_64 x86_64 GNU/Linux
OS: Red Hat Enterprise Linux Server release 6.6 (Santiago)
CPU model: model name : Intel Xeon E312xx (Sandy Bridge)
CPU processor: 12
MemTotal: 32877952 kB
MemFree: 7447468 kB
hostname: bd-133
IP: 192.168.128.133
Load Average: 0.03 0.14 0.16 1/1369 20338
```



### 附录二 Sysbench测试内容

### CPU 

```
[root@svr-146 ft]# sysbench --mysql-host=192.168.128.130 --mysql-port=3306 --mysql-user=root --mysql-password=root --threads=16 --report-interval=3 --max-requests=0 --time=30 --cpu-max-prime=10000 cpu  run
sysbench 1.0.7 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 16
Report intermediate results every 3 second(s)
Initializing random number generator from current time


Prime numbers limit: 10000

Initializing worker threads...

Threads started!

[ 3s ] thds: 16 eps: 2905.24 lat (ms,95%): 20.74
[ 6s ] thds: 16 eps: 2894.00 lat (ms,95%): 20.37
[ 9s ] thds: 16 eps: 3018.10 lat (ms,95%): 20.37
[ 12s ] thds: 16 eps: 2967.01 lat (ms,95%): 20.37
[ 15s ] thds: 16 eps: 2949.00 lat (ms,95%): 20.74
[ 18s ] thds: 16 eps: 3021.34 lat (ms,95%): 20.37
[ 21s ] thds: 16 eps: 2880.94 lat (ms,95%): 21.11
[ 24s ] thds: 16 eps: 3022.75 lat (ms,95%): 20.37
[ 27s ] thds: 16 eps: 2978.01 lat (ms,95%): 20.37

General statistics:
    total time:                          30.0032s
    total number of events:              88902

Latency (ms):
         min:                                  1.23
         avg:                                  5.39
         max:                                 59.35
         95th percentile:                     20.37
         sum:                             478922.28

Threads fairness:
    events (avg/stddev):           5556.3750/240.17
    execution time (avg/stddev):   29.9326/0.02

```

```
sysbench --mysql-host=192.168.128.133 --mysql-port=3306 --mysql-user=root --mysql-password=root --threads=16 --report-interval=3 --max-requests=0 --time=30 --cpu-max-prime=10000 cpu  run

[root@svr-146 ft]# sysbench --mysql-host=192.168.128.133 --mysql-port=3306 --mysql-user=root --mysql-password=root --threads=16 --report-interval=3 --max-requests=0 --time=30 --cpu-max-prime=10000 cpu  run
sysbench 1.0.7 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 16
Report intermediate results every 3 second(s)
Initializing random number generator from current time


Prime numbers limit: 10000

Initializing worker threads...

Threads started!

[ 3s ] thds: 16 eps: 2963.14 lat (ms,95%): 21.11
[ 6s ] thds: 16 eps: 2975.14 lat (ms,95%): 20.37
[ 9s ] thds: 16 eps: 3005.01 lat (ms,95%): 20.37
[ 12s ] thds: 16 eps: 2971.30 lat (ms,95%): 20.37
[ 15s ] thds: 16 eps: 2896.00 lat (ms,95%): 21.11
[ 18s ] thds: 16 eps: 2957.32 lat (ms,95%): 21.11
[ 21s ] thds: 16 eps: 2957.35 lat (ms,95%): 20.37
[ 24s ] thds: 16 eps: 2980.04 lat (ms,95%): 21.11
[ 27s ] thds: 16 eps: 3023.89 lat (ms,95%): 20.37

General statistics:
    total time:                          30.0035s
    total number of events:              88933

Latency (ms):
         min:                                  1.23
         avg:                                  5.39
         max:                                 51.17
         95th percentile:                     20.74
         sum:                             479092.26

Threads fairness:
    events (avg/stddev):           5558.3125/222.16
    execution time (avg/stddev):   29.9433/0.04
```


### FileIO 顺序读写

```
sysbench --mysql-host=192.168.128.130 --mysql-port=3306 --mysql-user=root --mysql-password=root --threads=16 --report-interval=3 --max-requests=0 --time=30 --file-total-size=1G fileio  prepare

sysbench  --mysql-host=192.168.128.130 --mysql-port=3306 --mysql-user=root --mysql-password=root --threads=16  --file-total-size=1G  --file-text-model=rndrw --report-interval=1  fileio run

 --file-total-size=1G prepare
```


```
[root@svr-146 ft]# sysbench  --mysql-host=192.168.128.130 --mysql-port=3306 --mysql-user=root --mysql-password=root --threads=16  --file-total-size=1G  --report-interval=1  --file-test-mode='seqwr' fileio run
sysbench 1.0.7 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 16
Report intermediate results every 1 second(s)
Initializing random number generator from current time


Extra file open flags: 0
128 files, 8MiB each
1GiB total file size
Block size 16KiB
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing sequential write (creation) test
Initializing worker threads...

Threads started!

[ 1s ] reads: 0.00 MiB/s writes: 20.22 MiB/s fsyncs: 1574.70/s latency (ms,95%): 28.162
[ 2s ] reads: 0.00 MiB/s writes: 20.32 MiB/s fsyncs: 1619.88/s latency (ms,95%): 27.165
[ 3s ] reads: 0.00 MiB/s writes: 15.63 MiB/s fsyncs: 1318.04/s latency (ms,95%): 29.725
[ 4s ] reads: 0.00 MiB/s writes: 14.06 MiB/s fsyncs: 1144.05/s latency (ms,95%): 31.375
[ 5s ] reads: 0.00 MiB/s writes: 18.75 MiB/s fsyncs: 1500.87/s latency (ms,95%): 25.278
[ 6s ] reads: 0.00 MiB/s writes: 21.87 MiB/s fsyncs: 1796.91/s latency (ms,95%): 25.278
[ 7s ] reads: 0.00 MiB/s writes: 10.94 MiB/s fsyncs: 933.91/s latency (ms,95%): 36.894
[ 8s ] reads: 0.00 MiB/s writes: 20.32 MiB/s fsyncs: 1686.30/s latency (ms,95%): 26.681
[ 9s ] reads: 0.00 MiB/s writes: 12.49 MiB/s fsyncs: 987.94/s latency (ms,95%): 34.954
[ 10s ] reads: 0.00 MiB/s writes: 12.51 MiB/s fsyncs: 1061.10/s latency (ms,95%): 36.236

File operations:
    reads/s:                      0.00
    writes/s:                     1068.21
    fsyncs/s:                     1362.32

Throughput:
    read, MiB/s:                  0.00
    written, MiB/s:               16.69

General statistics:
    total time:                          10.0146s
    total number of events:              24346

Latency (ms):
         min:                                  0.01
         avg:                                  6.58
         max:                                364.44
         95th percentile:                     28.67
         sum:                             160087.21

Threads fairness:
    events (avg/stddev):           1521.6250/106.13
    execution time (avg/stddev):   10.0055/0.00
```

```
 [root@svr-146 ft]# sysbench  --mysql-host=192.168.128.133 --mysql-port=3306 --mysql-user=root --mysql-password=root --threads=16  --file-total-size=1G  --report-interval=1  --file-test-mode='seqwr' fileio run
sysbench 1.0.7 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 16
Report intermediate results every 1 second(s)
Initializing random number generator from current time


Extra file open flags: 0
128 files, 8MiB each
1GiB total file size
Block size 16KiB
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing sequential write (creation) test
Initializing worker threads...

Threads started!

[ 1s ] reads: 0.00 MiB/s writes: 18.69 MiB/s fsyncs: 1477.51/s latency (ms,95%): 30.265
[ 2s ] reads: 0.00 MiB/s writes: 21.89 MiB/s fsyncs: 1746.93/s latency (ms,95%): 26.205
[ 3s ] reads: 0.00 MiB/s writes: 21.87 MiB/s fsyncs: 1778.91/s latency (ms,95%): 25.278
[ 4s ] reads: 0.00 MiB/s writes: 18.75 MiB/s fsyncs: 1549.08/s latency (ms,95%): 27.165
[ 5s ] reads: 0.00 MiB/s writes: 21.88 MiB/s fsyncs: 1779.10/s latency (ms,95%): 25.278
[ 6s ] reads: 0.00 MiB/s writes: 18.72 MiB/s fsyncs: 1593.36/s latency (ms,95%): 26.681
[ 7s ] reads: 0.00 MiB/s writes: 18.78 MiB/s fsyncs: 1495.38/s latency (ms,95%): 27.659
[ 8s ] reads: 0.00 MiB/s writes: 21.88 MiB/s fsyncs: 1838.04/s latency (ms,95%): 25.278
[ 9s ] reads: 0.00 MiB/s writes: 18.75 MiB/s fsyncs: 1489.14/s latency (ms,95%): 27.659
[ 10s ] reads: 0.00 MiB/s writes: 20.31 MiB/s fsyncs: 1679.79/s latency (ms,95%): 26.681

File operations:
    reads/s:                      0.00
    writes/s:                     1287.98
    fsyncs/s:                     1642.02

Throughput:
    read, MiB/s:                  0.00
    written, MiB/s:               20.12

General statistics:
    total time:                          10.0136s
    total number of events:              29346

Latency (ms):
         min:                                  0.01
         avg:                                  5.45
         max:                                198.72
         95th percentile:                     26.68
         sum:                             160017.79

Threads fairness:
    events (avg/stddev):           1834.1250/100.51
    execution time (avg/stddev):   10.0011/0.00


```

### FileIO 随机读写

```
[root@svr-146 ft]# sysbench  --mysql-host=192.168.128.130 --mysql-port=3306 --mysql-user=root --mysql-password=root --threads=16  --file-total-size=1G  --report-interval=1  --file-test-mode='rndrw' fileio run
sysbench 1.0.7 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 16
Report intermediate results every 1 second(s)
Initializing random number generator from current time


Extra file open flags: 0
128 files, 8MiB each
1GiB total file size
Block size 16KiB
Number of IO requests: 0
Read/Write ratio for combined random IO test: 1.50
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random r/w test
Initializing worker threads...

Threads started!

[ 1s ] reads: 1.51 MiB/s writes: 1.01 MiB/s fsyncs: 126.59/s latency (ms,95%): 211.599
[ 2s ] reads: 1.39 MiB/s writes: 0.77 MiB/s fsyncs: 162.04/s latency (ms,95%): 219.358
[ 3s ] reads: 1.19 MiB/s writes: 0.95 MiB/s fsyncs: 223.00/s latency (ms,95%): 164.449
[ 4s ] reads: 1.48 MiB/s writes: 0.92 MiB/s fsyncs: 132.00/s latency (ms,95%): 153.021
[ 5s ] reads: 1.06 MiB/s writes: 0.62 MiB/s fsyncs: 136.99/s latency (ms,95%): 277.214
[ 6s ] reads: 0.95 MiB/s writes: 0.78 MiB/s fsyncs: 230.01/s latency (ms,95%): 164.449
[ 7s ] reads: 1.61 MiB/s writes: 1.08 MiB/s fsyncs: 141.00/s latency (ms,95%): 164.449
[ 8s ] reads: 1.11 MiB/s writes: 0.64 MiB/s fsyncs: 133.00/s latency (ms,95%): 215.443
[ 9s ] reads: 1.03 MiB/s writes: 0.63 MiB/s fsyncs: 164.01/s latency (ms,95%): 262.636
[ 10s ] reads: 1.30 MiB/s writes: 1.02 MiB/s fsyncs: 214.97/s latency (ms,95%): 158.632

File operations:
    reads/s:                      81.82
    writes/s:                     53.52
    fsyncs/s:                     165.24

Throughput:
    read, MiB/s:                  1.28
    written, MiB/s:               0.84

General statistics:
    total time:                          10.0683s
    total number of events:              3027

Latency (ms):
         min:                                  0.01
         avg:                                 53.03
         max:                                546.65
         95th percentile:                    207.82
         sum:                             160521.44

Threads fairness:
    events (avg/stddev):           189.1875/12.71
    execution time (avg/stddev):   10.0326/0.02
```


```
[root@svr-146 ft]# sysbench  --mysql-host=192.168.128.133 --mysql-port=3306 --mysql-user=root --mysql-password=root --threads=16  --file-total-size=1G  --report-interval=1  --file-test-mode='rndrw' fileio run
sysbench 1.0.7 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 16
Report intermediate results every 1 second(s)
Initializing random number generator from current time


Extra file open flags: 0
128 files, 8MiB each
1GiB total file size
Block size 16KiB
Number of IO requests: 0
Read/Write ratio for combined random IO test: 1.50
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random r/w test
Initializing worker threads...

Threads started!

[ 1s ] reads: 1.03 MiB/s writes: 0.53 MiB/s fsyncs: 108.68/s latency (ms,95%): 287.379
[ 2s ] reads: 0.94 MiB/s writes: 0.78 MiB/s fsyncs: 138.05/s latency (ms,95%): 227.402
[ 3s ] reads: 1.70 MiB/s writes: 1.09 MiB/s fsyncs: 138.01/s latency (ms,95%): 186.540
[ 4s ] reads: 1.11 MiB/s writes: 0.62 MiB/s fsyncs: 195.99/s latency (ms,95%): 186.540
[ 5s ] reads: 0.94 MiB/s writes: 0.62 MiB/s fsyncs: 162.98/s latency (ms,95%): 196.894
[ 6s ] reads: 1.73 MiB/s writes: 1.25 MiB/s fsyncs: 155.02/s latency (ms,95%): 183.211
[ 7s ] reads: 1.11 MiB/s writes: 0.83 MiB/s fsyncs: 250.93/s latency (ms,95%): 183.211
[ 8s ] reads: 1.00 MiB/s writes: 0.63 MiB/s fsyncs: 123.02/s latency (ms,95%): 248.825
[ 9s ] reads: 1.69 MiB/s writes: 1.05 MiB/s fsyncs: 137.02/s latency (ms,95%): 144.974
[ 10s ] reads: 1.03 MiB/s writes: 0.63 MiB/s fsyncs: 230.00/s latency (ms,95%): 196.894

File operations:
    reads/s:                      78.26
    writes/s:                     51.18
    fsyncs/s:                     164.88

Throughput:
    read, MiB/s:                  1.22
    written, MiB/s:               0.80

General statistics:
    total time:                          10.0415s
    total number of events:              2956

Latency (ms):
         min:                                  0.01
         avg:                                 54.23
         max:                                649.30
         95th percentile:                    200.47
         sum:                             160289.48

Threads fairness:
    events (avg/stddev):           184.7500/14.52
    execution time (avg/stddev):   10.0181/0.01
```

### 内存顺序读写

```
[root@svr-146 ft]# sysbench  --mysql-host=192.168.128.130 --mysql-port=3306 --mysql-user=root --mysql-password=root --threads=16  --report-interval=1 --memory-total-size='64G' --memory-scope='global' --memory-access-mode='seq'  memory  run
sysbench 1.0.7 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 16
Report intermediate results every 1 second(s)
Initializing random number generator from current time


Running memory speed test with the following options:
  block size: 1KiB
  total size: 65536MiB
  operation: write
  scope: global

Initializing worker threads...

Threads started!

[ 1s ] 3960.05 MiB/sec
[ 2s ] 4068.61 MiB/sec
[ 3s ] 3910.37 MiB/sec
[ 4s ] 3868.35 MiB/sec
[ 5s ] 3927.65 MiB/sec
[ 6s ] 3979.88 MiB/sec
[ 7s ] 3827.92 MiB/sec
[ 8s ] 3827.45 MiB/sec
[ 9s ] 4086.96 MiB/sec
[ 10s ] 4119.10 MiB/sec
Total operations: 40535434 (4051507.68 per second)

39585.38 MiB transferred (3956.55 MiB/sec)


General statistics:
    total time:                          10.0005s
    total number of events:              40535434

Latency (ms):
         min:                                  0.00
         avg:                                  0.00
         max:                                 35.01
         95th percentile:                      0.00
         sum:                              62761.32

Threads fairness:
    events (avg/stddev):           2533464.6250/54388.71
    execution time (avg/stddev):   3.9226/0.18

```

```
[root@svr-146 ft]# sysbench  --mysql-host=192.168.128.133 --mysql-port=3306 --mysql-user=root --mysql-password=root --threads=16  --report-interval=1 --memory-total-size='64G' --memory-scope='global' --memory-access-mode='seq'  memory  run
sysbench 1.0.7 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 16
Report intermediate results every 1 second(s)
Initializing random number generator from current time


Running memory speed test with the following options:
  block size: 1KiB
  total size: 65536MiB
  operation: write
  scope: global

Initializing worker threads...

Threads started!

[ 1s ] 3658.36 MiB/sec
[ 2s ] 3870.01 MiB/sec
[ 3s ] 3709.79 MiB/sec
[ 4s ] 3816.73 MiB/sec
[ 5s ] 4004.27 MiB/sec
[ 6s ] 3791.36 MiB/sec
[ 7s ] 3828.60 MiB/sec
[ 8s ] 3616.10 MiB/sec
[ 9s ] 3305.23 MiB/sec
Total operations: 37935826 (3792696.86 per second)

37046.71 MiB transferred (3703.81 MiB/sec)


General statistics:
    total time:                          10.0002s
    total number of events:              37935826

Latency (ms):
         min:                                  0.00
         avg:                                  0.00
         max:                                 36.09
         95th percentile:                      0.00
         sum:                              61801.45

Threads fairness:
    events (avg/stddev):           2370989.1250/216522.74
    execution time (avg/stddev):   3.8626/0.27


```

### 内存随机读写

```
[root@svr-146 ft]# sysbench  --mysql-host=192.168.128.130 --mysql-port=3306 --mysql-user=root --mysql-password=root --threads=16  --report-interval=1 --memory-total-size='64G' --memory-scope='global' --memory-access-mode='rnd'  memory  run
sysbench 1.0.7 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 16
Report intermediate results every 1 second(s)
Initializing random number generator from current time


Running memory speed test with the following options:
  block size: 1KiB
  total size: 65536MiB
  operation: write
  scope: global

Initializing worker threads...

Threads started!

[ 1s ] 866.41 MiB/sec
[ 2s ] 930.91 MiB/sec
[ 3s ] 748.79 MiB/sec
[ 4s ] 713.38 MiB/sec
[ 5s ] 637.23 MiB/sec
[ 6s ] 710.76 MiB/sec
[ 7s ] 742.50 MiB/sec
[ 8s ] 885.38 MiB/sec
[ 9s ] 720.42 MiB/sec
Total operations: 7724222 (772244.27 per second)

7543.19 MiB transferred (754.14 MiB/sec)


General statistics:
    total time:                          10.0001s
    total number of events:              7724222

Latency (ms):
         min:                                  0.00
         avg:                                  0.02
         max:                                 42.87
         95th percentile:                      0.01
         sum:                             136724.00

Threads fairness:
    events (avg/stddev):           482763.8750/42028.05
    execution time (avg/stddev):   8.5452/0.43
```


```
[root@svr-146 ft]# sysbench  --mysql-host=192.168.128.133 --mysql-port=3306 --mysql-user=root --mysql-password=root --threads=16  --report-interval=1 --memory-total-size='64G' --memory-scope='global' --memory-access-mode='rnd'  memory  run
sysbench 1.0.7 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 16
Report intermediate results every 1 second(s)
Initializing random number generator from current time


Running memory speed test with the following options:
  block size: 1KiB
  total size: 65536MiB
  operation: write
  scope: global

Initializing worker threads...

Threads started!

[ 1s ] 677.44 MiB/sec
[ 2s ] 631.95 MiB/sec
[ 3s ] 678.27 MiB/sec
[ 4s ] 699.98 MiB/sec
[ 5s ] 741.74 MiB/sec
[ 6s ] 791.09 MiB/sec
[ 7s ] 677.31 MiB/sec
[ 8s ] 810.61 MiB/sec
[ 9s ] 896.03 MiB/sec
Total operations: 7437715 (743602.97 per second)

7263.39 MiB transferred (726.17 MiB/sec)


General statistics:
    total time:                          10.0001s
    total number of events:              7437715

Latency (ms):
         min:                                  0.00
         avg:                                  0.02
         max:                                 45.01
         95th percentile:                      0.01
         sum:                             134218.74

Threads fairness:
    events (avg/stddev):           464857.1875/19123.03
    execution time (avg/stddev):   8.3887/0.52

```


[1]:https://gist.github.com/futeng/5b5906b7b74184e71c11c8970da51278
[2]:https://github.com/akopytov/sysbench#installing-from-binary-packages
[3]:./mysql-group-replication-install.html
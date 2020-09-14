# Redis持久化

## 什么是Redis持久化？为什么要进行持久化？ 

**持久化**就是把内存的数据写到磁盘中去，防止服务宕机了内存数据丢失。 Redis 提供了两种持久化方式:RDB（默认）和AOF。

防止数据的意外丢失，确保数据安全性

## RDB的三种启动方式

### save指令

命令执行

谁：redis操作者（用户）

什么时间：即时（随时进行）

干什么事情：保存数据



首先进入客户端

```
[zouyu@bogon src]$ redis-cli -p 6380
127.0.0.1:6380> set name zs
OK
127.0.0.1:6380> save
OK
127.0.0.1:6380> 
```

进入服务端

```
total 24
-rw-rw-r--. 1 zouyu zouyu  2509 Sep  8 04:25 6379.log
-rw-rw-r--. 1 zouyu zouyu 13419 Sep  8 07:47 6380.log
-rw-rw-r--. 1 zouyu zouyu   178 Sep  8 05:48 dump.rdb
[zouyu@bogon data]$ ll

total 28
-rw-rw-r--. 1 zouyu zouyu  2509 Sep  8 04:25 6379.log
-rw-rw-r--. 1 zouyu zouyu 13465 Sep  8 07:48 6380.log
-rw-rw-r--. 1 zouyu zouyu   171 Sep  8 07:48 dump-6380.rdb
-rw-rw-r--. 1 zouyu zouyu   178 Sep  8 05:48 dump.rdb
```

查看dump-6380.rdb

```
[zouyu@bogon data]$ cat dump-6380.rdb
REDIS0008�	redis-ver4.0.0�
redis-bits�@�ctime²�W_used-mem¨�
                                aof-preamble��repl-id(2a11245001ea392fca0ec759ccf75c4e55b2657c�
                                                                                               repl-offset���namezs�b��+���[zouyu@bogon data]$ 
```

get>save>set>set   **单线程任务执行序列**

**注意：**save指令的执行会阻塞当前Redis服务器，直到当前RDB过程完成为止，有可能会造成长时间阻塞，线上环境不建议使用。

### bgsave

谁：redis操作者（用户）发起指令；redis服务器控制指令执行

什么时间：即时（发起）；合理的时间（执行）

干什么事情：保存数据



进入客户端

```
127.0.0.1:6380> set name ls
OK
127.0.0.1:6380> bgsave
Background saving started  //此时指令并不是立即执行，而是后台在合理的时间执行
127.0.0.1:6380> 
```

进入服务端

```
[zouyu@bogon data]$ cat dump-6380.rdb
REDIS0008�	redis-ver4.0.0�
redis-bits�@�ctime±�W_used-mem°�
                                aof-preamble��repl-id(2a11245001ea392fca0ec759ccf75c4e55b2657c�
                                                                                               repl-offset���namels����e�}[zouyu@bogon data]$ 
```

查看日志文件

```
[zouyu@bogon data]$ cat 6380.log
2801:C 08 Sep 04:11:04.867 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
2801:C 08 Sep 04:11:04.867 # Redis version=4.0.0, bits=64, commit=00000000, modified=0, pid=2801, just started
2801:C 08 Sep 04:11:04.867 # Configuration loaded
2802:M 08 Sep 04:11:04.870 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
2802:M 08 Sep 04:11:04.870 # Server can't set maximum open files to 10032 because of OS error: Operation not permitted.
2802:M 08 Sep 04:11:04.870 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
2802:M 08 Sep 04:11:04.872 * Running mode=standalone, port=6380.
2802:M 08 Sep 04:11:04.873 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
2802:M 08 Sep 04:11:04.873 # Server initialized
2802:M 08 Sep 04:11:04.873 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
2802:M 08 Sep 04:11:04.873 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
2802:M 08 Sep 04:11:04.873 * Ready to accept connections
4139:C 08 Sep 04:48:02.475 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
4139:C 08 Sep 04:48:02.475 # Redis version=4.0.0, bits=64, commit=00000000, modified=0, pid=4139, just started
4139:C 08 Sep 04:48:02.475 # Configuration loaded
4140:M 08 Sep 04:48:02.478 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
4140:M 08 Sep 04:48:02.478 # Server can't set maximum open files to 10032 because of OS error: Operation not permitted.
4140:M 08 Sep 04:48:02.478 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
4140:M 08 Sep 04:48:02.480 * Running mode=standalone, port=6380.
4140:M 08 Sep 04:48:02.480 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
4140:M 08 Sep 04:48:02.480 # Server initialized
4140:M 08 Sep 04:48:02.481 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
4140:M 08 Sep 04:48:02.481 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
4140:M 08 Sep 04:48:02.482 * DB loaded from disk: 0.001 seconds
4140:M 08 Sep 04:48:02.482 * Ready to accept connections
2680:C 08 Sep 04:56:29.365 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
2680:C 08 Sep 04:56:29.366 # Redis version=4.0.0, bits=64, commit=00000000, modified=0, pid=2680, just started
2680:C 08 Sep 04:56:29.366 # Configuration loaded
2681:M 08 Sep 04:56:29.368 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
2681:M 08 Sep 04:56:29.368 # Server can't set maximum open files to 10032 because of OS error: Operation not permitted.
2681:M 08 Sep 04:56:29.368 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
2681:M 08 Sep 04:56:29.370 * Running mode=standalone, port=6380.
2681:M 08 Sep 04:56:29.370 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
2681:M 08 Sep 04:56:29.370 # Server initialized
2681:M 08 Sep 04:56:29.370 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
2681:M 08 Sep 04:56:29.370 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
2681:M 08 Sep 04:56:29.371 * DB loaded from disk: 0.001 seconds
2681:M 08 Sep 04:56:29.371 * Ready to accept connections
2681:M 08 Sep 05:42:29.207 * DB saved on disk
2681:M 08 Sep 05:43:46.612 * DB saved on disk
4234:C 08 Sep 05:46:40.356 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
4234:C 08 Sep 05:46:40.356 # Redis version=4.0.0, bits=64, commit=00000000, modified=0, pid=4234, just started
4234:C 08 Sep 05:46:40.356 # Configuration loaded
4235:M 08 Sep 05:46:40.358 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
4235:M 08 Sep 05:46:40.358 # Server can't set maximum open files to 10032 because of OS error: Operation not permitted.
4235:M 08 Sep 05:46:40.358 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
4235:M 08 Sep 05:46:40.361 * Running mode=standalone, port=6380.
4235:M 08 Sep 05:46:40.361 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
4235:M 08 Sep 05:46:40.361 # Server initialized
4235:M 08 Sep 05:46:40.361 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
4235:M 08 Sep 05:46:40.362 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
4235:M 08 Sep 05:46:40.362 * DB loaded from disk: 0.000 seconds
4235:M 08 Sep 05:46:40.362 * Ready to accept connections
4235:M 08 Sep 05:48:06.015 * DB saved on disk
4631:C 08 Sep 05:55:52.863 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
4631:C 08 Sep 05:55:52.863 # Redis version=4.0.0, bits=64, commit=00000000, modified=0, pid=4631, just started
4631:C 08 Sep 05:55:52.863 # Configuration loaded
4632:M 08 Sep 05:55:52.867 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
4632:M 08 Sep 05:55:52.868 # Server can't set maximum open files to 10032 because of OS error: Operation not permitted.
4632:M 08 Sep 05:55:52.868 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
4632:M 08 Sep 05:55:52.870 * Running mode=standalone, port=6380.
4632:M 08 Sep 05:55:52.870 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
4632:M 08 Sep 05:55:52.870 # Server initialized
4632:M 08 Sep 05:55:52.870 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
4632:M 08 Sep 05:55:52.871 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
4632:M 08 Sep 05:55:52.871 * Ready to accept connections
4632:M 08 Sep 05:56:16.894 * DB saved on disk
4632:M 08 Sep 06:52:18.300 * Background saving started by pid 5172
5172:C 08 Sep 06:52:18.309 * DB saved on disk
5172:C 08 Sep 06:52:18.309 * RDB: 0 MB of memory used by copy-on-write
4632:M 08 Sep 06:52:18.327 * Background saving terminated with success
5544:C 08 Sep 07:04:45.603 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
5544:C 08 Sep 07:04:45.603 # Redis version=4.0.0, bits=64, commit=00000000, modified=0, pid=5544, just started
5544:C 08 Sep 07:04:45.603 # Configuration loaded
5545:M 08 Sep 07:04:45.607 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
5545:M 08 Sep 07:04:45.607 # Server can't set maximum open files to 10032 because of OS error: Operation not permitted.
5545:M 08 Sep 07:04:45.607 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
5545:M 08 Sep 07:04:45.609 * Running mode=standalone, port=6380.
5545:M 08 Sep 07:04:45.609 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
5545:M 08 Sep 07:04:45.609 # Server initialized
5545:M 08 Sep 07:04:45.609 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
5545:M 08 Sep 07:04:45.610 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
5545:M 08 Sep 07:04:45.610 * DB loaded from disk: 0.000 seconds
5545:M 08 Sep 07:04:45.610 * Ready to accept connections
5545:M 08 Sep 07:06:43.018 * 2 changes in 10 seconds. Saving...
5545:M 08 Sep 07:06:43.019 * Background saving started by pid 5693
5693:C 08 Sep 07:06:43.024 * DB saved on disk
5693:C 08 Sep 07:06:43.024 * RDB: 0 MB of memory used by copy-on-write
5545:M 08 Sep 07:06:43.120 * Background saving terminated with success
5545:M 08 Sep 07:07:29.754 * 2 changes in 10 seconds. Saving...
5545:M 08 Sep 07:07:29.754 * Background saving started by pid 5716
5716:C 08 Sep 07:07:29.757 * DB saved on disk
5716:C 08 Sep 07:07:29.757 * RDB: 0 MB of memory used by copy-on-write
5545:M 08 Sep 07:07:29.854 * Background saving terminated with success
5545:M 08 Sep 07:08:09.808 * 2 changes in 10 seconds. Saving...
5545:M 08 Sep 07:08:09.808 * Background saving started by pid 5748
5748:C 08 Sep 07:08:09.830 * DB saved on disk
5748:C 08 Sep 07:08:09.830 * RDB: 0 MB of memory used by copy-on-write
5545:M 08 Sep 07:08:09.909 * Background saving terminated with success
5545:M 08 Sep 07:08:33.521 * 2 changes in 10 seconds. Saving...
5545:M 08 Sep 07:08:33.522 * Background saving started by pid 5769
5769:C 08 Sep 07:08:33.532 * DB saved on disk
5769:C 08 Sep 07:08:33.532 * RDB: 0 MB of memory used by copy-on-write
5545:M 08 Sep 07:08:33.622 * Background saving terminated with success
5545:M 08 Sep 07:11:35.231 * 2 changes in 10 seconds. Saving...
5545:M 08 Sep 07:11:35.232 * Background saving started by pid 5884
5884:C 08 Sep 07:11:35.254 * DB saved on disk
5884:C 08 Sep 07:11:35.255 * RDB: 0 MB of memory used by copy-on-write
5545:M 08 Sep 07:11:35.333 * Background saving terminated with success
5545:M 08 Sep 07:12:26.304 * 2 changes in 10 seconds. Saving...
5545:M 08 Sep 07:12:26.304 * Background saving started by pid 5920
5920:C 08 Sep 07:12:26.320 * DB saved on disk
5920:C 08 Sep 07:12:26.321 * RDB: 0 MB of memory used by copy-on-write
5545:M 08 Sep 07:12:26.406 * Background saving terminated with success
5545:M 08 Sep 07:38:36.063 * DB saved on disk
5545:M 08 Sep 07:40:17.964 * DB saved on disk
5545:M 08 Sep 07:43:25.715 * DB saved on disk
5545:M 08 Sep 07:45:13.022 * DB saved on disk
5545:M 08 Sep 07:45:28.460 * DB saved on disk
5545:M 08 Sep 07:45:57.648 * 2 changes in 10 seconds. Saving...
5545:M 08 Sep 07:45:57.649 * Background saving started by pid 6606
6606:C 08 Sep 07:45:57.658 * DB saved on disk
6606:C 08 Sep 07:45:57.659 * RDB: 0 MB of memory used by copy-on-write
5545:M 08 Sep 07:45:57.750 * Background saving terminated with success
5545:M 08 Sep 07:47:32.719 * 2 changes in 10 seconds. Saving...
5545:M 08 Sep 07:47:32.720 * Background saving started by pid 6649
6649:C 08 Sep 07:47:32.721 * DB saved on disk
6649:C 08 Sep 07:47:32.722 * RDB: 0 MB of memory used by copy-on-write
5545:M 08 Sep 07:47:32.821 * Background saving terminated with success
5545:M 08 Sep 07:48:18.406 * DB saved on disk
5545:M 08 Sep 07:56:49.688 * Background saving started by pid 6857
6857:C 08 Sep 07:56:49.747 * DB saved on disk
6857:C 08 Sep 07:56:49.747 * RDB: 0 MB of memory used by copy-on-write

5545:M 08 Sep 07:56:49.816 * Background saving terminated with success
```

**注意：** bgsave命令是针对save阻塞问题做的优化。Redis内部所有涉及到RDB操作都采用bgsave的方式，save命令可以放弃使用**。**

### save和bgsave的区别

| 方式           | save | bgsave |
| -------------- | ---- | ------ |
| 读写           | 同步 | 异步   |
| 阻塞客户端指令 | 是   | 否     |
| 额外内存消耗   | 否   | 是     |
| 启动新进程     | 否   | 是     |

save保存时阻塞主线程，客户端无法连接redis，等save完成后，主进程才开始工作，客户端可以连接

bgsave是一个叫fork的子进程，在执行save过程中，不影响主进程，客户端可以正常连接redis，等子进程fork执行完save后，通知主进程，子进程关闭。

### save配置

配置

```
save second changes
```

作用

满足限定时间范围内key的变化数量达到指定数量即进行持久化

参数

second：监控时间范围

changes：监控key的变化量

位置

在conf文件中进行配置

![image-20200908230625450](C:\Users\15079\AppData\Roaming\Typora\typora-user-images\image-20200908230625450.png)

进入客户端

```
127.0.0.1:6380> set name adc
OK
127.0.0.1:6380> set age 22
OK
```

进入服务端

```
total 28
-rw-rw-r--. 1 zouyu zouyu  2509 Sep  8 04:25 6379.log
-rw-rw-r--. 1 zouyu zouyu 13720 Sep  8 07:56 6380.log
-rw-rw-r--. 1 zouyu zouyu   171 Sep  8 07:56 dump-6380.rdb
-rw-rw-r--. 1 zouyu zouyu   178 Sep  8 05:48 dump.rdb
[zouyu@bogon data]$ ll
total 28
-rw-rw-r--. 1 zouyu zouyu  2509 Sep  8 04:25 6379.log
-rw-rw-r--. 1 zouyu zouyu 14039 Sep  8 08:07 6380.log
-rw-rw-r--. 1 zouyu zouyu   179 Sep  8 08:07 dump-6380.rdb
-rw-rw-r--. 1 zouyu zouyu   178 Sep  8 05:48 dump.rdb
[zouyu@bogon data]$ 
```

**注意：** save配置要根据实际业务情况进行设置，频度过高或过低都会出现性能问题，结果可能是灾难性的

 save配置中对于second与changes设置通常具有互补对应关系，尽量不要设置成包含性关系

 save配置启动后执行的是bgsave操作

### rdb特殊启动方式

全量复制

服务器运行过程中重启

关闭服务器时指定保存数据

### RDB优点

RDB是一个紧凑压缩的二进制文件，存储效率较高

 RDB内部存储的是redis在某个时间点的数据快照，非常适合用于数据备份，全量复制等场景

 RDB恢复数据的速度要比AOF快很多

 应用：服务器中每X小时执行bgsave备份，并将RDB文件拷贝到远程机器中，用于灾难恢复

### RDB缺点

RDB方式无论是执行指令还是利用配置，无法做到实时持久化，具有较大的可能性丢失数据

bgsave指令每次运行要执行fork操作创建子进程，要牺牲掉一些性能

Redis的众多版本中未进行RDB文件格式的版本统一，有可能出现各版本服务之间数据格式无法兼容现象



### 解决思路

不写全数据，仅记录部分数据

 降低区分数据是否改变的难度，改记录数据为记录操作过程

对所有操作均进行记录，排除丢失数据的风险



## AOF

AOF(append only file)持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中命令

达到恢复数据的目的。与RDB相比可以简单描述为改记录数据为记录数据产生的过程

AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式

### AOF写数据三种策略

#### always(每次）

每次写入操作均同步到AOF文件中，数据零误差，性能较低

#### everysec（每秒）

每秒将缓冲区中的指令同步到AOF文件中，数据准确性较高，性能较高

在系统突然宕机的情况下丢失1秒内的数据

####  no（系统控制）

由操作系统控制每次同步到AOF文件的周期，整体过程不可控

### AOF功能开启

配置

```
appendonly yes|no
```

作用

是否开启AOF持久化功能，默认为不开启状态

 配置

```
appendfsync always|everysec| no
```

 

配置

```
appendfilename filename
```

作用

AOF持久化文件名，默认文件名未appendonly.aof，建议配置为appendonly-端口号.aof

### **AOF重写**

随着命令不断写入AOF，文件会越来越大，为了解决这个问题，Redis引入了AOF重写机制压缩文件体积。AOF文件重

写是将Redis进程内的数据转化为写命令同步到新AOF文件的过程。简单说就是将对同一个数据的若干个条命令执行结

果转化成最终结果数据对应的指令进行记录。

示例

```
127.0.0.1:6380> set name zs
OK
127.0.0.1:6380> set name ls
OK
127.0.0.1:6380> set name zc
OK
127.0.0.1:6380> bgrewriteaof    手动开启自动重写
Background append only file rewriting started
```

查看aof文件

```
[zouyu@localhost data]$ cat appendonly-6380.aof
*2
$6
SELECT
$1
0
*3
$3
SET
$4
name
$2
zc
```

### **AOF重写作用**

降低磁盘占用量，提高磁盘利用率

 提高持久化效率，降低持久化写时间，提高IO性能

降低数据恢复用时，提高数据恢复效率

### **AOF重写规则**

进程内已超时的数据不再写入文件

忽略无效指令，重写时使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令

如set key 111、set key 222等 

对同一数据的多条写命令合并为一条命令

如lpush list1 a、lpush list1 b、 lpush list1 c 可以转化为：lpush list1 a b c。

为防止数据量过大造成客户端缓冲区溢出，对list、set、hash、zset等类型，每条指令最多写入64个元素

### AOF的重写方式

#### 手动重写

```
bgrewriteaof
```

#### 自动重写

```
auto-aof-rewrite-min-size size
auto-aof-rewrite-percentage percentage
```

自动重写触发条件

```
aof-current-size>auto-aof-rewrite-min-size

aof-current-size-aof_base_size  /  aof_base_size >= auto-aof-rewrite-percentage percentage
```

### AOF和RDB的区别

![image-20200909142057757](C:\Users\15079\AppData\Roaming\Typora\typora-user-images\image-20200909142057757.png)



### 如何选择RDB和AOF

**对数据非常敏感，建议使用默认的AOF持久化方案**

 AOF持久化策略使用everysecond，每秒钟fsync一次。该策略redis仍可以保持很好的处理性能，当出

现问题时，最多丢失0-1秒内的数据。

 注意：由于AOF文件存储体积较大，且恢复速度较慢

 **数据呈现阶段有效性，建议使用RDB持久化方案**

 数据可以良好的做到阶段内无丢失（该阶段是开发者或运维人员手工维护的），且恢复速度较快，阶段

点数据恢复通常采用RDB方案

 注意：利用RDB实现紧凑的数据持久化会使Redis降的很低，慎重总结：

**综合比对**

 RDB与AOF的选择实际上是在做一种权衡，每种都有利有弊

 如不能承受数分钟以内的数据丢失，对业务数据非常敏感，选用AOF

如能承受数分钟以内的数据丢失，且追求大数据集的恢复速度，选用RDB

 灾难恢复选用RDB

 双保险策略，同时开启 RDB 和 AOF，重启后，Redis优先使用 AOF 来恢复数据，降低丢失数据的量
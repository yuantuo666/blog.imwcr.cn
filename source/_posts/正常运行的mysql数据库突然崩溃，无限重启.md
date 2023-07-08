---
title: 正常运行的MySQL数据库突然崩溃，无限重启
tags:
  - bug
  - Linux
  - MySQL
  - 技术
  - 服务器
  - 网路
id: '313'
categories:
  - - Web
  - - 后端
date: 2023-05-24 00:44:43
cover: https://blog.imwcr.cn/wp-content/uploads/2023/05/benjamin-lehman-GNyjCePVRs8-unsplash-scaled.jpg
coverWidth: 1200
coverHeight: 600
---

稳定运行了一个月的MySQL数据库突然寄了，连接的PHP返回错误是MySQL server has gone away.

SSH 连上服务器发现 mysqld 的 CPU 占用极高，查看 NGINX 日志和 netstat 连接情况确认没有人在攻击。

关闭 PHP-fpm 服务，重启 MySQL 和 服务器均不管用。

![](https://blog.imwcr.cn/wp-content/uploads/2023/05/8d1e05b8964b910ec378f9c9743e9f6.png)

检查 systemctl status mysql.service 和 journalctl -xe

```
root@linux:~# systemctl status mysql.service
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
     Active: activating (start) since Tue 2023-05-23 15:38:44 UTC; 435ms ago
    Process: 2894388 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
   Main PID: 2894402 (mysqld)
     Status: "Server startup in progress"
      Tasks: 1 (limit: 309253)
     Memory: 218.0M
     CGroup: /system.slice/mysql.service
             └─2894402 /usr/sbin/mysqld

May 23 15:38:44 linux.4849e9.com systemd[1]: Starting MySQL Community Server...
```

```
May 23 15:41:10 linux.4849e9.com systemd[1]: Starting MySQL Community Server...
-- Subject: A start job for unit mysql.service has begun execution
-- Defined-By: systemd
-- Support: http://www.ubuntu.com/support
-- 
-- A start job for unit mysql.service has begun execution.
-- 
-- The job identifier is 2578264.
```

均表明 MySQL 启动出现了问题。

但在数据库 crash 时，并没有对任何文件进行修改，令人感到奇怪。

随后检查 /var/log/mysql/error.log 时发现了问题所在。

```
2023-05-23T09:35:03.203642Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.32-0ubuntu0.20.04.2) starting as process 2669195
2023-05-23T09:35:03.235163Z 0 [Warning] [MY-013907] [InnoDB] Deprecated configuration parameters innodb_log_file_size and/or innodb_log_files_in_group have been used to compute innodb_redo_log_capacity=134217728. Please use innodb_redo_log_capacity instead.
2023-05-23T09:35:03.237246Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2023-05-23T09:35:04.426523Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2023-05-23T09:35:05.134049Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2023-05-23T09:35:05.134146Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2023-05-23T09:35:05.144203Z 0 [ERROR] [MY-000067] [Server] unknown variable 'query_cache_size=64M'.
2023-05-23T09:35:05.144432Z 0 [ERROR] [MY-010119] [Server] Aborting
2023-05-23T09:35:06.559744Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.32-0ubuntu0.20.04.2)  (Ubuntu).
2023-05-23T09:35:07.078938Z 0 [Warning] [MY-010140] [Server] Could not increase number of max_open_files to more than 10000 (request: 65535)
2023-05-23T09:35:07.390854Z 0 [Warning] [MY-000081] [Server] option 'max_allowed_packet': unsigned value 107374182400 adjusted to 1073741824.
2023-05-23T09:35:07.390914Z 0 [Warning] [MY-011068] [Server] The syntax 'expire-logs-days' is deprecated and will be removed in a future release. Please use binlog_expire_logs_seconds instead.
2023-05-23T09:35:07.390963Z 0 [Warning] [MY-010915] [Server] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
```

直接看中间 \[ERROR\] 级别的报错，就是 'query\_cache\_size=64M' 这个变量导致的问题，这很明显就是配置文件出了问题。

很久之前确实修改过默认的配置文件，但时间非常久远了，也不至于是这个时候出现问题，而且在修改之后也重启过服务器，理论上 MySQL 的配置文件应该是正常的重载了，那为什么当时没有出现这个问题，而在正常运行时出现了呢？

而且这个配置导致 MySQL 不断重启，一直没有成功启动。

```
root@linux:/etc/mysql# ll
total 32
drwxr-xr-x   4 root root 4096 Apr 28 18:29 ./
drwxr-xr-x 107 root root 4096 May 23 15:44 ../
drwxr-xr-x   2 root root 4096 Nov 21  2022 conf.d/
-rwxr-xr-x   1 root root  120 Jan 28 14:44 debian-start*
-rw-------   1 root root  317 Apr  8 05:28 debian.cnf
lrwxrwxrwx   1 root root   24 Nov 21  2022 my.cnf -> /etc/alternatives/my.cnf
-rw-r--r--   1 root root  839 Aug  3  2016 my.cnf.fallback
-rw-r--r--   1 root root 1797 Apr 28 18:49 mysql.cnf
drwxr-xr-x   2 root root 4096 Apr  8 05:28 mysql.conf.d/
```

可以看到 my.cnf 的修改日期是 Nov 21 2022 都是去年了。

vim 打开 my.cnf 并注释 query\_cache\_size=64M

重启 MySQL 服务就正常运行了。
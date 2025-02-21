# debian12安装clickhouse
> 建议使用Debian或Ubuntu的官方预编译deb软件包。运行以下命令来安装包:

```
sudo apt-get install -y apt-transport-https ca-certificates dirmngr
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754

echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list
sudo apt-get update

sudo apt-get install -y clickhouse-server clickhouse-client

sudo service clickhouse-server start
clickhouse-client # or "clickhouse-client --password" if you've set up a password.
```

## 测试是否安装成功
```
root@localhost:~# clickhouse-client
ClickHouse client version 25.1.5.31 (official build).
Connecting to localhost:9000 as user default.
Password for user (default): 
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 25.1.5.

Warnings:
 * Maximum number of threads is lower than 30000. There could be problems with handling a lot of simultaneous queries.
 * Linux transparent hugepages are set to "always". Check /sys/kernel/mm/transparent_hugepage/enabled
 * Linux threads max count is too low. Check /proc/sys/kernel/threads-max
 * Delay accounting is not enabled, OSIOWaitMicroseconds will not be gathered. You can enable it using `echo 1 > /proc/sys/kernel/task_delayacct` or by using sysctl.
 * Available memory at server startup is too low (2GiB).

localhost :) show databases;

SHOW DATABASES

Query id: f12547db-f0f1-456a-a441-3d5b32cbc4df

   ┌─name───────────────┐
1. │ INFORMATION_SCHEMA │
2. │ default            │
3. │ information_schema │
4. │ system             │
   └────────────────────┘

4 rows in set. Elapsed: 0.001 sec. 

localhost :) use default;

USE default

Query id: 76a3cb15-c22b-4756-b66f-128d2a915baa

Ok.

0 rows in set. Elapsed: 0.001 sec. 

localhost :) show tables;

SHOW TABLES

Query id: 2e7e3080-0a22-435c-94c2-b59e952183c6

Ok.

0 rows in set. Elapsed: 0.001 sec. 

localhost :) select version();

SELECT version()

Query id: fe3950d6-9206-4946-b67d-fa46a00ae7ad

   ┌─version()─┐
1. │ 25.1.5.31 │
   └───────────┘

1 row in set. Elapsed: 0.001 sec. 

localhost :) 
```
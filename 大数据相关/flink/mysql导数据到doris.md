
推荐做法：使用 JDBC Catalog
步骤一：创建 JDBC Catalog（首次配置即可）
```
CREATE CATALOG mysql_catalog_project PROPERTIES (
    "type" = "jdbc",
    "user" = "xxx",
    "password" = "xxx!",
    "jdbc_url" = "jdbc:mysql://xxx:3306/xxx?useSSL=false",
    "driver_url" = "https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/8.3.0/mysql-connector-j-8.3.0.jar",
    "driver_class" = "com.mysql.cj.jdbc.Driver"
);
```
步骤二：在 Catalog 下直接访问 MySQL 表
```
select * from mysql_catalog_project.projectc.hlhb_realtimedata limit 100
```

## 建表
```



# Doris 的 UNIQUE KEY/PRIMARY KEY 字段，必须在表结构里从第一列起排列，不能调整顺序
# 列顺序调整和 UNIQUE KEY 一致即可。
CREATE TABLE `hlhb_realtimedata` (
  `data_time` datetime NOT NULL COMMENT '数据时间',
  `station_id` int NOT NULL COMMENT '站点ID',
  `monitor_id` int NOT NULL COMMENT '监控点ID',
  `rtd` double NOT NULL COMMENT '实时数据值',
  `flag` varchar(32) NOT NULL COMMENT '数据标识',
  `flag_bit` int NOT NULL DEFAULT '0' COMMENT '标识位',
  `origin_data` varchar(2048) NOT NULL DEFAULT '' COMMENT '原始数据',
  `create_time` datetime COMMENT '创建时间',
  `update_time` datetime COMMENT '更新时间'
) 
ENGINE=OLAP
UNIQUE KEY(`data_time`, `station_id`, `monitor_id`)
DISTRIBUTED BY HASH(`station_id`, `monitor_id`) BUCKETS 8
PROPERTIES (
  "replication_num" = "1",
  "storage_format" = "V2"
);
```
# 执行导入
```
insert into hlhb.hlhb_realtimedata
SELECT
  data_time,
  station_id,
  monitor_id,
  rtd,
  flag,
  flag_bit,
  origin_data,
  create_time,
  update_time
FROM
  mysql_catalog_project.projectc.hlhb_realtimedata

```
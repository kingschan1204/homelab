# doris3 集群部署
> 3节点，资源有限每个节点安装BE 第一台安装的节点作为master节点，master节点安装fe


## 准备工作，系统设置
### /etc/sysctl.conf
```
# 编辑 sysctl 配置文件
vi /etc/sysctl.conf

# 在文件末尾添加以下内容
vm.max_map_count = 2000000

# 保存后加载配置（无需重启系统）
sysctl -p

 验证参数是否生效
# 查看当前值（应显示 2000000）
cat /proc/sys/vm/max_map_count


```
### /etc/security/limits.conf
```
vi /etc/security/limits.conf
# * 对root用户不生效
* soft nofile 655350
* hard nofile 655350

# 如果是root用户则变成
root soft nofile 655350
root hard nofile 655350

# 检查是否生效 断开会话，重新登录再执行
ulimit -n
```


### 数据目录放数据盘
```
# /data挂载另一块磁盘
mkdir -p /data/doris/fe/doris-meta
mkdir -p /data/doris/fe/log

```
---

## fe.conf
```
# 元数据目录
meta_dir = /data/doris/fe/doris-meta
# 日志
LOG_DIR = /data/doris/fe/log
# 绑定集群ip网段
priority_networks = 192.168.50.0/24
```
> priority_networks网段查看
```
获取 Doris 节点所处的内网网段，此示例的内网网段的 CIDR 是 10.1.11.0/24。
如不清楚，也可执行命令查询：
[root@Doris-1 ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
        valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:bb:75:3e brd ff:ff:ff:ff:ff:ff
    inet 10.1.11.39/24 brd 10.1.11.255 scope global noprefixroute ens192
        valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:febb:753e/64 scope link noprefixroute
        valid_lft forever preferred_lft forever

返回结果中的 10.1.11.39/24，也可作为当前节点的 CIDR 使用。

将网段信息配置到 /opt/doris/fe/conf/fe.conf 配置文件中：

echo "priority_networks = 10.1.11.39/24" >> /opt/doris/fe/conf/fe.conf

# 绑定集群ip网段
priority_networks = 192.168.50.0/24
```



## be.conf
```
# /data挂载另一块磁盘
mkdir -p /data/doris/be/doris-data
mkdir -p /data/doris/be/log
```
```
# 绑定集群ip网段
priority_networks = 192.168.50.0/24
# 替换为新路径，多个路径用逗号分隔
storage_root_path = /data/doris/be/doris-data
# java home设置加在最后面
JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```

## 启动查看效果
```
# 启动 FE
./usr/doris/fe/bin/start_fe.sh --daemon
# 停止FE
./usr/doris/fe/bin/stop_fe.sh

# 查看 FE 日志确认状态
tail -f /data/doris/fe/log/fe.log
# 查看 FE 运行状态
curl http://127.0.0.1:8030/api/bootstrap


# 启动 BE
./usr/doris/be/bin/start_be.sh --daemon

# 停止 BE 服务
./usr/doris/be/bin/stop_be.sh

# 查看 BE 日志确认状态
tail -f /data/doris/be/log/be.out


```

### 将 BE 注册到 FE
> 通过 MySQL 客户端连接 FE，执行以下命令：
```bash

# debian 12安装mysql客户端
# Debian 12 默认仓库中包含 mariadb-client（MariaDB 是 MySQL 的兼容分支，客户端命令完全兼容 MySQL）。
apt install mariadb-client -y

# 使用 MySQL 客户端连接 FE
mysql -h 127.0.0.1 -P 9030 -u root

# 在 MySQL 客户端中执行以下命令注册 BE (输出为空，没关系，继续往下执行)
SHOW PROC '/backends';  # 查看 BE 节点状态

# 注册 BE 节点 (使用实际 IP 和端口)
ALTER SYSTEM ADD BACKEND "127.0.0.1:9050";


# 再次检查 BE 状态，确保 Alive 为 true
SHOW PROC '/backends';
```


## node 1 node 2 
> 复制虚拟机为node1、node2然后启动be服务并注册到主节点
```
# 在mater节点登录
mysql -h 127.0.0.1 -P 9030 -u root

# 注册node1 到master节点
ALTER SYSTEM ADD BACKEND "192.168.50.154:9050";

# 注册node2 到master节点
ALTER SYSTEM ADD BACKEND "192.168.50.21:9050";

# 再次检查 BE 状态，确保 Alive 为 true
SHOW PROC '/backends';
```
> 部署 FE 集群
>> FE Follower（包括 Master）节点的数量建议为奇数，建议部署 3 个组成高可用模式。
>> 当 FE 处于高可用部署时（1 个 Master，2 个 Follower），我们建议通过增加 Observer FE 来扩展 FE 的读服务能力
> 由于是从主节点整个虚拟机复制过来的所以需要删除fe metadata目录再启动fe服务，不然alive服务一直是false
```
# 通过以下命令，可以启动 FE Follower 节点，并自动同步元数据。
bin/start_fe.sh --helper <helper_fe_ip>:<fe_edit_log_port> --daemon
# 其中，helper_fe_ip 是 FE 集群中任何存活节点的 IP 地址。--helper 参数仅在第一次启动 FE 时需要，之后重启无需指定。

# 添加node1
# 在主节点mysql客户端执行
# 要先在主节点添加后，再去从节点执行启动fe服务的命令
# 注册node1 mysql中执行
ALTER SYSTEM ADD FOLLOWER "192.168.50.154:9010";
# 启动Node1 fe服务
./fe/bin/start_fe.sh --helper 192.168.50.229:9010 --daemon


#注册node2
ALTER SYSTEM ADD FOLLOWER "192.168.50.21:9010";
# 启动Node2 fe服务
# 由于是从主节点整个虚拟机复制过来的所以需要删除fe metadata目录再启动
./fe/bin/stop_fe.sh 
./fe/bin/start_fe.sh --helper 192.168.50.229:9010 --daemon

ALTER SYSTEM DROP FOLLOWER "192.168.50.154:9010";
SHOW PROC '/frontends'\G
# 在node1执行

```
---
```
systemctl stop doris-fe
systemctl stop doris-be

# leader节点id edit_log_port端口
/usr/doris/fe/bin/start_fe.sh --helper 192.168.50.25:9010 --daemon

# 添加follower节点
ALTER SYSTEM ADD FOLLOWER "192.168.50.116:9010";
ALTER SYSTEM DROP FOLLOWER "192.168.50.116:9010";


# 查看状态（需要多等待一会儿，直到Alive列值都为true）
SHOW PROC '/frontends';
show backends;

#删除节点
ALTER SYSTEM DROP FOLLOWER[OBSERVER] "fe_host:edit_log_port";
```

## master节点
```
mysql -uroot -P 9030

ALTER SYSTEM ADD BACKEND '192.168.50.25:9050';

ALTER SYSTEM ADD BACKEND "192.168.50.116:9050";
ALTER SYSTEM DECOMMISSION BACKEND "192.168.50.116:9050";

ALTER SYSTEM DROP BACKEND "192.168.50.116:9050";

```



## 参考
https://doris.apache.org/zh-CN/docs/3.0/install/deploy-manually/integrated-storage-compute-deploy-manually
# doris3 集群部署
> 3节点，资源有限每个节点安装FE，BE 第一台安装的节点作为master节点
```
# 查看 FE 运行状态
curl http://127.0.0.1:8030/api/bootstrap
```
## 准备工作，系统设置
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


vi /etc/security/limits.conf
# * 对root用户不生效
* soft nofile 655350
* hard nofile 655350

# 如果是root用户则变成
root soft nofile 655350
root hard nofile 655350

# 检查是否生效 断开会话，重新登录再执行
ulimit -n




# 停止 BE 服务
./be/bin/stop_be.sh

# 启动 BE 服务
./be/bin/start_be.sh --daemon

# 查看日志确认是否成功
tail -f be/log/be.log
```

## 数据目录放数据盘
```
# /data挂载另一块磁盘
mkdir -p /data/doris/fe/doris-meta
mkdir -p /data/doris/fe/log

```
## fe.conf
```
# 元数据目录
meta_dir = /data/doris/fe/doris-meta
# 日志
LOG_DIR = /data/doris/fe/log
# 绑定集群ip网段
priority_networks = 192.168.50.0/24
```
### priority_networks网段查看
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
# 启动 FE（首次启动需指定 --daemon 后台运行）
./fe/bin/start_fe.sh --daemon

# 查看 FE 日志确认状态
tail -f /data/doris/fe/log/fe.log

# 启动 BE
./be/bin/start_be.sh --daemon

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
https://juejin.cn/post/7302023698722471977
https://doris.apache.org/zh-CN/docs/2.0/install/cluster-deployment/standard-deployment#3-%E9%9B%86%E7%BE%A4%E8%A7%84%E5%88%92
# doris 集群部署
> 3节点，资源有限每个节点安装FE，BE 第一台安装的节点作为master节点
```
# 查看 FE 运行状态
curl http://127.0.0.1:8030/api/bootstrap
```


## fe.conf
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
# 绑定集群ip网段
priority_networks = 192.168.50.0/24
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
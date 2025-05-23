# 单机版doris安装
以下是在单机环境下安装 Apache Doris 2.1.10 的详细步骤：

---

### **1. 系统要求**
- **操作系统**: Linux (推荐 CentOS 7+/Ubuntu 16.04+)
- **Java**: JDK 8 或 JDK 11 (需配置 `JAVA_HOME`)
- **内存**: 至少 4GB (建议 8GB+)
- **磁盘**: 预留至少 20GB 空间

---

### **2. 下载 Doris 2.1.10**
```bash
# 从 Apache 镜像站下载二进制包
wget https://archive.apache.org/dist/doris/2.1.10/apache-doris-2.1.10-bin-x64.tar.gz

# 解压安装包
tar -zxvf apache-doris-2.1.10-bin-x64.tar.gz
cd apache-doris-2.1.10/
```
### 安装至
```
mkdir -p /usr/doris

mv ./* /usr/doris
```
---

### **3. 单机部署配置**
#### **3.1 配置 Frontend (FE)**
- 修改 `fe/conf/fe.conf`：
  ```properties
  # 元数据存储路径
  meta_dir = /usr/doris/fe/doris-meta
  ```

#### **3.2 配置 Backend (BE)**
- 修改 `be/conf/be.conf`：
  ```properties
  # 数据存储路径
  storage_root_path = /usr/doris/be/storage
  ```

---

### **4. 启动 Doris 服务**
#### **4.1 启动 FE**
```bash
# 启动 FE（首次启动需指定 --daemon 后台运行）
./fe/bin/start_fe.sh --daemon

# 查看 FE 日志确认状态
tail -f fe/log/fe.log
```

#### **4.2 启动 BE**
```bash
# 启动 BE
./be/bin/start_be.sh --daemon

# 查看 BE 日志确认状态
tail -f be/log/be.log
```

##### 启动出现问题
```
root@iZwz9hp7b8y6ywwtu0pkt0Z:/usr/doris# ./be/bin/start_be.sh --daemon
Please set vm.max_map_count to be 2000000 under root using 'sysctl -w vm.max_map_count=2000000'.
```
> 解决方法
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

# 停止 BE 服务
./be/bin/stop_be.sh

# 启动 BE 服务
./be/bin/start_be.sh --daemon

# 查看日志确认是否成功
tail -f be/log/be.log
```

> jdk问题
```
root@iZwz9hp7b8y6ywwtu0pkt0Z:/usr/doris# ./be/bin/start_be.sh --daemon
The JAVA_HOME environment variable is not defined correctly
This environment variable is needed to run this program
NB: JAVA_HOME should point to a JDK not a JRE
You can set it in be.conf
root@iZwz9hp7b8y6ywwtu0pkt0Z:/usr/doris# echo $JAVA_HOME
/usr/lib/jvm/java-17-openjdk-amd64/


```
解决
```
# 编辑配置文件（如 ~/.bashrc 或 /etc/profile）
sudo vi /etc/profile

# 在文件末尾添加以下内容
JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64/"
PATH="$PATH:$JAVA_HOME/bin"

# 使配置生效
source /etc/profile

# 验证全局生效
su - root  # 重新登录 root 账户
echo $JAVA_HOME  # 应显示 Java 11 路径
```
> 问题3
```
root@iZwz9hp7b8y6ywwtu0pkt0Z:/usr/doris# ./be/bin/start_be.sh --daemon
./be/bin/start_be.sh: line 75: swapon: command not found


解决：


# Ubuntu/Debian 系统
sudo apt update
sudo apt install util-linux -y

# 确认 swapon 安装路径（通常在 /sbin/swapon）
which swapon || find / -name swapon 2>/dev/null

echo 'export PATH=/sbin:$PATH' | sudo tee -a /etc/profile
source /etc/profile
```

---

### **5. 将 BE 注册到 FE**
通过 MySQL 客户端连接 FE，执行以下命令：
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

---

### **6. 验证安装**
#### **6.1 检查 FE/BE 状态**
```sql
SHOW PROC '/frontends'\G
SHOW PROC '/backends'\G
```
- 确保 `Alive` 字段为 `true`。

#### **6.2 简单测试**
```sql
CREATE DATABASE test_db;
USE test_db;
CREATE TABLE test_table (id INT, name VARCHAR(20)) DISTRIBUTED BY HASH(id);
INSERT INTO test_table VALUES (1, 'Doris Test');
SELECT * FROM test_table;
```

---

### **常见问题排查**
1. **端口冲突**  
   - FE 默认端口：`8030` (HTTP), `9020` (BRPC), `9030` (MySQL)
   - BE 默认端口：`9060` (BRPC), `8040` (HTTP), `9050` (Heartbeat)
   - 检查端口占用：`netstat -tunlp | grep <端口号>`

2. **元数据/存储目录权限**  
   ```bash
   chmod 755 /path/to/doris-meta
   chmod 755 /path/to/doris-storage
   ```

3. **内存不足**  
   - 调整 `fe/conf/fe.conf` 和 `be/conf/be.conf` 中的 JVM 参数（如 `-Xmx4G`）。

---

常用命令
```
# 一键清理所有 Doris 进程
pkill -f 'doris_fe|doris_be|java.*Doris'

#启动
bash fe/bin/start_fe.sh --daemon 
bash be/bin/start_be.sh --daemon
```

## 设置root密码 可远程登录
```
mysql -h 127.0.0.1 -P 9030 -uroot
修改 root 密码

sql
-- 设置新密码（将 YourPassword 替换为实际密码）
ALTER USER 'root' IDENTIFIED BY 'YourPassword';

#启用密码验证（如果未开启）
#编辑 fe/conf/fe.conf，添加或修改：


enable_auth_check = true
重启 FE 服务：

bash
./fe/bin/stop_fe.sh
./fe/bin/start_fe.sh --daemon
```

# 访问doris web页面
http://ip地址:8030/
navicat : 端口为9030
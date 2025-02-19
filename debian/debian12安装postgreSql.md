# Debian 12 安装 PostgreSQL 

以下是 Debian 12 安装 PostgreSQL 的步骤：

---

步骤 1：更新系统
```bash
sudo apt update && sudo apt upgrade -y
```
确保系统软件包为最新版本。[6]

---

步骤 2：安装 PostgreSQL 及附加组件
```bash
sudo apt install postgresql postgresql-contrib -y
```
安装 PostgreSQL 数据库服务器及常用工具（如扩展支持）。[2][6][7]

---

步骤 3：验证服务状态
```bash
sudo systemctl status postgresql
```
检查服务是否正常运行（显示 `active (running)` 即为成功）。[6]

---

步骤 4：访问 PostgreSQL 控制台
```bash
sudo -i -u postgres psql
```
以默认管理员用户 `postgres` 身份进入数据库命令行。[6][7]

---

步骤 5：修改默认用户密码（可选但推荐）
在 PostgreSQL 控制台中执行：
```sql
ALTER USER postgres WITH PASSWORD '你的新密码';
\q  # 退出控制台
```
修改 `postgres` 用户的密码以增强安全性。[6][7]

---

步骤 6：配置远程访问（可选）
编辑配置文件：
   ```bash
   sudo nano /etc/postgresql/*/main/postgresql.conf
   ```
   修改 `listen_addresses = '*'` 以允许远程连接。[1][6]

配置访问权限：
   ```bash
   sudo nano /etc/postgresql/*/main/pg_hba.conf
   ```
   添加行：
   ```text
   host  all  all  0.0.0.0/0  scram-sha-256
   ```

重启服务：
   ```bash
   sudo systemctl restart postgresql
   ```

---

步骤 7：安装图形管理工具（可选）
```bash
sudo apt install pgadmin3 -y
```
安装 `pgAdmin III` 方便可视化操作。[7]

---

验证安装
重新登录并查看数据库列表：
```bash
sudo -i -u postgres psql -c "\l"
```
若显示默认数据库（如 `postgres`, `template1`），则安装成功。[6]

---

附注
源码安装：如需自定义编译安装，可参考源码包步骤（需处理依赖和配置）。[1]
用户管理：建议创建独立用户和数据库，而非直接使用 `postgres` 超级用户。[2][6]
# debian12安装mysql8

以下是在 Debian 12 上安装 MySQL 的步骤，综合多个来源的最佳实践：


步骤 2：添加 MySQL 官方仓库并安装
下载 MySQL 仓库配置包  
   从官网获取最新版（以 2025 年为例，版本号可能更新）：
   ```bash
   wget https://repo.mysql.com/mysql-apt-config_0.8.29-1_all.deb
   apt install gnupg
   ```
  

安装配置包  
   ```bash
   sudo dpkg -i mysql-apt-config_0.8.29-1_all.deb
   ```
   安装时选择默认配置（按 `Tab` 键确认 MySQL 8.0/8.4 版本）。  

更新软件包列表  
   ```bash
   sudo apt update
   ```

---

步骤 3：安装 MySQL Server
安装核心组件  
   ```bash
   sudo apt install mysql-server
   ```
   安装过程中会提示设置 root 密码，`建议直接设置（避免后续配置麻烦）`。  


处理依赖问题（若出现）  
   - 依赖 `libssl1.1`（若报错）：
     ```bash
     wget https://mirrors.tuna.tsinghua.edu.cn/debian/pool/main/o/openssl/libssl1.1_1.1.1n-0+deb10u3_amd64.deb
     sudo dpkg -i libssl1.1_1.1.1n-0+deb10u3_amd64.deb
     ```
   - 依赖 `mecab-ipadic-utf8`：
     ```bash
     sudo apt install mecab-ipadic-utf8
     ```

---

步骤 4：安全配置（若安装时未设置密码）
初始化安全设置  
   ```bash
   sudo mysql_secure_installation
   ```
   按提示设置密码、移除匿名用户、禁止远程 root 登录等。  

---

步骤 5：验证安装
启动服务并检查状态  
   ```bash
   sudo systemctl start mysql
   sudo systemctl status mysql
   ```

登录 MySQL  
   ```bash
   mysql -u root -p
   ```
   输入密码后进入 MySQL Shell 即表示成功。  

---
设置远程登录
```
mysql -u root -p 输入你设置的密码

use mysql;

select host, user, authentication_string, plugin from user; 

update user set host='%' where user='root';

Grant all privileges on *.* to 'root'@'%';

flush privileges;
```

---

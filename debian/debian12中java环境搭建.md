# debian12中java环境搭建

## jdk安装
> Debian 系统的官方软件源中通常包含了 OpenJDK 17 的软件包
```
apt install openjdk-17-jdk
```
> 安装完成后，可以通过以下命令验证 OpenJDK 17 是否安装成功：
```
java -version
```
> 一般情况下，安装 OpenJDK 17 后，系统会自动配置好 Java 环境变量。但如果需要手动配置，可以按照以下步骤进行：
> 可以使用以下命令查找 OpenJDK 17 的安装路径：
```
readlink -f $(which java) | sed "s:bin/java::"
root@localhost:~# readlink -f $(which java) | sed "s:bin/java::"
/usr/lib/jvm/java-17-openjdk-amd64/
```
> 编辑环境变量文件
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
echo $JAVA_HOME  # 应显示 Java 17 路径
```



## debian12安装maven
```
apt install maven
mvn -version
```

## 安装git
```
apt install git -y
git --version
# 配置全局用户信息
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
# 查看配置信息
git config --list
```
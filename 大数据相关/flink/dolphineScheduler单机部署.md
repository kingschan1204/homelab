# dolphineScheduler 单机部署
```
tar -zxvf apache-dolphinscheduler-3.2.0-bin.tar.gz

```
## 使用mysql作为存储库
## 将 MySQL 的驱动包上传到 DolphinScheduler 的每个模块的 libs 目录下，其中包括

- standalone-server/libs/standalone-server/
- tools/libs/
- api-server/libs
- alert-server/libs
- master-server/libs
- worker-server/libs

## 修改配置
> 修改 `bin/env/dolphinscheduler_env.sh`设定下列环境变量，修改如下配置：
```
vim bin/env/dolphinscheduler_env.sh
export DATABASE=mysql
export SPRING_PROFILES_ACTIVE=${DATABASE}
export SPRING_DATASOURCE_URL=jdbc:mysql://192.168.1.73:3306/ds1?useUnicode=true&characterEncoding=UTF-8&useSSL=false
export SPRING_DATASOURCE_USERNAME=root
export SPRING_DATASOURCE_PASSWORD=root123
```
## 修改 `tools/conf/application.yaml`中 MySQL 的相关配置
```
vim tools/conf/application.yaml

# 修改下面的 mysql url 跟前面的配置保持一致
---
spring:
  config:
    activate:
      on-profile: mysql
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.1.73:3306/ds1?useUnicode=true&characterEncoding=UTF-8
    username: xxx
    password: xxx
```
## 修改`standalone-server/conf/application.yaml` 中的 MySQL 相关配置
```
vim standalone-server/conf/application.yaml

---------
spring:
  config:
    activate:
      on-profile: mysql
  sql:
     init:
       schema-locations: classpath:sql/dolphinscheduler_mysql.sql
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.1.73:3306/ds1?useUnicode=true&characterEncoding=UTF-8
    username: xxx
    password: xxx
```

## 初始化数据库
```
bash tools/bin/upgrade-schema.sh
```
## 启动 standalone-server
> 此时你已经连接上mysql，重启 或 停止 standalone-server 并不会清空您数据库里的数据
```

# 启动 Standalone Server 服务
bash ./bin/dolphinscheduler-daemon.sh start standalone-server

# 停止 Standalone Server 服务
bash ./bin/dolphinscheduler-daemon.sh stop standalone-server

# 查看 Standalone Server 状态
bash ./bin/dolphinscheduler-daemon.sh status standalone-server

# 查看日志
tail -fn 200 standalone-server/logs/dolphinscheduler-standalone.log
```
## 访问系统
```
浏览器访问地址 http://localhost:12345/dolphinscheduler/ui 即可登录系统UI

默认的用户名和密码是 admin / dolphinscheduler123
```
---
> 参考： https://www.hnbian.cn/posts/723b083d.html#toc-heading-1
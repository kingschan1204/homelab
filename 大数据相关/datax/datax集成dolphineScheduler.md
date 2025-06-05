# datax集成dolphineScheduler

## 1.安装dataX

## 2.修改`/etc/profile`文件增加如下环境变量
```
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PYTHON_LAUNCHER=/usr/bin/python3
export DATAX_LAUNCHER=/usr/datax/bin/datax.py
export PATH=$JAVA_HOME/bin:$PATH
```
> 完成，可以去海豚里测试调度
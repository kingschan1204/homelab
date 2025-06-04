# DataX安装
> https://github.com/alibaba/DataX/blob/master/userGuid.md

## 前置条件
- jdk
- python 2,3

> debian12已经预安装了python不过默认的python命令并没有链接到 Python 3但是无法执行python命令
```
# 修复无法执行python命令
ln -s /usr/bin/python3 /usr/bin/python
```
## 开始安装
```
wget https://datax-opensource.oss-cn-hangzhou.aliyuncs.com/202309/datax.tar.gz

tar -zxvf datax.tar.gz

cd datax/bin
mv datax /usr/

# 自检脚本
python {YOUR_DATAX_HOME}/bin/datax.py {YOUR_DATAX_HOME}/job/job.json
python /usr/datax/bin/datax.py /usr/datax//job/job.json
```
> 如何成功运行说明安装成功
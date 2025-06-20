# doris导入测试数据
> 官方tpch测试数据
https://doris.apache.org/zh-CN/docs/benchmark/tpch

## 下载 并上传到linux服务器
https://github.com/apache/doris/tree/master/tools/tpch-tools

## 依赖
```
apt install gcc
apt install unzip
apt install make

```
## 编译 tpch-tools
```
sh bin/build-tpch-dbgen.sh
```
> 安装成功后，将在 TPC-H_Tools_v3.0.0/ 目录下生成 dbgen 二进制文件。
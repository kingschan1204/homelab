可以上传一个iso，然后再xshell后台使用find命令查找这个iso，从而确定loacl目录iso的存储路径，例如：
使用find命令查找ubuntu-20.04.4-live-server-amd64.iso
```
find / -name ubuntu-20.04.4-live-server-amd64.iso
```
输出结果
```
/var/lib/vz/template/iso/ubuntu-20.04.4-live-server-amd64.iso
```
从而确定目录为`/var/lib/vz/template/iso/`
可不用web页面上传,直接上传到这个目录即可。


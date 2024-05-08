---
title: "常用命令"
date: 2024-05-08T17:05:28+08:00
description: "linux常用命令"
draft: false
tags: ["linux"]
categories: ["linux"]
series: ["linux"]
---
**1、rpm文件解压命令**
```shell
rpm2cpio filename | cpio -div
```
**2、yum一次性删除多个rpm包**

    yum list --installed|grep xxx|awk '{print $1}'|xargs yum remove -y

**3、批量解压当前目录所有rpm包**
```shell
find . -type f -print0 | xargs -0 -I x sh -c 'rpm2cpio x | cpio -idm'
```
**4、查看全部的vsock连接**
```shell
# ss -a --vsock -p
``` 
**5、解压initrd文件**
`mv initrd initrd.gz && gunzip initrd.gz && cpio -i < initrd`
将解压后的initrd合成initrd
`find .|cpio --quiet -H newc -o|gzip -9 -n > ../initrd`

**6、测试内存读写速度**
Stream测试内存性能数据

`gcc -O -fopenmp -DSTREAM_ARRAY_SIZE=100000000 -DNTIME=20 stream.c -o stream`

https://www.cnblogs.com/iouwenbo/p/14377478.html
**7、查看128个CPU内存占用情况**
```shell
yum install sysstat -y
service sysstat restart
# 查看全部CPU每个CPU的占用情况，方便观察不同numa上CPU占用分布
sar -P ALL -u 1 100
```

**8、查看vsock是否通**
`yum install nmap`
服务端开启：`nc -v --vsock -l 1234`
客户端连接：`nc -v --vsock 3 1234`

**9、查看根目录大文件超过1G**
```shell
find / -xdev -size +1G -exec ls -l {} \;
find / -type f -size +1G -exec du -h {} \;

# 选其一即可
# 命令作用是：查询根目录（/）下超过1G大小的文件
```
**10、调试coredump文件**
```shell
gdb binary_file core_file
bt
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbNDUxMzc4NTgyLDczMDk5ODExNl19
-->
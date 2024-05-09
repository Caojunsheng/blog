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
**11、内核模块编译**
```shell
#编译kernel配置文件
cd /usr/src/kernels/4.18.0-147.5.2.19.centos.x86_64/
make oldconfig && make prepare
# 修改内核代码后，编译vsock ko模块
make M=/mnt/xxx/tmp/usr/src/linux-5.10.0-60.18.0.50.x86_64/net/vmw_vsock/ -C /lib/modules/`uname -r`/build -j10
# 卸载原有ko，安装新的ko
rmmod vhost_vsock
rmmod vmw_vsock_virtio_transport_common
insmod /mnt/xxx/tmp/usr/src/linux-5.10.0-60.18.0.50.x86_64/net/vmw_vsock//vmw_vsock_virtio_transport_common.ko
insmod /lib/modules/`uname -r`/kernel/drivers/vhost/vhost_vsock.ko.xz
```

**12、设置git https校验位false**

`git config --global http.sslVerify false`

13、git命令补全
拷贝下面文件内容放到文件~/.git-completion.bash里面
https://github.com/git/git/blob/master/contrib/completion/git-completion.bash

vim ~/.bash_profile

if [ -f ~/.git-completion.bash ]; then 
. ~/.git-completion.bash 
fi 

source ~/.bash_profile


git保存密码
git config --global credential.helper store

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY2NjU5NTQyMSwtNDgwNjY5MTcyLC0xMT
kxMzA2MTIsMTYzMjA4MTQzMiw3MzA5OTgxMTZdfQ==
-->
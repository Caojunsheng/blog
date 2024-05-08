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
**13、查看全部的vsock连接**
```shell
# ss -a --vsock -p 

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTUzNDYyMzQwNSw3MzA5OTgxMTZdfQ==
-->
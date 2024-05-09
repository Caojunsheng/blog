
### 1.安装一个包
```shell
# rpm -ivh < rpm package name>
```
### 2.安装参数
--force 即使覆盖属于其它包的文件也强迫安装
--nodeps 如果该RPM包的安装依赖其它包，即使其它包没装，也强迫安装。
### 3.升级一个包
```shell
# rpm -Uvh < rpm package name>
```
### 4.移除一个包
```shell
# rpm -e < rpm package name>
```
### 5.查询一个包是否被安装
```shell
# rpm -q < rpm package name>
```
### 6.得到被安装的包的信息
```shell
# rpm -qi < rpm package name>
```
### 7.列出该包中有哪些库和头文件
```shell
# rpm -ql < rpm package name>
```
### 8.列出服务器上的一个文件属于哪一个RPM包
```shell
#rpm -qf + 文件名
```
### 9.可综合好几个参数一起用
```shell
# rpm -qil < rpm package name>
```
### 10.列出所有被安装的rpm package
```shell
# rpm -qa
```
### 11.列出一个RPM包文件中包含有哪些文件
```shell
# rpm -qlp < rpm package name>
```
### 12.解压一个rpm包
```shell
# rpm2cpio xxx.rpm | cpio -idm
```
### 13.批量解压当前目录所有rpm包
```shell
find . -type f -print0 | xargs -0 -I x sh -c 'rpm2cpio x | cpio -idm'
```

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzE2OTUyNTM2XX0=
-->
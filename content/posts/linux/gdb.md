---
title: "gdb常用命令"
date: 2024-05-09T17:15:28+08:00
description: "linux gdb常用命令"
draft: false
tags: ["gdb"]
categories: ["linux"]
series: ["linux"]
---

**相关命令：**

r（run）：运行

q（quit）：退出

b（break）：打断点

	• b function_name
	• b row_num
	• b file_name:row_num
	• b row_num if condition
	
c（continue）：继续执行

print $xx：打印具体变量

watch $xx：监控一个对象

(gdb) rwatch num          当要观察的变量num被读时，程序暂停运行

(gdb) awatch num          当要观察的变量num被读或被写，程序暂停运行

(gdb) info watchpoints    查看当前设置的所有观察点

disable $num：禁用断点

d $num（delete）：删除断点

n（next）：单步执行

s（step）：跳入一个函数

finish：跳出函数

whatis $xx：查看变量类型

winheight：启动可视化调试

bt ：bt是 backtrace 指令的缩写，显示所有的函数调用栈的信息，栈中的每个函数都被分配了一个编号，最近被调用的函数在 0 号帧中（栈顶），并且每个帧占用一行。

bt n ：显示函数调用栈从栈顶算起的n帧信息（n 表示一个正整数）。

bt -n ：显示函数调用栈从栈底算起的n帧信息。

bt full ：显示栈中所有信息如：函数参数，本地变量等。

bt full n ：显示函数调用栈从栈顶算起的n帧的所有信息。

bt full -n ：显示函数调用栈从栈底算起的n帧的所有信息。

set print pretty on：格式化打印

连续输出数组

(gdb) set print array-indexes on

输出index从0开始共十个

(gdb) p buffer[0]@10

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwMTE5MDM4NTRdfQ==
-->
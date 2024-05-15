---
title: "PCIe aer-inject注入AER错误"
date: 2024-05-15T15:59:28+08:00
description: "通过aer-inject工具注入AER错误"
draft: false
tags: ["PCIe", "AER"]
categories: ["linux"]
series: ["linux"]
---
对AER注入错误需要内核模块支持注入，需要安装aer_inject.ko模块，以及需要工具aer-inject命令行工具

### 1、编译出aer_inject.ko
```shell
#编译ko模块
make M=/mnt/cjs/kernel/usr/src/linux-4.19.90-vhulk2202.2.0.h1064.aarch64/drivers/pci/pcie -C /lib/modules/4.19.90-vhulk2202.2.0.h1064.aarch64/build/ -j10 CONFIG_PCIEAER_INJECT=m
#安装ko模块
insmod aer_inject.ko
```

安装成功后，能够在dev设备下看到aer_inject

```shell
# ll /dev/aer_inject 
crw------- 1 root root 10, 51 May 15 10:14 /dev/aer_inject
```

### 2、获取aer-inject aer错误注入工具

```shell
git clone https://github.com/jderrick/aer-inject.git
cd aer-inject
make
```

### 3、aer错误注入
执行下面命令注入错误，这个代表针对08:0e.0设备注入不可校正的SURPDN（Surprise Down Error）错误

`./aer-inject -s 08:0e.0 examples/fatal`

examples/fatal文件

```shell
AER
# PCI_ID [WWWW:]XX.YY.Z
UNCOR_STATUS SURPDN
HEADER_LOG 0 1 2 4
```
#### 3.1 先屏蔽AER错误，观察是否正常注入错误
由于该设备不可校正错误掩码寄存器中将SDES置为1，所以将该错误屏蔽了，从/var/log/message中日志也可以看出。
```shell
	Capabilities: [100 v1] Advanced Error Reporting
		UESta:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
		UEMsk:	DLP- SDES+ TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
		UESvrt:	DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
		CESta:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr-
		CEMsk:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr+
		AERCap:	First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
			MultHdrRecCap- MultHdrRecEn- TLPPfxPres- HdrLogCap-
		HeaderLog: 00000000 00000000 00000000 00000000

```
![](http://image.huawei.com/tiny-lts/v1/images/92dc466439eaff8b5599570acce3fa70_1015x154.png)

/var/log/message日志
```shell
2024-05-15T14:57:36.877516+08:00|warning|kernel[-]|[17763.652315] pci 0000:08:0e.0: aer_inject: The uncorrectable error(s) is masked by device
2024-05-15T14:57:36.887546+08:00|notice|[/bin/bash]|[2024-05-15 14:57:36 root ./aer-inject -s 08:0e.0 examples/fatal] return code=[255], execute failed by [root(uid=0)] from [pts/0 (52.170.130.96)]
```

#### 3.2 不屏蔽AER错误，观察是否正常注入错误

通过setpci将该掩码SDES置为0，取消错误屏蔽

`setpci -s 08:0e.0 100+08.L=00400000`

再执行aer错误注入

`./aer-inject -s 08:0e.0 examples/fatal`

会发现该设备已出现错误，故障注入成功

```shell
# lspci -vvs 08:0e.0
08:0e.0 PCI bridge: Huawei Technologies Co., Ltd. Hi1822 Family Virtual Bridge (rev ff) (prog-if ff)
	!!! Unknown header type 7f
```



> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTMyMDM2NTI1N119
-->
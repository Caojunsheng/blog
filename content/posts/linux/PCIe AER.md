---
title: "PCIe AER寄存器值查看和修改"
date: 2024-05-08T15:59:28+08:00
description: "如何查看PCIe AER寄存器值和修改寄存器值"
draft: false
tags: ["PCIe"]
categories: ["linux"]
series: ["Hugo Guide"]
---

### 1、AER简介
AER 即 Advanced Error Reporting高级错误报告，是PCIe高级特性，用于报告PCIe 错误信息。

分为
- 可纠正错误（Correctable errors）是指错误发生后，硬件可以自动恢复。
- 不可纠正错误（Uncorrectable errors）错误发生后，影响设备功能，硬件不能自动恢复。

不可纠正错误分为
- ERR_FATAL是致命错误，此错误类型影响了PCIe link链路。
- ERR_NONFATAL是指影响了设备功能，但是PCIe link还是稳定的。

### 2、lspci查看PCIe设备AER能力
使用下面命令可以查看PCIe设备的详细信息和相关的Capabilities
```shell
# lspci -vvs 89:00.7
```

其中AER相关能力在这里
```shell
Capabilities: [100 v2] Advanced Error Reporting
		UESta:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq+ ACSViol-
		UEMsk:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq+ ACSViol-
		UESvrt:	DLP+ SDES+ TLP+ FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
		CESta:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr+
		CEMsk:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr-
		AERCap:	First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
			MultHdrRecCap- MultHdrRecEn- TLPPfxPres- HdrLogCap+
		HeaderLog: 00000000 00000000 00000000 00000000
```

**寄存器命名**
- UESta: Uncorrectable Error Status，表示不可纠正错误状态，用于指示发生不可纠正错误的状态。 
- UEMsk: Uncorrectable Error Mask，表示不可纠正错误掩码，用于掩盖或屏蔽不可纠正错误。 
- UESvrt: Uncorrectable Error Severity，表示不可纠正错误严重性，用于指示不可纠正错误的严重程度。 
- CESta: Correctable Error Status，表示可纠正错误状态，用于指示发生可纠正错误的状态。 
- CEMsk: Correctable Error Mask，表示可纠正错误掩码，用于掩盖或屏蔽可纠正错误。 

**Uncorrectable Error**
- DLP: Data Link Protocol Error，数据链路协议错误，表示数据链路协议错误。 
- SDES: Surprise Down Error Status，意外下降错误状态，表示设备在意外下降时的错误状态。 
- TLP: Transaction Layer Packet，事务层包，表示事务层包错误。 
- FCP: Flow Control Protocol Error，流控制协议错误，表示流控制协议错误。 
- CmpltTO: Completion Timeout，完成超时，表示完成操作超时。 
- CmpltAbrt: Completion Abort，完成中止，表示完成操作中止。 
- UnxCmplt: Unsupported Completion，不支持的完成，表示不支持的完成操作。 
- RxOF: Receiver Overflow，接收器溢出，表示接收器溢出错误。 
- MalfTLP: Malformed TLP，格式错误的TLP，表示格式错误的事务层包。 
- ECRC: ECRC Error，ECRC 错误，表示ECRC校验错误。 
- UnsupReq: Unsupported Request，不支持的请求，表示设备不支持的请求。 
- ACSViol: ACS Violation，ACS 违规，表示ACS规则违反。 

**Correctable Error**
- RxErr: Receive Error，表示接收数据时发生错误。 
- BadTLP: Bad TLP，表示传输层包（TLP）格式错误或无效。 
- BadDLLP: Bad DLLP，表示数据链路层包（DLLP）格式错误或无效。 
- Rollover: 溢出，表示发生了溢出错误。 
- Timeout: 超时，表示操作超时。 
- AdvNonFatalErr: 高级非致命错误，表示发生了高级非致命错误。

**AERCap**
- ECRCGenCap 和 ECRCGenEn 表示设备是否支持和启用 ECRC 生成（ECRC generation）功能。 
- ECRCChkCap 和 ECRCChkEn 表示设备是否支持和启用 ECRC 检查（ECRC checking）功能。 
- MultHdrRecCap 和 MultHdrRecEn 表示设备是否支持和启用多头记录（Multiple Header Record）功能。 
- TLPPfxPres 表示设备是否支持 TLP 前缀保留（TLP Prefix Preservation）功能。 
- HdrLogCap 表示设备是否支持头部日志（Header Logging）功能。 

### 3、setpci查看设备AER寄存器值
查看设备89:00.7的AER Uncorrectable Error Severity寄存器值的命令如下所示：
```shell
# setpci -s 89:00.7 100+0C.L
00463030
```
-s 用于指定设备的bdf号，格式如：`[[[<domain>]:][<bus>]:][<slot>][.[<func>]]`
89:00.7 代表具体的设备
100 是参考lspci查出来的起始位`Capabilities: [100 v2] Advanced Error Reporting`
+0C 代表AER Uncorrectable Error Severity Register值的偏移量，这个值是根据PCIe Spec里面查到的
L代表是Long，长字，即32位，也可以是B、W等
- B：表示 Byte，即字节。在寄存器中，每个字节通常由 8 位组成。 
- W：表示 Word，即字。在寄存器中，一个字通常由 16 位或 2 个字节组成。 
- L：表示 Long，即长字。在寄存器中，一个长字通常由 32 位或 4 个字节组成。 

而查询出来的值00463030则对应到32位寄存器值，每个bit对应一个错误的严重程度，对应到lspci上查询出来的，+代表该bit位为1，-代表该bit位为0。
`UESvrt:	DLP+ SDES+ TLP+ FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-`

00463030换算成2进制为0100 0110 0011 0000 0011 0000

其中Bit4、Bit5、Bit12、Bit13、Bit17、Bit18、Bit22位为1，对应到PCIe Spec上，即
- Bit4：Data Link Protocol Error Severity（DLP）
- Bit5：Surprise Down Error Severity（SDES）
- Bit12：Poisoned TLP Received Severity（TLP）
- Bit13：Flow Control Protocol Error Severity（FCP）
- Bit17：Receiver Overflow Severity（RxOF）
- Bit18：Malformed TLP Severity（MalfTLP）
- Bit22：Uncorrectable Internal Error Severity
正好与lspci查询出来的结果一致。

### 4、setpci设置设备AER寄存器值
同样以设备89:00.7的AER Uncorrectable Error Severity寄存器值为例，
上面查询到的UESvrt的CmpltTO为-，我们想把它设置为+，参考PCIe Spec，我们只要设置对应寄存器的Bit14为1即可，命令如下：

```shell
# setpci -s 89:00.7 100+0C.L
00463030
# setpci -s 89:00.7 100+0C.L=00467030
# setpci -s 89:00.7 100+0C.L
00467030
# lspci -vvs 89:00.7
	Capabilities: [100 v2] Advanced Error Reporting
		UESta:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq+ ACSViol-
		UEMsk:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq+ ACSViol-
		UESvrt:	DLP+ SDES+ TLP+ FCP+ CmpltTO+ CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
		CESta:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr+
		CEMsk:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr-
		AERCap:	First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
			MultHdrRecCap- MultHdrRecEn- TLPPfxPres- HdrLogCap+
		HeaderLog: 00000000 00000000 00000000 00000000

```
上述查询出来的UESvrt的CmpltTO确实是+，设置成功。


> https://blog.csdn.net/zsmcdut/article/details/120151896


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM1OTA2MTM2OCwxNTg4Mzg4ODI2LC0yMT
E5MDA1NDkzXX0=
-->